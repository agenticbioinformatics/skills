---
name: hackathon-cleanup
description: Use when the user asks to clean up, refactor, tidy, or productionize a hackathon project or GitHub repository — one that likely has dead code, unused variables, hardcoded paths, notebooks, unrelated experiments, or messy structure. Triggers on phrases like "clean up this repo/project", "clean up after the hackathon", "make this repo presentable", "productionize this codebase", "tidy up this project". Turns a messy hackathon repo into an organized, documented, minimally-tested pipeline while keeping every user instruction and its outcome logged in a prompt_history file.
---

# Hackathon Repo Cleanup

Turn a hackathon-grade GitHub repository — dead code, notebooks, hardcoded
paths, half-finished experiments — into a clean, documented, working
pipeline. Work in the phases below, in order. Do not skip understanding the
project before touching files: renaming or deleting things you don't
understand yet causes rework.

Every user request handled under this skill, and a summary of what changed,
must be appended to a `prompt_history` file (see Phase 6). Do this
throughout the session, not just at the end.

## Phase 0 — Scope the session

Confirm with the user (or infer from their first message) whether they want
the full cleanup described below or a subset (e.g. "just convert notebooks
to scripts", "just fix the README"). If they ask for "clean up this
project" with no further qualifiers, run the full sequence. If a request is
narrower, still append it to `prompt_history` (Phase 6) but only do the
work asked.

Before making destructive changes (deleting files, removing large data
directories, rewriting README from scratch), check `git status` if the repo
is a git working tree — uncommitted work should not be silently discarded.
If the repo isn't a git repository yet, mention that to the user; large
deletions are harder to reverse without version control.

## Phase 1 — Understand the project aim

Do not start reorganizing yet. First build a mental model of what this
project is for:

1. Read the existing README(s) fully, including any TODO/FIXME notes,
   half-written sections, or leftover conference/team names — these often
   contain the actual research question or task description even if poorly
   written.
2. Walk the whole repository tree (not just top-level) to see what exists:
   notebooks, scripts, data folders, configs, requirements files, Docker
   files, images/figures, checkpoints.
3. Skim the largest/most-central scripts and notebooks to see what they
   actually compute — the README may be stale or aspirational.
4. From this, write for yourself (and later reuse in the README overview)
   a hypothesis: what problem were the contributors addressing, what
   inputs/data did they use, what did they try as a solution, and what,
   concretely, does the code currently do end-to-end. If multiple
   plausible interpretations exist, list them and pick the one best
   supported by the code (most-developed path, most-referenced data, most
   complete notebook) — think step by step rather than guessing from the
   repo name alone.
5. If the aim is still genuinely ambiguous after this pass, ask the user
   rather than guessing — a wrong hypothesis here misdirects the whole
   cleanup (wrong things get merged, wrong things get deleted).

Keep this hypothesis explicit; it drives what counts as "dead code" or
"unrelated to the project aim" in later phases — code can look unused
without being genuinely off-topic, and code that is on-topic can still be
dead.

## Phase 2 — Reorganize the repository structure

1. Propose a logical directory layout based on what the code actually does
   (e.g. one directory per method/pipeline stage, plus `scripts/`, `src/`
   or a shared package, `docs/`, `tests/`). Mirror the structure to the
   project's actual components, not a generic template — a repo with three
   independent methods compared against each other should get three
   sibling directories, not one flat `src/`.
2. Move files into place. Rename files/functions/variables so names
   describe what they do, not their history ("script2.py", "final_v3.py",
   "test.ipynb" are candidates for renaming).
3. Merge near-duplicate scripts (e.g. two inference scripts differing only
   by hardcoded parameters) into one parameterized version.
4. Split scripts that do multiple unrelated things into separate,
   single-purpose scripts or modules.
5. Remove or relocate large input/output data files, model checkpoints,
   and generated images out of version control (delete if regeneratable,
   or note in README how to obtain/regenerate them) — hackathon repos
   routinely commit multi-GB data files that don't belong in git history
   going forward.
6. Confirm before deleting anything that isn't clearly disposable
   (failed-experiment notebooks, one-off exploration scripts) — ask the
   user if it's unclear whether something is a discarded dead end or
   in-progress work.

## Phase 3 — Clean the code

For every file kept:

- Remove dead code: unreachable branches, unused functions, commented-out
  blocks with no explanatory value, unused imports/variables.
- Remove code unrelated to the project aim established in Phase 1 (e.g.
  leftover scratch experiments, unrelated demo snippets).
- Replace hardcoded paths, hyperparameters, and magic constants with
  `argparse` CLI arguments and/or a `config.yaml`. Prefer one shared config
  format across the repo rather than a mix of `.yml`/`.yaml`/`.ini`/inline
  constants.
- Add comments only where the *why* isn't obvious from the code itself —
  do not narrate what the code does line by line.

## Phase 4 — Convert notebooks to scripts

Convert every `.ipynb` in the repo to a `.py` script:

- Strip notebook outputs and `!pip install` cells before/during
  conversion.
- Preserve only the logic that's actually part of the pipeline; drop
  exploratory/dead-end cells (per the Phase 1 hypothesis of project aim),
  after confirming with the user if it's unclear whether a cell was a
  meaningful experiment or scratch work.
- Turn the resulting script into a proper CLI entry point with `argparse`,
  not a linear script with hardcoded values.
- If a notebook mixes multiple pipeline stages (e.g. build data + train +
  evaluate), split it into separate stage scripts per Phase 2 rather than
  one long script.

## Phase 5 — Build a modular, portable, defensive pipeline

Where the cleaned-up code forms a pipeline (data → processing → output),
make the stages explicit and composable:

- Structure it as a sequence of scripts/functions with clear inputs and
  outputs on disk (e.g. `download_data.py` → `build_*.py` → `run_*.py` /
  `query_*.py`), each independently runnable via CLI arguments — mirror the
  "one script per stage, later stages take the previous stage's output
  path as an argument" pattern.
- Let the user substitute their own data or swap out a pipeline stage:
  accept input paths/formats as arguments rather than assuming one fixed
  dataset; if a stage wraps a model or algorithm, make it accept
  alternate implementations where feasible (e.g. a `--model` argument for
  any compatible model, an abstraction/interface for swappable backends)
  instead of hardcoding one choice.
- Platform portability: avoid OS-specific paths/commands; prefer
  `pathlib` and cross-Linux-distro-safe shell calls; avoid assumptions
  about specific hardware.
- Resource usage: auto-detect GPU vs CPU (e.g.
  `torch.device("cuda" if torch.cuda.is_available() else "cpu")`) and run
  correctly either way; avoid loading entire large datasets into memory
  when a streaming/chunked approach is feasible; save checkpoints/outputs
  in a device-agnostic form (e.g. CPU tensors) so results portable across
  machines.
- Defensive programming: validate inputs (file exists, expected columns
  present, non-empty results), fail with a clear error message instead of
  a stack trace on missing/malformed input, guard against divide-by-zero
  and other edge cases found while reading the code.
- Testing: add a lightweight `--test` flag to every runnable script that
  exercises it end-to-end on a small/synthetic slice of data using as
  little CPU/memory/GPU as possible, so the user can confirm the script
  works without running the full pipeline. Document this flag in the
  README (Phase 6).

## Phase 6 — Rewrite the README

Replace the README with a minimal version containing only:

1. **Overview** — 3 to 5 sentences: what the project does and what
   problem it addresses (from the Phase 1 hypothesis, confirmed against
   the actual code).
2. **Installation** — how to set up the environment (from
   `requirements.txt`/equivalent), including both CPU and GPU install
   instructions where relevant (e.g. separate `torch` CPU/CUDA wheel
   commands).
3. **Usage** — how to run the code, one entry per pipeline script/stage,
   each with a one-sentence description, the run command, and its
   `--test` flag noted.
4. **Data** — links to external data sources and tools used, how to
   download the data, and a short description of the data format(s)
   involved (e.g. columns, file types, expected directory layout).

Do not include a Methods/Results/Future-directions/References-to-
everything section — keep strictly to the four sections above unless the
user explicitly asks for more. Consolidate any install/run instructions
scattered across sub-directory READMEs into this top-level README; leave
sub-READMEs (if kept) only for supplementary reference detail (e.g. input
schema notes) with a pointer back to the top-level README.

## Phase 7 — Maintain prompt_history

Maintain a single plain-text file named `prompt_history` at the repo root
(not `.md` — plain text) that is the running log of this cleanup session.
For every user request handled under this skill, append an entry with:

- The prompt number and a short title.
- The request, quoted or paraphrased.
- A concise summary of the changes made in response (files
  added/removed/renamed, key logic changes) — specific enough that
  someone reading only this file understands what happened, without
  reproducing full diffs.

Append new entries as work happens; do not wait until the end of the
session to write the whole file at once, and do not overwrite prior
entries when adding new ones — extend the existing file if it already
exists. If the user asks to convert an existing `prompt_history.md` (or
similar) into plain text, do so and remove the old file.

## Notes on judgment calls

- If it's ambiguous whether something is dead code vs. an alternate code
  path the user still wants, ask rather than deleting.
- If the user's README-minimization request conflicts with details you
  think are valuable (e.g. results, references), default to what the user
  asked for; only push back once, briefly, if dropping something loses
  information not recoverable elsewhere (e.g. citations to papers the code
  implements) — a "References" section listing the tools/papers used is
  usually worth keeping even in a minimal README, distinct from a
  "Results" section of the winning submission, which is safe to drop.
- Keep changes reversible where possible (rename/move rather than delete)
  when unsure whether a file is disposable.
