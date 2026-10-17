# Evidence Bundles for LLM-Assisted MRI QC

**Branch:** [`evidence_bundle`](https://github.com/nipoppy/qc-studio/tree/evidence_bundle)  
**Branch snapshot:** October 5, 2026

## Purpose

The goal was a **companion for the reviewer, not an automated rater.** Deciding
whether a scan passes is a hard problem, and the right tool for it would be a
model trained for that task — a classifier or a deep-learning model — rather
than a general language model. Even purpose-trained models are not reliably
successful at it, so using an LLM to produce the final verdict is the wrong use
of it.

What an LLM can usefully do is surface the possible issues in a scan and say
what to follow up on: what to look at next, and what the reviewer should judge
for themselves. The QC decision stays with the human. Everything here follows
from that. The package is built so a reviewer can check the model's output
against the evidence it was given, and the response schema asks for flagged
metrics and recommended follow-up rather than a verdict to be trusted.

## Intended Product

On starting a session, a batch of subjects would be sent to the model. Each
subject's summary — the possible problems and the suggested next steps — would
be shown to the reviewer as a pop-up when they reach that subject.
Summarization comes first; a chat interface for asking follow-up questions about
a subject's data would be a later step.

The branch itself is a sandbox: exploratory infrastructure, not a production
feature, and not a claim of clinical validity.

## Architecture

A human reviewer weighs images, pipeline outputs, measurements, and acquisition
context together. The question was whether a model could summarize such a
package without inventing findings or treating one unusual metric as proof of
poor quality.

[Issue #24](https://github.com/nipoppy/qc-studio/issues/24) describes the
intended production architecture: MRIQC IQMs, BIDS sidecars, MRIQC Web API
reference statistics, pipeline-specific metrics, and eventually image slices as
sources, with a builder, typed evidence models, IQM and BIDS loaders, reference
helpers (`builder.py`, `models/evidence_bundle.py`, `loaders/`, `stats/`,
`renderers/markdown.py`, `renderers/json.py`), and a future Nipoppy layer. None
of this is a completed package; the sandbox validates the idea first.

## Format Validation

[Issue #33](https://github.com/nipoppy/qc-studio/issues/33) narrowed the first
step to package-format validation: can an LLM read the evidence and return
grounded JSON for examples meant to represent clear-pass, clear-fail, and
borderline cases? It was a feasibility experiment, and the issue was closed once
the text-package builder and sandbox experiment were done.

- The data is MR-ART from OpenNeuro. It is public, so overlap with model
  training data is possible.
- The IQM semantic explanations were generated with Codex assistance and need
  expert review.
- Markdown made packages easy to inspect; JSON may suit production better.
  Experiment tracking such as MLflow would help if the work grows.
- The saved IQM-only examples are three acquisitions of one subject, which
  cannot support an accuracy estimate.

## Sandbox Layout

Shared code is in `sandbox/qc_package.py`, semantic context in
`sandbox/iqms_context/`, and per-scan inputs in `sandbox/data/qc_inputs/`.
Cloud experiments are in `sandbox/cloud_model_experiments/IQMs_only_exp/` and
`.../image_evidence_exp/`; local ones in
`sandbox/local_model_experiments/exp_LLaVA-OneVision-0.5B/`,
`exp_LLaVA-OneVision-1.5-4B-Instruct/`, and `exp_LLaVA-OneVision-2-8B-Instruct/`.
Dependencies are in `sandbox/requirements-experiments.txt`. A directory for a
model setup does not mean its evaluation was completed; the recorded findings
cover the 0.5B and 4B runs.

History: an initial proof of concept, then generated IQM context, a local 0.5B
experiment, a larger 4B CPU experiment, and a layout cleanup separating helpers,
inputs, and outputs.

## Building a Package

The helper loads JSON and TSV/CSV inputs, selects BIDS and scanner metadata,
loads the scan's MRIQC IQMs, and reads reference tables. It filters reference
rows by scanner metadata and computes percentiles and z-scores, then attaches an
interpretation and its limitations to each metric. That per-metric context comes
from a dictionary built by extracting the MRIQC documentation with Codex: what
each metric means, which direction is concerning, and when it is informative. A
metric therefore reaches the model already interpreted rather than as a bare
number.

Highlighted metrics are those marked important in the context, at or below the
10th percentile, at or above the 90th, or with an absolute z-score of at least
1.5. These are experimental rules, not validated clinical thresholds or a
production outlier detector.

A package identifies the participant, session, modality, and acquisition. For
each highlighted metric it records value, percentile, z-score, direction of
concern, interpretation, caveats, and evidence strength. It adds task
instructions, rules discouraging unsupported claims, and a response schema
(`decision`, `confidence`, `summary`, `flagged_metrics`, `reasons`,
`recommended_followup`).

Reference-data quality is shared with the IQM viewer
([#82](https://github.com/nipoppy/qc-studio/issues/82)): malformed or extreme
reference values distort both plots and the percentiles and z-scores given to a
model. A production builder should reuse the same validated handling.

## IQM-Only Experiment

`PACKAGE_TEMPLATE.md`, `build_package.py`, and `run_experiment.py` assemble the
evidence, send it to Claude, and validate the response with Pydantic. Each scan
keeps `package/<scan>/PACKAGE.md`, `result.json`, and where applicable
`result_error.json`.

Recorded responses: `sub-000103_acq-standard_T1w` and
`sub-000103_acq-headmotion1_T1w` were `uncertain` (medium confidence);
`sub-000103_acq-headmotion2_T1w` was `fail` (high confidence). The more clearly
degraded example was easier to interpret and the others drew conservative
answers. These are qualitative observations, not measured performance.

## Image Evidence and Local Models

[Issue #40](https://github.com/nipoppy/qc-studio/issues/40) explores representing
3D MRI and surface data for models that mostly take 2D images: middle slices in
three planes, a 3x3 montage, maximum-intensity projections, ROI crops,
adjacent-slice comparisons, surface renderings, and overlays. The plan starts
with tri-slice images and montages, then labels and ROI crops; surface work is
deferred. The cloud design adds an `{image_section}` placeholder and an
`image_observations` field, asking the model to ground observations in visible
content and not let ambiguous impressions override strong quantitative evidence.

- **0.5B** (`llava-hf/llava-onevision-qwen2-0.5b-ov-hf`): useful for testing the
  input pipeline (compacted evidence, optional NIfTI middle slices), but
  responses contained unsupported findings, copied schemas, confused values with
  percentiles, and included unrelated text.
- **4B** (`lmms-lab/LLaVA-OneVision-1.5-4B-Instruct`): Apple MPS ran into
  memory and allocation problems, so a CPU path was used, which made each run
  slow enough to discourage repeated iteration. Output structure improved, but
  semantic problems remained: excessive reasons, missing follow-up,
  inconsistent `direction_of_concern`, and flagged metrics that did not match
  the prose.
- Issue discussion also recorded more coherent responses from online LLaVA (with
  reasoning errors) and more conservative IQM-only responses from Claude. This
  is not a controlled ranking.

Valid JSON is not the same as correct interpretation — which is the clearest
argument for the companion framing. A model that can produce a well-formed
response while misreading the metrics should be surfacing things for a reviewer
to check, not issuing ratings.

## Infrastructure Limits

The text stage worked. The Claude API read the Markdown package and returned a
structured response that Pydantic could validate, and the IQM-only findings rest
on that.

The next stage needed images, and that is where the work stopped. The vision
model the project intended to use was a local one, and I could not run it on my
own machine: MPS failed on memory and allocation, the CPU fallback was too slow
to iterate on, and the larger open-weight models were out of reach entirely. The
server intended for this was not yet available. The models that did run locally
were useful for validating the input pipeline but too small to reason about MRI
quality.

So the stopping point was infrastructure, not a design dead end. Anyone
continuing this should secure that compute first; without it the open-weight
side cannot produce a meaningful evaluation, whatever the prompt or package
design.

## Findings and Future Work

The result is a reusable structure for evidence preparation, prompt design,
output validation, and inspecting model failures. Markdown worked well for
debugging, and reference statistics and semantic context made the evidence more
informative, but both bring dependencies: clean reference data and
expert-reviewed explanations.

Next: move helpers into an `evidence_bundle/` package, define typed Pydantic
models, add a JSON renderer, test the percentile, z-score, and highlighting
logic, build a larger labeled evaluation set, track experiments, and choose an
image representation. Dashboard integration, a final model and prompt, and a
systematic evaluation of factual accuracy or workflow impact were not completed.
Any integration should stay advisory and leave the QC decision with the human
reviewer.

See the [GSoC Final Report](GSoC_FINAL_REPORT.md) and
[appendices](GSoC_REPORT_APPENDIX.md).
