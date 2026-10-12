# Evidence Bundles for LLM-Assisted MRI QC

**Branch:** [`evidence_bundle`](https://github.com/nipoppy/qc-studio/tree/evidence_bundle)  
**Branch snapshot:** October 5, 2026. Narrative revised October 10, 2026.

## Overview

I built the evidence-bundle branch as a sandbox for investigating whether the
evidence used in MRI quality control could be packaged for an LLM in a structured,
inspectable way. The work combines scanner metadata, image quality metrics
(IQMs), reference statistics, provenance warnings, and explanations of individual
metrics. It then asks a model to return a constrained response that can be checked
against the supplied evidence.

The branch is exploratory infrastructure rather than a production feature in
QC-Studio. Its purpose was to make an initially broad idea concrete: prepare
repeatable inputs, define the expected output, try different model setups, and
record where the approach breaks down. It does not establish clinical validity
or replace human QC.

## Motivation and Proposed Architecture

A human reviewer considers several kinds of evidence together, including images,
pipeline outputs, quantitative measurements, and acquisition context. The
evidence-bundle idea was to collect those sources for one scan and preserve their
context, rather than presenting a model with an unexplained list of numbers. The
central question was whether the model could summarize the package without
inventing findings or treating an unusual metric as conclusive evidence of poor
quality.

[Issue #24](https://github.com/nipoppy/qc-studio/issues/24) describes the broader
architecture. It proposes MRIQC IQMs, BIDS sidecars, MRIQC Web API reference
statistics, pipeline-specific metrics, and eventually selected image slices as
evidence sources. The proposed production structure separates a builder, typed
evidence models, IQM and BIDS loaders, reference-statistics helpers, and Markdown
and JSON renderers. It also leaves room for pipeline extensions and a future
Nipoppy integration layer.

The production module names proposed in that issue include `builder.py`,
`models/evidence_bundle.py`, `loaders/iqm_loader.py`,
`loaders/bids_loader.py`, `stats/mriqc_reference.py`, `stats/iqm_stats.py`,
`renderers/markdown.py`, and `renderers/json.py`. These describe the intended
architecture, not a completed production package. The sandbox branch was a
validation step before committing to that structure.

## Format Validation and Its Limits

[Issue #33](https://github.com/nipoppy/qc-studio/issues/33) narrowed the initial
work to package-format validation. The aim was to see whether an LLM could read
the evidence and return grounded JSON, using examples intended to represent
clear-pass, clear-fail, and borderline situations. This was a format and
feasibility experiment, not a clinical validation study. The issue was closed
after the initial text-package builder and sandbox experiment were completed.

The experiments used MR-ART data from OpenNeuro. Because the dataset is public,
overlap with model training data is possible. The IQM semantic explanations were
generated with Codex assistance and still require expert review. Markdown made
the packages easy to inspect during development, although JSON may be a more
compact production representation. Experiment tracking, such as MLflow, would
be useful if this work expands.

The saved IQM-only examples are three acquisitions from the same subject, not
three independent subjects. That small set is useful for inspecting model
behavior, but it cannot support an accuracy estimate or general conclusions
about model performance.

## Sandbox Organization and Development

The shared implementation lives in `sandbox/qc_package.py`, with semantic
context in `sandbox/iqms_context/` and per-scan inputs in
`sandbox/data/qc_inputs/`. Cloud experiments are separated into
`sandbox/cloud_model_experiments/IQMs_only_exp/` and
`sandbox/cloud_model_experiments/image_evidence_exp/`. This layout keeps
reusable evidence preparation separate from prompts, model responses, and
validation artifacts.

Local experiment directories include
`sandbox/local_model_experiments/exp_LLaVA-OneVision-0.5B/`,
`sandbox/local_model_experiments/exp_LLaVA-OneVision-1.5-4B-Instruct/`, and
`sandbox/local_model_experiments/exp_LLaVA-OneVision-2-8B-Instruct/`.
Dependencies are recorded in `sandbox/requirements-experiments.txt`. A directory
for a model setup does not, by itself, establish a completed evaluation; the
recorded findings discussed here concern the 0.5B and 4B experiments.

The development history moved from an initial evidence-bundle proof of concept
to generated IQM context, a local 0.5B experiment, and a larger 4B CPU experiment.
A later layout cleanup separated shared helpers, input data, and experiment
outputs. These changes made the exploratory work easier to inspect and continue.

## Building an Evidence Package

The shared helper loads JSON and TSV/CSV inputs, selects relevant BIDS and
scanner metadata, loads the scan's MRIQC IQMs, and reads reference IQM tables.
It filters reference rows by scanner metadata and computes percentiles and
z-scores. Semantic context is then attached to each metric so its numerical value
is accompanied by an interpretation and limitations.

The sandbox highlights metrics marked as important in the context, as well as
metrics at or below the 10th percentile, at or above the 90th percentile, or with
an absolute z-score of at least 1.5. These are experimental highlighting rules,
not validated clinical thresholds or a completed production outlier detector.
The resulting Markdown includes metadata, highlighted IQMs, and provenance
warnings.

Each package identifies the participant, session, modality, and acquisition.
For highlighted metrics, it records the value, percentile, z-score, direction of
concern, interpretation, caveats, and evidence strength. The package also provides
task instructions, rules intended to discourage unsupported claims, and a
required response schema containing `decision`, `confidence`, `summary`,
`flagged_metrics`, `reasons`, and `recommended_followup`.

Reference-data quality is a shared dependency with the IQM viewer.
[Issue #82](https://github.com/nipoppy/qc-studio/issues/82) concerns reference
cleaning and display correctness. Malformed or extreme reference values can
distort both a plot and the percentile or z-score context passed to a model.
A future production builder should reuse the same validated reference-data
handling rather than maintain a separate, inconsistent interpretation.

## IQM-Only Experiment

The IQM-only experiment uses `PACKAGE_TEMPLATE.md`, `build_package.py`, and
`run_experiment.py` to assemble evidence, send the package text to Claude, and
validate the response with Pydantic. Per-scan packages and results are stored as
`package/<scan>/PACKAGE.md`, `result.json`, and, where applicable,
`result_error.json`. Keeping both the input and output makes the experiment
inspectable rather than reducing it to a final pass/fail label.

The recorded responses for `sub-000103_acq-standard_T1w` and
`sub-000103_acq-headmotion1_T1w` were both `uncertain` with medium confidence.
For `sub-000103_acq-headmotion2_T1w`, the response was `fail` with high
confidence. The observations suggested that the more clearly degraded example
was easier to interpret, while the other examples prompted conservative
responses. These are qualitative findings from the saved examples, not evidence
of measured diagnostic performance.

## Image Evidence and Local Model Experiments

[Issue #40](https://github.com/nipoppy/qc-studio/issues/40) explores how to represent
3D MRI and surface data for models that primarily accept 2D images. Candidate
representations included axial/coronal/sagittal middle slices, a 3x3 montage,
maximum-intensity projections, ROI crops, adjacent-slice comparisons, surface
renderings, and overlays or contours. The staged plan began with tri-slice images
and montages, then labels and ROI crops, with more complex representations later.
Surface work was deferred.

The cloud image-evidence design adds an `{image_section}` placeholder and an
`image_observations` response field. It asks the model to ground observations in
visible content and use images as supporting evidence, without overriding strong
quantitative evidence on the basis of ambiguous impressions. This remains an
early design, not a settled image-representation strategy.

The local 0.5B setup used `llava-hf/llava-onevision-qwen2-0.5b-ov-hf`. Its script
reads `PACKAGE.md`, compacts the IQM evidence, optionally renders NIfTI middle
slices, and supplies text and an image to the model. It was useful for testing the
input pipeline, but the responses included unsupported findings, copied schemas,
confusion between metric values and percentiles, and unrelated text. It did not
provide reliable MRI QC reasoning in these experiments.

The larger setup used `lmms-lab/LLaVA-OneVision-1.5-4B-Instruct`. Apple MPS
encountered memory/allocation problems, so a CPU path was tested. The experiment
retained compact prompts, raw responses, validation JSON, and formatted results
when validation passed. Output structure improved, but semantic problems
remained: excessive reasons, missing follow-up content, inconsistent use of
`direction_of_concern`, and mismatches between flagged metrics and the concerns
described in prose.

The issue discussion also recorded more coherent responses from online LLaVA,
with reasoning errors still present, and more conservative IQM-only responses
from Claude. These observations reinforce the distinction between a response
that follows a JSON schema and one that interprets the evidence correctly.
They are not a controlled ranking of the models.

## Findings and Future Work

The main result is a reusable experimental structure for evidence preparation,
prompt design, output validation, and inspection of model failures. Markdown
worked well for debugging, while reference statistics and semantic context made
the evidence more informative. At the same time, those additions introduced
dependencies that need validation: clean reference data and expert-reviewed
metric explanations.

Further work would move reusable helpers into an `evidence_bundle/` package,
define typed Pydantic evidence models, and add a JSON renderer alongside
Markdown. Percentile/z-score calculations and highlighting rules need tests.
The research side needs a larger labeled evaluation set, explicit experiment
tracking, and a clearer choice of image representation before drawing
performance conclusions.

My contribution was turning an exploratory LLM-QC idea into concrete code,
repeatable packages, local and cloud experiments, schema checks, and documented
caveats. Production dashboard integration, a final model/prompt choice, and a
systematic evaluation of factual accuracy or workflow impact were not completed.
Any future integration should keep generated guidance advisory and preserve the
human reviewer's responsibility for the QC decision.

This work is summarized in the [GSoC Final Report](GSoC_FINAL_REPORT.md), with
linked contribution records in the [report appendices](GSoC_REPORT_APPENDIX.md).
