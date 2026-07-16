# Competition Battle Card

The tactical companion to `DATATHON_PLAYBOOK.md` — what to actually *do and type*
during the competition. Three parts: the published evidence on how LLM-driven
workflows win competitions (this is the "how should I use Claude" answer, backed by
benchmarks, not opinion), an hour-by-hour battle plan, and a copy-paste prompt arsenal
including plotting prompts. Ends with the finals presentation playbook, because a
7,000→10 funnel means the final round is a *pitch* competition, not a code one.

---

## Part 1 — The benchmark evidence: HOW you drive the model is worth 10×

This is the most important research finding in all three documents, and it settles the
"was it the model or how we used it?" question with published data.

**MLE-bench** (OpenAI, ICLR 2025) tested AI agents on 75 real Kaggle competitions.
**AIDE** (Weco AI) is an agent scaffold purpose-built for such competitions. The results:

- The **same model (GPT-4o)** earned medals in **8.7%** of competitions inside AIDE's
  scaffold vs **0.8%** in a generic scaffold (MLAB) — **over 10× difference from the
  workflow alone, zero difference in the model.**
- AIDE's approach won **4× more medals** than the best *linear* agent (OpenHands) —
  meaning: iterate-and-branch beats straight-line "do step 1, then step 2, then submit."
- The best combination (o1-preview + AIDE) medaled in ~17% of competitions — scaffolding
  plus a strong reasoning model compounds.

**What AIDE actually does** — and what you should copy manually with Claude:

1. **Every candidate solution is a node in a tree.** Draft a solution, run it, score it
   against *validation*.
2. **The measured score decides everything.** Improve from the best-scoring node;
   abandon branches that underperform. No self-assessment, no "this looks better" —
   only the number.
3. **Debug and improve are separate moves.** A failing script gets a targeted fix; a
   working script gets *one* hypothesis-driven improvement at a time, so you always know
   what caused a change.

**Translation into your workflow (the "AIDE-manual" loop):**

- Build the evaluation harness FIRST (CV score, contract test suite — whatever the real
  judge is). This is the tree's scoring function; without it there is no search, only
  guessing. *(This is the deepest reason the harness-first rule from the playbook works.)*
- Make every Claude iteration end in a **measured score**, and paste that score back
  into the conversation. Claude improving against numbers converges; Claude improving
  against its own opinion of its code wanders.
- **Branch, don't marathon.** Keep your best-known-good solution committed. Try one
  improvement idea per branch/commit; keep it if the score improves, revert if not. Never stack
  five untested ideas.
- **One change per iteration.** The moment you let Claude change four things at once,
  you can no longer attribute the score movement, and the search degrades to noise.

Why this matters to *your* last loss: your team used strong models (Fable, Opus) in a
**linear, unmeasured** workflow — write fixes, believe them, upload. The winner ran a
measured loop (38 commits, each verified). The benchmark says that difference alone is
worth an order of magnitude. It wasn't the model. It was the loop.

---

## Part 2 — The battle plan (24h format; scale ×2 for 48h)

**H0–H1 — Decode & interrogate.** Read the full problem statement twice. Run the Phase-0
prompt (below). **Ask the organizers** the evaluation questions: exact metric? hidden
test split — how drawn? submission limit? restart/persistence? external data allowed?
what do finalists get judged on? Every answer is strategy.

**H1–H3 — Recon & validation design.** Load data, profile it (missingness, types,
distributions, target balance). Run **adversarial validation** immediately (train-vs-test
classifier: AUC ≈ 0.5 → CV trustworthy; AUC ≫ 0.5 → distribution shift, design around
it). Choose the CV split to match the structure (GroupKFold / time split / stratified).
**Commit the harness before any modeling.**

**H3–H5 — Baseline node.** Simplest credible end-to-end pipeline → CV score → submit
once to check CV↔leaderboard correlation. This is your tree's root. Commit it.

**H5–H12 — The measured loop.** ROI-ranked improvements, one per iteration, each ending
in a CV score pasted back to Claude. Keep/revert by the number. Meanwhile one teammate
starts insight mining (Phase-5 prompt) in parallel — the metric and the story are
separate workstreams.

**H12–H16 — Insight & hero chart.** Pick the ONE defensible finding. Build the hero
chart (finding-as-title, annotated, uncertainty shown). Run the confounder checks so the
finding survives questioning.

**H16–H19 — Red team.** Fresh Claude session / subagent: "assume ≥5 flaws, find them"
on both the pipeline and the narrative. Fix what's real. Re-run the harness green.

**H19–H22 — The submission itself.** Notebook runs top-to-bottom clean with fixed seed.
Narrative arc: problem → data reality → insight → validation → impact → recommendation.
Every chart title states its finding. Limitations section written *deliberately*.

**H22–H24 — Freeze & polish.** No new ideas after H22 (the classic self-inflicted loss
is a "quick improvement" at H23 that breaks the pipeline). Polish wording, verify the
submitted artifact one last time, submit early, keep the proof.

**Team-of-3/4 role split:** (1) pipeline & metric owner, (2) insight & visualization
owner, (3) validation/red-team & submission owner — rotating who drives Claude. The
worst configuration is everyone modeling and nobody owning validation or story.

---

## Part 3 — The prompt arsenal (copy-paste, adapted per phase)

These are deliberately *hard questions*, because strong models on weak prompts produce
weak work. Give Opus/Fable-class models the reasoning prompts; give fast models the
execution prompts.

**P0 — Rubric decode (reasoning):**
> Here is the complete problem statement and rules: [paste]. Extract: exactly what is
> scored and with what weights; deliverable format; disqualifiers; what the hidden
> evaluation most likely tests that the visible one doesn't. Then: a judge has 7,000
> submissions and 60 seconds for mine — what makes them stop scrolling? End with the
> 3 highest-leverage decisions for this specific competition.

**P1 — Validation design (reasoning — the championship prompt):**
> Design my validation before I model. Data: [description/sample]. Hidden test: [what
> you know]. (a) Which CV split matches this structure (KFold/GroupKFold/time) and why;
> (b) write the adversarial-validation check; (c) list every leakage vector in this
> dataset (target, group, temporal) and the test that detects each; (d) every way my
> local score could diverge from the hidden score, ranked by likelihood.

**P2 — Hypothesis-driven EDA (execution):**
> Given the goal [X], generate 10 ranked hypotheses about this data. For each: the ONE
> plot or statistic that confirms or kills it, and the code. I run, you interpret —
> only proceed on survivors.

**P3 — The measured loop (execution, repeated):**
> Current best: [approach], CV = [score ± std across folds]. Propose the single
> highest-ROI change. One change only. Predict its effect before I run it.
> *(After running:)* CV moved [old → new]. Keep or revert? Update your model of what
> works on this data, then next proposal.

**P4 — Insight mining (reasoning):**
> Beyond the metric: propose 3 non-obvious, decision-relevant findings this data could
> support. For each: the test that rules out confounding, leakage, and coincidence; and
> what a domain-expert judge would ask first. Which single finding, if it survives,
> most impresses — and why?

**P5 — Hero chart (execution — the "plot prompt"):**
> Build the hero chart for this finding: [finding]. Requirements: title states the
> finding as a sentence, not the variables; one comparison only; the key point annotated
> with an arrow/callout; competitors/context in muted gray, our result in one accent
> color; colorblind-safe; bars from zero, honest axes; uncertainty shown (CI band or
> error bars); n and source in a footnote. Then show me a 'default matplotlib' version
> side by side so I can see the difference. [If working in Claude Code: it has a
> `dataviz` skill — invoke it before chart code.]

**P6 — Cold red team (reasoning, FRESH session or subagent):**
> You have no stake in this work. Here is the spec, the code, and the claims: [paste].
> Assume at least 5 real flaws exist — leakage, invalid CV, confounded findings,
> misleading charts, unsupported claims, irreproducibility. Find them, ranked by how
> badly each would embarrass us in front of judges.

**P7 — Narrative assembly (reasoning):**
> Here are all findings and charts. Design the submission's story arc (problem → data
> reality → insight → validation → impact → recommendation). Pick the ONE hero chart.
> Rewrite every chart title as its takeaway. Cut everything that doesn't change what
> the judge believes. Then write the limitations section — honest but confident.

**P8 — Reproducibility (execution):**
> Make this run top-to-bottom clean: fixed seeds, pinned versions, no hidden state,
> README with exact reproduce steps. Then give me the atomic commit sequence with
> messages that read as an engineering log.

---

## Part 4 — The finals playbook (10 teams, one pitch)

If the funnel is 7,000 → 10, the final is decided by a *presentation*, and the judging
research is blunt: **a mediocre project with a great pitch regularly beats a great
project with a poor pitch**, and a strong project with a confusing demo loses to a
simpler one the judges understand.

**Structure (the consensus): Problem → Solution → Demo → Impact → Future.**
- **Open with the problem, one sentence, before you mention your product.** The sharper
  the problem, the more inevitable your solution feels. Make the judges share the
  frustration first.
- **One thing done excellently beats five things halfway.** Feature sprawl in a pitch
  reads as nothing-works-end-to-end.
- **The demo is a job, not an afterthought.** One person owns it. Pre-fill every form,
  mock every slow call, remove every place it could stall. Judges reward the working
  demo they saw, not the architecture they didn't.
- **Rehearse out loud, timed, at least once.** Teams that practiced look like they
  practiced; it changes how the content lands.
- **End with force.** Don't trail off into "…so, thank you." Land on the impact and
  what happens next. The work does not speak for itself — you speak for it.
- **Anticipate the Q&A with Claude:** *"You are a skeptical judge with 10 minutes of
  questions. Attack our method, our validation, our impact claim. Then coach the
  answers."* Confident, specific answers under questioning are scored — often
  explicitly in the rubric.

**Sources**
- [MLE-bench: Evaluating ML Agents on ML Engineering (OpenAI)](https://openai.com/index/mle-bench/) · [paper](https://arxiv.org/pdf/2410.07095)
- [AIDE: Human-Level Kaggle Performance (Weco AI technical report)](https://www.weco.ai/blog/technical-report) · [github](https://github.com/WecoAI/aideml)
- [How to Win a Hackathon: Notes From the Judging Table (JetBrains)](https://blog.jetbrains.com/ai/2026/06/how-to-win-a-hackathon-notes-from-the-judging-table/)
- [How to Create a Winning Hackathon Pitch (TAIKAI)](https://taikai.network/en/blog/how-to-create-a-hackathon-pitch)
- [Creating a 5-Minute Kickass Hackathon Pitch (Circles.Life)](https://medium.com/circleslife/creating-a-5-minute-kickass-hackathon-pitch-17cdcb42c3bc)
