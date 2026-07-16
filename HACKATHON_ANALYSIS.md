# Why We Lost, Why They Won, and How to Use Claude to Win Next Time

A deep, evidence-based post-mortem of the IUT 12th ICT Fest Agentic AI Hackathon
(Preliminary Round), comparing:

- **Ours** — `AdilShamim8/ICT_Fest_Hackathon_Preliminary` (not selected)
- **Winner** — `Hemalv02/ICT_Fest_Hackathon_Preliminary` (selected)

Both started from the identical buggy template (`Initial commit` `5bb6f56`). This
document is written from the actual code and git history of both repos, not from
impressions.

---

## 0. What the challenge actually was

This was **not** a "build something" hackathon and not really a data-science one.
It was a **black-box debugging challenge**:

- You're given a working-but-buggy FastAPI "CoWork" room-booking API.
- There is a precise spec: **16 business rules** + an exact API contract (paths,
  status codes, error codes, JSON field names, JWT claims).
- **Grading is black-box**: the grader builds your container and asserts behavior
  by talking to the API over HTTP only. It never reads your code.
- Bugs are scored by difficulty tier (the winner's report cites **easy 3 / medium 5
  / hard 10** points), and a `bug_report.md` is a tie-breaker/quality artifact.

So the entire game is: **find every deviation from the spec, fix it at the root, and
prove it holds under the conditions the grader will actually use.** Whoever does that
most completely and most robustly wins. That framing matters for everything below.

---

## 1. The surprising part: on paper, we were competitive

This is important for morale and for diagnosis. We did **not** lose because we're bad.

| | Ours | Winner |
|---|---|---|
| Bugs documented | 26 | 28 |
| Bug report quality | Strong, tiered, per-bug root cause + fix | Strong, tiered, per-bug root cause + fix |
| Core logic bugs fixed | ~23 of 26 correct | 28 correct |
| Concurrency bugs fixed | Yes (locks) | Yes (locks) |

We correctly fixed the hard-looking stuff most teams miss: the notifications
**deadlock** (lock-order inversion), the **double-booking** race, the **double-cancel**
race, the **rate-limit** bypass, refund half-cent rounding, the datetime UTC-offset
bug, the IDOR on `GET /bookings/{id}`, pagination, cache invalidation, and more. Our
`bug_report.md` is genuinely good.

**We lost on a small number of specific, decisive gaps** — not on volume. That's
actually the encouraging news: the gap is a *method* gap, not a *talent* gap, and
method is learnable and repeatable.

---

## 2. Exactly where we lost (the evidence)

Three substantive gaps, ranked by likely impact. Every one is backed by the diff.

### Gap #1 — We ignored container restart. This is almost certainly the decisive one.

The winner's bug report states plainly:

> "Because the grader restarts the container between test phases (confirmed by the
> organizers) and the SQLite file persists across a restart, four fixes were also
> checked by seeding data in one process, killing it, and starting a fresh process
> against the same database file."

They **asked the organizers how grading worked**, learned the container is restarted
mid-grading, and then engineered for it. We did not. Our fixes keep critical state
**in process memory**, which is wiped on restart. Three concrete failures result:

**a) Reference codes — `app/services/reference.py`**

- **Ours:** an in-memory counter `_counter = {"value": 1000}`, incremented per booking.
  On restart it resets to 1000, so the next booking gets `CW-001000` again — a
  duplicate of a code already in the database. **Violates Rule 7 (reference codes must
  be unique).** (`reference_code` is indexed but *not* a unique column, so this doesn't
  even crash — it silently produces duplicates the grader detects.)
- **Winner:** derives the next code from the persisted data —
  `max(existing "CW-" number in the bookings table) + 1`. Survives restart, stays
  unique.

**b) Room stats — `app/services/stats.py` + `app/routers/rooms.py`**

- **Ours:** `GET /rooms/{id}/stats` reads from an in-memory dict maintained by
  `record_create` / `record_cancel`. On restart the dict is empty, so a room that
  still has bookings in the DB reports **0 bookings / 0 revenue.** **Violates Rule 14**
  ("always equals the values derivable from the bookings themselves").
- **Winner:** made the read path query the `bookings` table directly
  (`count` and `sum(price_cents)` where `status = 'confirmed'`). Always correct, and it
  can never drift from the source of truth even *without* a restart.

**c) Token revocation — `app/auth.py`**

- **Ours:** logout blacklist and refresh single-use are two in-memory sets
  (`_revoked_tokens`, `_revoked_refresh_tokens`). On restart both are empty, so a
  logged-out access token works again and a spent refresh token is accepted again.
  **Violates Rule 8** (logout must invalidate; refresh tokens are single-use).
- **Winner:** added a persistent `RevokedToken` table (unique `jti`) — see their
  `models.py` change. Revocation survives restart, and the unique-`jti` insert doubles
  as the atomic single-use guard.

We literally wrote in our own bug report, under "Deliberately unchanged," that we kept
these in memory on purpose. That was the wrong call — and it was wrong precisely because
**we never modeled the grading environment.** The winner touched `models.py` and
`database`-backed logic; **we never touched `models.py` at all.**

### Gap #2 — `/admin/export` returns the wrong status for a cross-org / unknown room

- **Ours:** we fixed the *data leak* (we scoped `fetch_bookings_raw` by org, so no
  other org's rows come out — good). But for a cross-org or non-existent `room_id`, our
  handler returns **`200` with an empty CSV**.
- **Winner:** validates `room_id` against the caller's org first and returns
  **`404 ROOM_NOT_FOUND`**, matching how availability/stats/booking-create behave.
  **Rule 9** says cross-org IDs must behave as non-existent → `404`.

We found the leak but stopped one step short of the full rule. This is a "fixed the
symptom, missed the sibling case" miss — it needed an endpoint-vs-rule cross-check we
didn't do.

### Gap #3 — Login timing side-channel (security depth) not fixed

- **Ours:** `if user is None or not verify_password(...)`. The `or` short-circuits, so
  an unknown username **skips PBKDF2 entirely** (~24 ms faster). An attacker can
  enumerate valid usernames by timing responses.
- **Winner:** hashes the supplied password against a fixed dummy hash when the user is
  missing, so every login runs exactly one PBKDF2 — constant time. They added it as bug
  #28.

This one is hard to grade purely black-box, but it signals depth and security awareness
— exactly the kind of thing a judge remembers when breaking a tie.

### Gap #4 — Presentation & engineering trail (a trust signal, not a rule)

- **Ours:** **5 commits total**, two of them literally `"Add files via upload"` — i.e.
  files were produced somewhere else and dumped into GitHub through the web UI. There is
  no visible engineering process, nothing is bisectable, and it reads like the work
  appeared out of nowhere.
- **Winner:** **38 commits**, each an atomic fix with a precise message
  (`"fix: overlap check used inclusive <=, rejecting back-to-back bookings; use strict <"`),
  most paired with a matching `docs:` commit updating the bug report. A judge can read
  the history top to bottom and *trust* every claim.

For an **Agentic AI** hackathon especially, the commit history is part of the story: it
shows disciplined, tool-driven iteration. Ours hid all of that.

---

## 3. The root-cause of *our* mistakes (the pattern to kill)

Every gap above traces back to the same four habits. Fix the habits, not just this repo.

1. **We fixed symptoms, not root causes.** We saw a race and added a `threading.Lock`.
   We never asked the deeper question: *"why is this state in memory at all, and what is
   the real source of truth?"* The winner's fixes repeatedly make **the database the
   authority** (stats from bookings, ref codes from bookings, revocation in a table).
   Locks make concurrency safe *within one run*; persistence makes correctness hold
   *across runs*. We only did the first half.

2. **We never modeled the grader's environment.** Concurrency, **restart**, cross-org
   IDs, boundary values, malformed input, timing — these are the grader's weapons. We
   optimized for "does it work when I click through it once," not "what will an
   adversarial automated grader do to it." The single most valuable thing the winner did
   was **ask the organizers how grading works** and then build to that answer.

3. **We didn't do an exhaustive rule×endpoint audit.** 16 rules × ~15 endpoints is a
   finite grid. Every cell should be checked. The two misses (export `404`, login timing)
   are cells we never visited. A complete traceability pass finds them mechanically.

4. **We didn't work in a reproducible, tested, version-controlled loop.** "Add files via
   upload" means we weren't building the container, running it, hammering it, and
   committing per fix. We were editing and hoping. That's why the restart failure and the
   export-status miss survived to submission — nothing was *proving* them wrong.

---

## 4. How the winner actually won (so you can copy it)

- **Root-cause thinking** → made the DB the source of truth wherever state mattered.
- **Environment modeling** → confirmed the restart with organizers, then *verified* it
  by seeding data, killing the process, restarting, and re-asserting.
- **Rubric awareness** → ordered the report **hardest-first**, explicitly tied to the
  scoring tiers (10-point hard bugs first). They spent effort where the points were.
- **Rigorous, per-bug verification** → every entry names the exact check that now passes,
  and four name the restart check specifically.
- **Clean engineering trail** → one fix per commit, fix+docs pairs, 38 legible commits.

None of this is genius. It's **method**. And method is exactly what an AI-assisted
workflow is best at executing consistently — which is the whole point of the next section.

---

## 5. The Claude playbook — how to actually win these with Claude

You told me you leaned heavily on Claude (Fable and Opus) and still lost. The problem
almost certainly wasn't the model — Fable/Opus can absolutely reason its way to the
restart bug and the export-404. The problem was **how it was driven.** You used Claude as
a *code fixer* ("here's a bug, patch it"). To win, use Claude as a *reasoning partner and
a relentless verification engine.* Here's the concrete system.

### 5.1 Feed the spec, and force a traceability matrix (this alone catches Gap #2)

Before touching code, paste the **entire** problem statement and business rules into
Claude and say:

> "Build a traceability matrix: every business rule and every line of the API contract
> as a row; for each, cite the exact file/function that enforces it and mark PASS / FAIL /
> UNSURE with the reason. Do not fix anything yet. List every rule you can't map to code."

This converts a fuzzy hunt into a finite checklist. The export-`404` and the login-timing
misses are cells that a complete matrix *forces you to visit*. We skipped this and skipped
those cells.

### 5.2 Make Claude play the adversarial grader (this catches Gap #1)

Separately, prompt:

> "You are the black-box grader. You can only talk HTTP, and you will try to break this
> service. Enumerate every attack: concurrency on each mutating endpoint, **restarting the
> container mid-test**, running multiple workers, cross-org IDs, boundary values (exactly
> 48h, back-to-back, 0-hour), malformed input, timing side-channels, cache staleness. For
> each, tell me which rule it threatens and how you'd detect a violation."

The word **"restart"** comes out of this prompt. If we'd run it, we'd have found the
persistence bug ourselves. Adversarial framing is the highest-leverage prompt you're not
using.

### 5.3 Enforce root-cause discipline on every fix

For each bug, don't accept the first patch. Make Claude answer:

> "What is the **root cause**, not the symptom? Does this fix survive a **restart**? Does
> it survive **multiple processes**? Is the read path derived from the **source of truth**,
> or from a shadow copy that can drift? Show me the failure case where your fix is still
> wrong."

A lock is not a persistence fix. Asking "does this survive restart?" on the stats fix
would have flipped us from the in-memory dict to the DB query — the winner's exact move.

### 5.4 Build the verification harness *first*, and iterate against RED/GREEN

This is the biggest workflow change. Have Claude (Claude Code can do this end to end):

1. Write an **HTTP contract test suite** that hits every endpoint and asserts every rule.
2. Write **concurrency probes** (fire N parallel requests, assert exactly-one-wins).
3. Write a **restart probe**: seed data → `docker kill` / restart the container → re-assert
   stats, reference-code uniqueness, and token revocation.
4. **Build and run the actual container** and loop: run suite → read failures → fix →
   re-run, one fix per commit, until green.

Let Claude iterate against *test output*, not against its own confidence. Our "upload"
history proves we never ran this loop. The winner's history is that loop.

### 5.5 Use the right model for the right job

You have Fable and Opus — use them deliberately:

- **Opus / Fable (top-tier reasoning):** root-cause analysis, the adversarial-grader pass,
  concurrency/consistency reasoning, "does this survive restart/multi-process," and the
  final cold-eyes audit. This is where deep reasoning earns its keep — don't waste it as a
  typing assistant.
- **A fast model (e.g. Sonnet/Haiku):** mechanical, well-specified edits once the reasoning
  is done — apply this diff, rename this, write this boilerplate test.

The mistake to avoid isn't "wrong model," it's **using a strong model for a weak prompt.**
A 10-point reasoning model given a "patch this line" task produces a line patch. Give it the
hard question.

### 5.6 Red-team with a *fresh* Claude before you submit (catches leftover misses)

When you think you're done, open a **new** Claude session (or a subagent — Claude Code's
`/code-review` or a general-purpose agent) with **no memory of your fixes** and say:

> "Here is the spec and the current diff. Assume I missed at least 5 spec violations. Find
> them. Be specific about the rule and the failing input."

Cold eyes catch what tired eyes rationalized away (we rationalized the in-memory state in
writing). This one pass would likely have surfaced the export-404 and the restart problem.

### 5.7 Let Claude drive git — atomic commits + living bug report

Work *in* the repo, not in a scratch buffer you upload later. Have Claude:

- Make **one fix per commit** with a precise message.
- Update `bug_report.md` in the **same or paired commit**.
- Order the report **hardest-first, mapped to the scoring tiers.**

You get a bisectable history, a trustworthy story for the judges, and a report that writes
itself as you go. Never `"Add files via upload"` again.

### 5.8 Feed human intel back to Claude

Claude cannot know the grading harness unless you tell it. **Ask the organizers the
grading questions** ("Do you restart the container? How many workers? Do you test
concurrency? Is the DB file persistent?") and feed every answer into the adversarial pass.
The winner's decisive advantage started as a *question to a human*, then became an
engineering requirement Claude could build against.

---

## 6. The repeatable "champion loop" (use this every time)

For any bug-fix, backend, or data hackathon with a hidden grader:

1. **Ask the organizers** how grading works. Write the answers down.
2. **Paste the full spec** into Claude → build the **rule×endpoint traceability matrix**.
3. **Adversarial pass** (Opus/Fable): "you are the grader, how do you break this?" — include
   restart, concurrency, multi-worker, boundaries, cross-tenant, timing.
4. **Build the verification harness first**: contract suite + concurrency probes + restart
   probe. Run it in the real container.
5. **Fix root causes** one at a time; for each, prove it survives restart/multi-process and
   derives from the source of truth. One commit per fix.
6. **Cold red-team** with a fresh Claude/subagent: "find 5 things I missed."
7. **Bug report** hardest-first, tied to the point tiers, every entry naming the check that
   proves it.
8. Re-run the full harness green, then submit.

### For data-science hackathons specifically

The same loop maps directly — the "grader" is a held-out test set / leaderboard metric
instead of an HTTP suite:

- **Ask the organizers**: exact metric, is the test set from the same distribution, is there
  a time/compute limit, how many submissions.
- **Traceability** → validation strategy: does my local CV mirror how they'll score me? (The
  restart bug's analogue is **train/test leakage** and **CV that doesn't match the private
  split** — the environment mismatch that silently sinks you.)
- **Adversarial pass** → "how will my model fail on the private set?": distribution shift,
  leakage, overfit to public LB, edge segments.
- **Harness first** → a trustworthy local validation loop before any modeling; iterate
  against it, not against the public leaderboard.
- **Root cause** → understand *why* a feature helps, not just that it nudged the metric.
- **Red-team + clean notebook/commit trail** → reproducible, legible, trusted.

---

## 7. Honest bottom line

- We were **close** — a strong 26-bug fix with a good report. We didn't lose on effort or
  on the hard concurrency bugs.
- We lost on **three specific things**: (1) ignoring the container **restart** so our
  in-memory fixes for stats, reference codes, and token revocation evaporated; (2) returning
  `200` instead of `404` on `/admin/export` for a cross-org/unknown room; (3) leaving the
  login **timing side-channel** unfixed — plus a **messy, upload-based git history** that
  gave judges nothing to trust.
- All four are **method gaps**, not talent gaps. The winner didn't out-code us; they
  **out-processed** us — they asked how grading worked, modeled the adversary, fixed root
  causes, and proved every fix.
- No one can honestly promise "100% win." But if you run the **champion loop** in §6 and
  drive Claude as a **reasoning + verification partner** (§5) instead of a line-patcher, you
  close every gap that cost you this one — and you'll be operating at the level that gets
  selected.
