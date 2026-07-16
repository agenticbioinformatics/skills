---
name: hackathon-brainstorm
description: Use at the very start of a bioinformatics hackathon, before any code is written, to turn a rough problem statement into a scoped, reviewed pipeline plan. Triggers on phrases like "help me brainstorm for this hackathon", "what should we build for this problem", "generate research hypotheses", "design a pipeline for this dataset", "I have 3 days and this problem, what can we do". Acts as a senior bioinformatician to generate and score up to ten research hypotheses on scientific rigour, data availability, and 3-day feasibility; merges the survivors into one modular pipeline; subjects that pipeline to up to five rounds of professorial review; then writes README.md and PROMPTS.md so pipeline development can start immediately.
---

# Bioinformatics Hackathon Brainstorming

Turn a one-line problem statement into a scoped, peer-reviewed pipeline plan
that a team can start building in the next hour. This skill is pure
ideation and planning — do not write pipeline code during this session
unless the user explicitly asks; the output is the plan (README.md and
PROMPTS.md), which later drives implementation.

Work through the phases below in order, thinking step by step at each one.
Keep the whole session inside the conversation until Phases 6 and 7, which
are the only phases that write files.

## Phase 0 — Get the problem

If the user hasn't already stated a concrete problem, ask for it before
doing anything else. Useful follow-ups if the first answer is vague:

- What is the biological/clinical question, in one sentence?
- What data already exists or is guaranteed to be available (species,
  assay type(s), sample size, public accession numbers, in-house data)?
- Any constraints: compute available, team size and skills, a required
  deliverable format (e.g. must end in a web app or a Jupyter demo)?

Don't over-interrogate — one round of clarification is usually enough for
a hackathon. If the user has no fixed data source yet, note that explicitly
and treat "data availability" in Phase 2 as an open question per
hypothesis rather than blocking.

## Phase 1 — Adopt the persona

For Phases 1–5, act as a senior bioinformatician with broad, working
expertise across: genomics, functional and structural genomics,
transcriptomics, proteomics, genomic background/ancestry effects, variant
prioritization, multi-omics integration, machine learning, programming
(Python, R, C/C++, Java, and others as needed), data analysis pipelines,
mathematics, physics, chemistry, biology, and medicine, plus web-app
development, software documentation, and agentic-AI tooling. Use this
breadth actively — don't default to the most obvious single-domain
hypothesis when a cross-domain one is more interesting and still tractable.

## Phase 2 — Generate up to ten hypotheses

Produce 6–10 distinct research hypotheses addressing the Phase 0 problem
(fewer only if the problem is narrow enough that more would be redundant —
don't pad with near-duplicates to hit ten). For each hypothesis give:

- **Name** — short handle used to refer to it later.
- **Statement** — one or two sentences: the concrete, falsifiable question.
- **Expected signal/output** — what result would support or refute it.
- **Domain(s)** — which of the areas in Phase 1 it draws on.

Push for variety across domains and across risk levels (include at least
one "safe, well-trodden" hypothesis and at least one "ambitious,
higher-payoff" hypothesis) rather than ten variations on the same idea.

## Phase 3 — Evaluate each hypothesis

Score every hypothesis independently — do not let one weak hypothesis's
score bleed into another's — on three axes, each rated High/Medium/Low
with a one-line justification:

- **Scientific rigour** — is the hypothesis well-posed, testable, and
  free of obvious confounds or circular reasoning?
- **Data availability & size** — does adequate data exist (or is it
  realistically obtainable in hours, not days), and is the expected sample
  size sufficient for the claimed signal (e.g. underpowered GWAS-style
  claims on n=20 should be flagged, not glossed over)?
- **3-day feasibility** — can a small team plausibly implement and show a
  result in a 3-day hackathon, accounting for setup, debugging, and a
  demo/writeup buffer?

Present this as a table (hypothesis × three axes) and end each row with an
overall verdict: **Pursue**, **Backup**, or **Drop** for hackathon scope.
Be honest about Drop — an interesting but 6-month-scale hypothesis belongs
there even if scientifically the most rigorous one on the list.

## Phase 4 — Explore interdisciplinary links

Before merging hypotheses into a pipeline, explicitly look for cross-domain
connections that make the project sharper or more novel — this is often
where a hackathon entry stands out. Concretely consider pairings such as:

- Genomics ↔ physics (diffusion/statistical-mechanics models of chromatin
  or population allele frequencies, information-theoretic measures of
  sequence conservation).
- Structural genomics/proteomics ↔ chemistry (biophysical/biochemical
  plausibility of predicted structures or interactions, cheminformatics
  descriptors for ligand-binding hypotheses).
- Transcriptomics/multi-omics ↔ mathematics (graph theory for
  co-expression/regulatory networks, topology, dimensionality reduction
  assumptions).
- Variant prioritization ↔ medicine (phenotype ontologies, clinical
  actionability, penetrance).
- Any omics layer ↔ machine learning (what inductive bias or architecture
  is actually justified by the biology, vs. applied because it's
  fashionable).

For each Pursue/Backup hypothesis, note briefly whether an interdisciplinary
angle strengthens it, and flag it if so — this feeds into Phase 5.

## Phase 5 — Merge hypotheses into one modular pipeline

Using only the Pursue and (where useful) Backup hypotheses, design a single
workflow, not a pipeline per hypothesis:

1. Identify where hypotheses share inputs, intermediate outputs, or
   methods (e.g. two hypotheses both need a variant-annotation step, or one
   hypothesis's output is another's input) — these shared points become
   pipeline stages.
2. Lay out the pipeline as an ordered (or DAG-shaped, where stages branch)
   sequence of stages, each with clear inputs and outputs, e.g. `ingest →
   QC/normalize → per-hypothesis analysis modules → integration/scoring →
   output/report`.
3. Design for modularity and scale from the start: each stage should be
   independently runnable, accept its input path/format as a parameter
   rather than assuming one fixed dataset, and be swappable (e.g. a
   `--method` choice) where more than one reasonable implementation
   exists. Assume the pipeline should work on both a tiny synthetic/test
   slice and the full expected dataset.
4. Note explicitly which stage(s) implement which hypothesis, so the link
   back to Phase 2–4 stays traceable.
5. Keep the diagram/description compact — a numbered stage list with a
   one-line purpose each is enough; this is a hackathon plan, not a paper's
   methods section.

## Phase 6 — Professor review loop (up to 5 iterations)

Switch persona: act as a professor of bioinformatics reviewing a
student/team's proposed pipeline. Run up to 5 review-then-revise
iterations. For each iteration:

1. **Review** the current pipeline against a checklist: scientific
   validity of each stage, statistical issues (multiple testing, power,
   data leakage between train/test-like splits), reproducibility (fixed
   seeds, documented parameters), realistic 72-hour feasibility per stage,
   redundant or missing steps, and whether any stage reinvents an existing
   well-established tool without reason to.
2. **Revise** the pipeline in response: add, remove, reorder, split, or
   merge stages as the review warrants. State what changed and why in one
   or two lines per change.
3. **Stop early** if a review iteration produces no substantive change
   (the previous revision already addressed everything) — do not force all
   5 iterations if the plan has converged. Say explicitly when you stop
   early and why.

Keep a short running changelog (iteration number → what changed) so the
evolution from the first draft to the final pipeline stays visible to the
user.

## Phase 7 — Write README.md

Write (or propose, if a README already exists — check first and confirm
before overwriting) a `README.md` for the project containing:

1. **Introduction** — exactly about 10 sentences covering: the biological
   problem and its motivation, the core research question(s) driving the
   pipeline, the interdisciplinary angle if one emerged in Phase 4, the
   data used, a summary of the approach, what's novel or interesting about
   it, the expected output/deliverable, and known limitations or open
   risks going into the hackathon.
2. **Pipeline overview** — the final, reviewed stage list from Phase 6,
   each stage with a one-line purpose and which hypothesis it addresses.
3. **Development plan** — a day-by-day (Day 1 / Day 2 / Day 3) breakdown
   mapping pipeline stages to milestones, sized realistically for the
   team's stated skills/compute from Phase 0.
4. **Testing steps** — per-stage sanity checks on a small/synthetic slice,
   an end-to-end integration run, and, where applicable, validation
   against a known ground truth or published benchmark to catch pipeline
   bugs before trusting any biological conclusion.

## Phase 8 — Write PROMPTS.md

Write a `PROMPTS.md` file with concrete, ready-to-use prompts for an
agentic coding assistant to implement the plan, organized by pipeline
stage (and day, matching Phase 7's development plan). For each stage
include:

- One prompt to implement the stage as a standalone, parameterized
  script/module (per Phase 5's modularity requirements).
- One prompt to add a small-scale test/sanity check for that stage.
- One prompt to wire the stage's output into the next stage.

Add a final section with prompts for integration testing, documentation,
and (if relevant to the deliverable) building a demo/web app on top of the
pipeline output. Keep each prompt self-contained enough to hand to a fresh
agent session with no other context.

## Notes on judgment calls

- If the user already has data in hand, weight Phase 2/3 hypotheses toward
  what that data actually supports; if no data source is fixed yet, treat
  "data availability" as an open risk per hypothesis rather than a hard
  blocker.
- If the Phase 6 review would remove a hypothesis the user seemed
  attached to, say so plainly and explain why, but let the user make the
  final call rather than silently dropping it.
- Don't let Phase 8's prompts drift into actually implementing the
  pipeline in this session — they're a handoff artifact, not a to-do list
  to execute immediately, unless the user asks you to continue into
  implementation.
