# The Champion System — First to Last

The capstone of the four-document set. `HACKATHON_ANALYSIS.md` explains what happened.
`DATATHON_PLAYBOOK.md` gives the strategy. `COMPETITION_BATTLE_CARD.md` gives the
during-event tactics and prompts. This document adds what serial winners know that
one-time competitors don't: **the game outside the event** — preparation, portfolio
strategy against a 7,000-team field, decision discipline under pressure, and the
compounding loop that turns one competition into a career of them.

---

## Part 0 — The "extraordinary thing," answered honestly

You keep asking for the one extraordinary thing. Here is the veteran answer, and it's
worth more precisely because it isn't magic:

**There is no secret technique. The extraordinary thing is that champions arrive with
the system already built and rehearsed, so the competition is the *execution of a
practiced play* while 6,990 other teams are improvising from a blank notebook.**

Watch a serial winner's first hour and it looks unremarkable: they clone their template,
run their profiling script, start their validation harness. Nothing clever is happening.
The cleverness happened over the preceding months. By hour 3 they have infrastructure
most teams won't have by hour 20 — and every subsequent hour of theirs is spent on the
two things that actually score (insight and communication) while everyone else is still
fighting plumbing. The compounding of small, prepared advantages *is* the extraordinary
thing. Everything below is how you build it.

And the honest calibration one more time: "win 100%" does not exist in a 7,000-team
field — anyone who promises it is selling something. What exists is moving your
*percentile of preparation* so high that you're reliably in the final-10 conversation,
and then winning your share of those. Serial winners lose competitions constantly. They
win *more* than anyone else because their floor is high and their process compounds.

---

## Part 1 — The preparation game (T-30 days → T-0)

This is the highest-ROI, least-practiced layer. Grandmasters treat competition skill as
something you build *between* events, not during them.

### 1.1 Build the arsenal repo (once, reuse forever)

A private `competition-arsenal` repo containing:

- **Profiling kit**: one script that takes any tabular/text/image dataset and outputs
  the full recon report — shapes, dtypes, missingness map, cardinalities, target
  balance, distribution plots, duplicate detection, train/test drift summary.
- **Validation harness skeleton**: parameterized CV runner (KFold / GroupKFold /
  StratifiedKFold / TimeSeriesSplit), OOF prediction saving, per-fold score + std
  reporting, and a ready-made **adversarial validation** module.
- **Model zoo baselines**: ready-to-run GBDT (LightGBM/XGBoost/CatBoost), linear,
  and a simple NN, each wired to the harness. Grandmaster workflow is EDA → baselines
  → feature engineering → hill-climb/stack; your zoo makes step 2 a 15-minute step.
- **The plotting theme + hero-chart function** from the playbook, importable.
- **Submission checklist** as a file: seeds fixed, runs top-to-bottom, artifact
  verified, limitations section present, hero chart titled with the finding.
- **The prompt arsenal** (`COMPETITION_BATTLE_CARD.md` P0–P8) as text files ready to
  paste.
- **Speed matters more than you think**: Chris Deotte (one of the most decorated
  grandmasters alive) lists "accelerate your experimental pipeline" as one of his four
  keys — the team that runs 40 measured experiments beats the team that runs 8, almost
  regardless of talent. Pre-solve your compute: GPU access, cached environments,
  intermediate-artifact saving so nothing is ever computed twice.

### 1.2 Set up Claude Code as a competition machine (the deep integration)

This is "how should we use it in our full workflow," answered at the infrastructure
level rather than the prompt level:

- **`CLAUDE.md` in the arsenal repo** — your standing orders, loaded automatically
  every session: "This is a competition repo. Always: validation before modeling; one
  change per iteration; report CV mean ± std after every experiment; never touch the
  test set except through the harness; every experiment gets a commit with its score
  in the message." Claude now *defaults* to championship discipline instead of you
  re-prompting it every time.
- **Custom skills** (`.claude/skills/`): package your repeatable plays as slash
  commands — `/recon` (run profiling kit, summarize), `/harness` (build CV for this
  data's structure), `/experiment <idea>` (implement one change, run, report score,
  commit with score), `/redteam` (spawn a fresh-context subagent attacking the current
  work), `/herochart <finding>`. You are literally encoding the battle card into the
  tool.
- **Hooks**: a post-edit hook that auto-runs the fast validation script and surfaces
  the score — every Claude edit gets scored without anyone remembering to ask. This
  mechanizes the AIDE loop ("every iteration ends in a measured number").
- **Subagents + git worktrees = manual tree search**: run 2–3 experiment branches in
  parallel worktrees (one subagent tries feature idea A, another tries target
  transform B) while the main session holds the best-known-good. Merge winners by
  score, discard losers. This is the 10×-medal AIDE structure, implemented with the
  tools you already have.
- **Fresh sessions for red-teaming, always.** A session that built the solution will
  defend it; context is bias. The red-team prompt goes to a cold session or subagent.
- **Plan mode for the big forks**: before committing to an approach (feature-heavy
  GBDT vs NN vs ensemble), have Claude plan and argue the fork explicitly rather than
  drifting into the first idea.

### 1.3 The study regime: winning write-ups are the curriculum

The single most consistent grandmaster advice: **after every competition, read the
top-5 solution write-ups and rebuild what you didn't understand.** Ruchi Bhatia
(grandmaster) describes rebuilding winners' models from scratch until she truly
understood *why* they worked. Treat discussion forums like research papers — top
competitors leave leakage hints, metric quirks, and feature ideas in threads.

Concrete regime: one evening per week, pick a finished competition on Kaggle similar
to your target events, read the 1st–5th place write-ups, and add every *reusable
pattern* (not the specifics) to a personal `patterns.md`: "time-series comp → check
future leakage in lag features"; "imbalanced target → stratify + PR-AUC not ROC";
"text comp → domain-pretrained embeddings beat generic." After ten write-ups you will
recognize the *shape* of a winning solution on sight — that recognition is what people
mistake for genius during the event.

### 1.4 The mock drill (T-7)

One full-speed rehearsal on a past competition with your team, half the real time
budget, real roles (pipeline owner / insight+viz owner / validation+red-team owner),
Claude in the loop, ending with an actual mock pitch. The drill's purpose is not the
score — it's finding where *your team* breaks: who bottlenecks, which handoff is
clumsy, where the checklist has a hole. Fix the system, not the people.

---

## Part 2 — The field game: playing against 7,000 teams, not against the dataset

Competitions are adversarial. The dataset is the board; the opponents are the field.
Three principles from competitive strategy:

### 2.1 The field clusters — expected value lives off-cluster

With 7,000 teams, the obvious approach will be executed *thousands of times*. If you
do the obvious thing well, you tie with a thousand teams and tie-breakers decide. The
finalists are teams that were **different in one defensible way**: an insight others
didn't surface, an external dataset others didn't join, a validation subtlety others
missed, a presentation others couldn't match. Budget explicitly for differentiation:
after your safe baseline exists, dedicate a workstream to the question *"what will
almost nobody else do?"* — and pressure-test it with Claude (battle card P4).

### 2.2 Portfolio strategy: one safe, one bold

When the format allows multiple final submissions (Kaggle allows two; many datathons
let you choose what to emphasize), the documented winning pattern is: **submission 1 =
your most robust CV-validated solution (the floor); submission 2 = your highest-upside
differentiated bet (the ceiling).** Never two variants of the same idea — if the idea
is wrong, both die; diversify across model classes / approaches for redundancy. The
Jigsaw shake-up winner famously made only two submissions total, both chosen by CV.
In a single-submission datathon, the same logic applies inside the deliverable: a
rock-solid validated core plus one bold, clearly-flagged insight — the core earns
credibility, the bold part earns the finalist slot.

### 2.3 Variance management: your position dictates your risk

Poker logic, directly applicable: **if your safe solution already puts you in
contention, reduce variance** (polish, verify, rehearse — don't gamble the floor).
**If you're clearly behind at the midpoint, increase variance** — a safe 500th place
is worth exactly as much as a risky 5,000th, so the bold bet becomes the rational
play. Most teams do the opposite: they gamble while ahead (hour-23 "improvements")
and play safe while behind (polishing a submission that cannot make top 10). Decide
your risk posture *explicitly* at the midpoint, out loud, as a team.

---

## Part 3 — Decision discipline under pressure (where good teams actually die)

Twenty years of watching teams lose distills to: they rarely lose on skill. They lose
on decisions made after hour 15, tired, attached, and unmeasured. Protocols:

- **Timebox every experiment before starting it.** "This idea gets 90 minutes; if CV
  hasn't improved by then, it dies." The timebox is set *before* you're attached.
- **The sunk-cost kill rule.** The question is never "how much did we invest in this
  approach?" — it's "knowing what we know now, would we *start* it?" If no, kill it
  now. Have Claude arbitrate: it has no ego in the approach (*"here's our position,
  time left, and score history — continue or kill?"*).
- **Keep a decision log** (five lines each: what we chose, why, what would change our
  mind). When you're exhausted at hour 20, the log stops you from re-litigating
  settled questions — re-litigation is the signature time-sink of tired teams.
- **Panic protocol — score won't move:** stop stacking ideas. Return to the last
  committed best, re-verify the harness itself (a silently broken validation wastes
  more hours than any bad model), then run the red-team prompt on your *assumptions*,
  not your code.
- **Panic protocol — the demo/pipeline breaks late:** revert to the last verified
  commit, cut the feature, never debug live into the deadline. This is why the battle
  card freezes at H22 — the freeze is a pre-commitment device against your own
  hour-23 self.
- **Sleep is a scoring feature in 48h events.** Staggered 3–4h shifts beat a team of
  four zombies; judgment quality at hour 40 is the difference between reading the
  rubric correctly and hallucinating requirements. The team that sleeps beats the
  team that doesn't, roughly always.

---

## Part 4 — The compounding loop (T+1 and the career game)

What makes someone win *many* of these instead of one:

1. **Post-mortem every event within a week** — exactly like `HACKATHON_ANALYSIS.md`:
   diff yourself against the winner, name the method gap, never the talent gap.
2. **Feed the arsenal.** Every gap becomes a checklist line, a skill, a `patterns.md`
   entry, or a `CLAUDE.md` standing order. The restart bug from your last competition
   should now be *impossible* for you — not because you'll remember, but because
   "model the real evaluator / ask the organizers" is in the checklist your tooling
   surfaces automatically.
3. **Study the winners' write-ups** of the event you just lost (§1.3) while the pain
   is fresh — retention is never better.
4. **Re-drill with the upgraded system** before the next event.

Loop iterations compound: each competition makes the *system* stronger, so your floor
rises monotonically even when individual results vary. That's the whole trick. The
20-year veteran isn't 20 years smarter — they're carrying 20 years of encoded
post-mortems into hour zero.

---

## Part 5 — The whole system on one page

**T-30 → T-7:** build/refresh the arsenal repo; Claude Code configured (CLAUDE.md,
skills, hooks); weekly winner-write-up study into `patterns.md`.
**T-7:** mock drill, half time budget, real roles, real pitch. Fix the system.
**T-0, H0–H3:** decode rubric (P0); interrogate organizers; recon; adversarial
validation; **harness committed before any modeling** (P1).
**H3–H12:** baseline node → measured one-change loop (P3), parallel worktree branches,
scores in commit messages; insight workstream in parallel (P4).
**Midpoint:** explicit risk-posture decision (ahead → reduce variance; behind → raise it).
**H12–H19:** hero chart (P5); cold red-team (P6); fix what's real.
**H19–H22:** narrative assembly (P7); reproducibility (P8); safe+bold portfolio chosen.
**H22:** freeze. Polish, verify, submit early, keep proof.
**Final round:** problem-first pitch, owned demo, rehearsed out loud, skeptical-judge
Q&A drill with Claude.
**T+1:** post-mortem → arsenal upgrade → the loop compounds.

**Sources**
- [Kaggle Grandmasters Unveil Winning Strategies (NVIDIA)](https://developer.nvidia.com/blog/kaggle-grandmasters-unveil-winning-strategies-for-data-science-superpowers/)
- [The Kaggle Grandmasters Playbook (NVIDIA)](https://developer.nvidia.com/blog/the-kaggle-grandmasters-playbook-7-battle-tested-modeling-techniques-for-tabular-data/)
- [Winning a Kaggle Competition with Generative-AI-Assisted Coding (NVIDIA)](https://developer.nvidia.com/blog/winning-a-kaggle-competition-with-generative-ai-assisted-coding/)
- [A Grandmaster's Guide to Machine Learning Challenges (Mindful Modeler)](https://mindfulmodeler.substack.com/p/a-grandmasters-guide-to-machine-learning)
- [Kaggle Model Selection Techniques (ML Journey)](https://mljourney.com/kaggle-model-selection-techniques-explained/)
- [Surviving a Kaggle Shake-up — fundamentals (Medium)](https://medium.com/global-maksimum-data-information-technologies/kaggle-handbook-fundamentals-to-survive-a-kaggle-shake-up-3dec0c085bc8)
- [Winning solutions of Kaggle competitions (compendium)](https://www.kaggle.com/code/sudalairajkumar/winning-solutions-of-kaggle-competitions)
