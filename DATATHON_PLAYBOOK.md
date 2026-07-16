# The Datathon / Hackathon Championship Playbook

A general-purpose system for going deep in **any** competition — datathons, ML
challenges, open-ended data-storytelling, product builds, agentic-AI tracks — not just
bug-fixing. Written to answer three things: how to survive a 7,000→10 funnel, how to use
**visualization (plots)** as a weapon, and how to **drive Claude** as a reasoning +
verification partner instead of a code typist.

> Calibration note: no method guarantees winning a 7,000-team field — a lot of that
> funnel is skill *plus* variance. What a good method reliably does is put you in the top
> fraction of a percent by maximizing the things you control: validity, insight,
> communication, and rigor. Aim for "consistently top-10-worthy," not "100% guaranteed
> first," and you'll actually win more often.

---

## Part 1 — The 7,000 → 10 funnel: what *actually* gets you selected

You are not being graded like a homework problem. With thousands of submissions, judging
happens in passes, and each pass answers a different question. Optimize for the pass
you're currently facing.

**Pass 1 — the seconds filter (kills ~90%).** A judge skims your notebook/report/deck
for 30–120 seconds. They are not reading your code. They decide "credible or not" from:
does it look *complete*, is there *one clear result stated up front*, does a *visual*
communicate value immediately, is it *structured*. Most teams die here not because the
work is bad but because the submission doesn't *show* a result fast. **This is why plots
matter more than you think — they are your bandwidth into a time-starved brain.**

**Pass 2 — the rigor filter (kills most survivors).** Now a judge spends a few minutes.
Is the approach *valid*? Is the evaluation honest (no leakage, CV matches the real test)?
Is the insight *true*, not a spurious artifact? Is it *reproducible*? This is where teams
with a flashy chart but a broken validation scheme get cut — and where teams with a real,
defensible finding rise.

**Pass 3 — the differentiation filter (picks the 10).** Among the credible-and-rigorous,
who has a *non-obvious, decision-relevant insight* or a *materially better result*,
communicated so well the judge could re-explain it to their boss? Novelty + impact +
clarity + defensibility.

**The strategic consequence — where to spend effort:**
- Do **not** over-invest in the 4th decimal of the metric. Past a credible score, marginal
  accuracy rarely moves you up the funnel; **valid evaluation, one strong insight, and
  excellent communication** do.
- Budget roughly: **30% valid pipeline + honest validation**, **30% insight mining**,
  **30% communication (plots + narrative)**, **10% reproducibility/polish.** Most losing
  teams spend 80% on modeling and 5% on communication. That ratio is why strong analysts
  with weak decks lose to clear thinkers with a good chart.

---

## Part 2 — A taxonomy of competition types (win condition differs)

Match your effort to what the format actually rewards.

| Type | The real win condition | Where teams lose |
|---|---|---|
| **Bug-fix / black-box** (your last one) | Completeness + robustness under the grader's real conditions (concurrency, **restart**, edge cases) | Fixing symptoms not root causes; not modeling the grader |
| **Predictive ML / leaderboard** | A **validation scheme that matches the hidden test**, then honest incremental gains | Leakage; CV that doesn't mirror the private split; overfitting the public LB |
| **Open-ended data-storytelling** | One **non-obvious, true, actionable insight**, beautifully communicated | Aimless EDA; charts with no takeaway; unsupported claims |
| **Product / build hackathon** | A working demo that nails **one** user problem + a crisp story | Scope sprawl; nothing runs at the deadline |
| **Agentic-AI** | A robust, verifiable agent + a legible engineering trail | No verification loop; messy history; hand-wavy claims |

The **through-line** across all of them: (1) decode the exact win condition, (2) model how
you'll be evaluated *before* you build, (3) fix/find root causes, (4) verify against
evidence, (5) communicate the result so a tired judge gets it in seconds. Everything below
is that through-line made concrete.

---

## Part 3 — Plot mastery: how to use visualization to win

Plots are not decoration. In a datathon they are the **primary channel** through which a
judge experiences your analysis. Treat every chart as an argument.

### 3.1 The three jobs of a plot — keep them separate

1. **Discovery (EDA)** — for *you*. Fast, ugly, disposable. Distributions, missingness,
   correlations, leakage checks, target-vs-feature scans. Never ship these raw.
2. **Evidence (diagnostics)** — proves your work is *valid*. ROC/PR, calibration curve,
   confusion matrix, residuals, learning curves, SHAP/feature importance, CV-fold spread.
   These are how you survive Pass 2. Judges trust teams who show their model's failure
   modes.
3. **Persuasion (the hero chart)** — makes the judge *believe your finding*. Polished,
   annotated, one message. This is how you win Pass 1 and Pass 3.

### 3.2 The Hero Chart — design backward from it

Every winning submission has **one** image that, seen alone, conveys the contribution and
impresses. Before you plot anything, answer: *"If the judge gives me 10 seconds and one
picture, which picture wins?"* Build that first; let the rest of the submission support it.

A hero chart has: a **title that states the finding** (not the variable), a single clear
comparison, the key point **annotated**, everything else de-emphasized, and honest scales.

### 3.3 Chart selection — form follows the question

| The question | Use | Not |
|---|---|---|
| Compare a metric across categories | Horizontal **bar**, sorted by value | Pie beyond 2–3 slices |
| Change over time | **Line**, direct-labeled; many series → **small multiples** | One cluttered multi-line spaghetti |
| Relationship between two continuous vars | **Scatter** + trend/LOESS; overplotted → hexbin/alpha/sample | Opaque 100k-point blob |
| Distribution / spread | Histogram, **density/violin**, or **ECDF** (rigorous) | Bar of a mean with no spread |
| Part-to-whole | **Stacked bar** or small multiples | 3D pie |
| Many groups at once | **Faceting / small multiples** | Rainbow legend of 12 colors |
| Model quality | **ROC/PR, calibration, confusion, lift/gain** | A single accuracy number |
| Feature effects | **SHAP beeswarm**, partial dependence | "Correlation = 0.3", unexplained |

### 3.4 The craft rules (each one is a credibility signal)

1. **Title = the takeaway.** "Churn triples after 3 failed logins" beats "Churn vs
   failed_logins." A titled-with-insight chart does the judge's interpretation for them —
   the single highest-leverage habit.
2. **One message per chart.** If it needs a paragraph to explain, split it.
3. **Direct labeling > legends.** Label lines/bars at their ends; kill the legend hunt.
4. **Annotate the punchline.** Arrow/callout the one point that matters; gray out the rest
   so the eye lands where you want.
5. **Honest axes.** Bars start at 0. Never truncate the y-axis to fake a big effect — a
   sharp judge catches it and now distrusts everything. Log scale only when justified and
   labeled.
6. **Show uncertainty.** Error bars, CI bands, or at least *n*. Signals statistical
   maturity and is disproportionately persuasive.
7. **Reduce ink (Tufte).** Kill gridlines, borders, backgrounds, redundant ticks. Data
   first.
8. **Consistent, colorblind-safe palette.** Avoid red/green pairs and the `jet`/rainbow
   colormap. 2–4 colors, each with a *consistent meaning* across every chart, one accent
   color for the highlight.
9. **Sort by value, not alphabetically.** Sorting is free insight — the pattern pops.
10. **Units, source, and *n* on every chart.** Small footnote. Reads as professional.

### 3.5 Anti-patterns that instantly read "amateur"

Default matplotlib blue with no title/labels · pie charts with many slices · 3D anything ·
dual y-axes · rainbow (jet) colormaps · truncated axes · overplotted scatters · screenshot
of a table where a bar chart belongs · a chart with no stated takeaway.

### 3.6 A reusable, presentation-grade matplotlib theme (drop in once)

```python
import matplotlib.pyplot as plt
import matplotlib as mpl

mpl.rcParams.update({
    "figure.figsize": (8, 5), "figure.dpi": 130,
    "axes.spines.top": False, "axes.spines.right": False,
    "axes.grid": True, "grid.color": "#E6E6E6", "grid.linewidth": 0.8,
    "axes.titlesize": 15, "axes.titleweight": "bold", "axes.titlelocation": "left",
    "axes.labelsize": 11, "font.size": 11,
    "xtick.color": "#555", "ytick.color": "#555", "axes.edgecolor": "#BBBBBB",
})
# Colorblind-safe accents; use ONE accent for the highlight, gray for the rest.
ACCENT, MUTED = "#0072B2", "#BBBBBB"
```

### 3.7 A hero-chart pattern (title tells the story, one point annotated)

```python
fig, ax = plt.subplots()
ax.plot(x, baseline, color=MUTED, lw=2, label="_nolegend_")
ax.plot(x, ours, color=ACCENT, lw=3)
ax.text(x[-1], ours[-1], "  Our model", color=ACCENT, va="center", fontweight="bold")
ax.set_title("Our model cuts stockouts 38% vs the current rule")   # the FINDING
ax.set_xlabel("Week"); ax.set_ylabel("Stockout rate")
ax.annotate("Peak season:\nrule fails, ours holds", xy=(peak_x, peak_y),
            xytext=(peak_x-6, peak_y+0.1), color="#333",
            arrowprops=dict(arrowstyle="->", color="#333"))
fig.text(0.005, -0.02, "Source: 2024 sales panel · n=52 weeks", fontsize=8, color="#888")
fig.tight_layout()
```

### 3.8 The narrative arc — sequence plots as an argument

Problem → **Data reality** (1 chart: the key challenge/shape of the data) → **Insight(s)**
(hero charts) → **Method + validation** (diagnostics that prove rigor) → **Result/impact**
(the payoff chart) → **Recommendation**. No orphan charts; every plot advances the story.
If a chart doesn't change what the judge believes, cut it.

---

## Part 4 — Prompting Claude for open-ended datathons

The lesson from last time: you used Claude as an **executor** ("fix this," "plot this").
Champions use it as a **reasoning partner and a verification engine**. Give it the hard
questions, make it argue with itself, and make it prove things against evidence. Run these
phases in order; each has a copy-ready prompt.

### Phase 0 — Decode the win condition (do this before touching data)
> "Here is the full problem statement and rules. Extract EXACTLY what is scored and the
> weights; the required deliverable format; hard constraints and disqualifiers; and the
> single highest-leverage thing to nail. Then tell me how a judge with 7,000 submissions
> will actually skim mine, and what would make them stop and look."

### Phase 1 — Plan backward from the deadline
> "We have N hours and this deliverable. Give a time-boxed plan that guarantees a
> COMPLETE, submittable artifact by 70% of the time, leaving 30% for insight polish and
> the story. Define the minimum viable submission and the stretch goals separately."

### Phase 2 — Hypothesis-driven EDA (not aimless)
> "Given the goal, list 10 hypotheses about this data worth testing, ranked by potential
> impact on the outcome. For each: the exact check or plot that confirms/kills it. I'll run
> them and report back."

### Phase 3 — Validation strategy FIRST (the datathon 'restart bug')
This is the phase that most directly maps to how you lost last time. There, the unseen
condition was container restart; in a datathon it's the **hidden test set**.
> "Design a validation scheme that MIRRORS how the hidden/private set will score us.
> Check for: target leakage, train/test distribution shift, temporal ordering, group
> leakage (same entity in train and val). Tell me every way my local score could diverge
> from the real leaderboard, and how to detect each before I trust a single result."

### Phase 4 — Baseline fast, then improve by ROI
> "Give the simplest credible baseline I can run in 20 minutes. Then a ranked list of
> improvements by expected return (metric gain vs effort/risk). I'll climb the list and
> stop when returns flatten."

### Phase 5 — Insight mining (the differentiator)
> "Beyond the metric: what non-obvious, decision-relevant insight can we DEFEND from this
> data? Propose 3 candidate 'wow' findings, and for each, the test that proves it isn't a
> confounder, a coincidence, or leakage. Which one, if true, would most impress a judge?"

### Phase 6 — Cold red-team (fresh session or subagent)
> "Here is our full analysis and diff. Attack it: leakage, p-hacking, confounders,
> overfitting, unsupported claims, misleading charts, truncated axes. Assume there are at
> least 5 real flaws and find them."

### Phase 7 — Narrative + hero charts
> "Here are my findings and charts. Design the submission's story arc, pick the ONE hero
> chart, and rewrite every chart title to state its finding. Cut any chart that doesn't
> change what the judge believes."

### Phase 8 — Reproducibility + trail
> "Make the notebook run top-to-bottom clean with a fixed seed; write a short README;
> propose an atomic commit sequence with messages. I want a legible history, never an
> 'Add files via upload' dump."

### Model choice inside this
- **Opus / Fable (deep reasoning):** Phases 0, 3, 5, 6 — decoding the rubric, validation
  design, insight defense, red-teaming. This is where a strong model earns its cost. The
  mistake to avoid is a *strong model on a weak prompt* — don't waste it typing boilerplate.
- **A fast model:** Phases 2, 4, 7, 8 execution — running specified plots, applying a chosen
  diff, boilerplate. Cheap and quick once the thinking is done.

---

## Part 5 — The reusable championship loop

1. **Decode** the win condition and judging passes (Phase 0).
2. **Ask the organizers** the evaluation questions (hidden set? metric? constraints?
   restart? submission count?) and feed answers to Claude.
3. **Plan backward** to a guaranteed-complete submission (Phase 1).
4. **Validation strategy first** — build the honest eval before modeling (Phase 3). This is
   the single habit that would have changed last time.
5. **Baseline → ROI-ranked improvements** (Phase 4), iterating against your validation, not
   the public leaderboard.
6. **Mine one defensible insight** (Phase 5).
7. **Cold red-team** with a fresh Claude/subagent (Phase 6).
8. **Communicate**: hero chart + narrative arc + finding-stating titles (Parts 3 & 7).
9. **Reproducible, clean commit trail** (Phase 8), then submit at 70% time and polish.

---

## Part 6 — Your specific weaknesses → the fix (from the last competition)

| Weakness observed | Root habit | The fix to drill |
|---|---|---|
| In-memory fixes died on restart | Fixing symptoms, not root causes | Always ask Claude: "does this survive the *real* eval condition — restart / hidden set / edge case? Is the read path derived from the source of truth?" |
| Never anticipated the grader's restart | Not modeling the evaluation | Phase 0 + Phase 3 every time; ask organizers; run the adversarial-grader / hidden-set prompt |
| Missed export-404 and login-timing | No exhaustive rule×surface audit | Build the traceability grid (every rule/metric × every surface) and the cold red-team pass |
| "Add files via upload" history | No reproducible, tested loop | Work in git with Claude; one fix/experiment per commit; a living report |
| (Predicted) weak communication vs strong winner | Under-investing in the story | Spend ~30% on plots + narrative; design the hero chart first |
| Used Claude as a typist | Weak prompts to a strong model | Give Opus/Fable the hard questions (Phases 0/3/5/6); make it argue against itself |

---

## Part 7 — The one-paragraph version

Stop optimizing the metric's last decimal and start optimizing the funnel: decode exactly
how you'll be judged, model the hidden evaluation *before* you build (the datathon version
of the restart bug is a validation scheme that doesn't match the private set), find one
non-obvious insight you can defend, red-team it with a fresh Claude, and communicate it
with a hero chart whose title states the finding. Drive Opus/Fable as a reasoning and
verification partner on the hard questions, a fast model on the mechanical ones, and keep a
clean commit trail the whole way. Do that consistently and you won't win every time —
nobody does — but you'll be in the top-10 conversation far more often than a stronger
modeler who can't communicate or can't validate.
