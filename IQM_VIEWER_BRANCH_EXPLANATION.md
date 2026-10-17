# IQM Viewer Development

**Primary PR:** [#22](https://github.com/nipoppy/qc-studio/pull/22), merged  
**Branch:** [`iqm-viewer-fixes`](https://github.com/nipoppy/qc-studio/tree/iqm-viewer-fixes)  
**Branch snapshot:** October 5, 2026

## Overview

I developed the IQM viewer to bring MRIQC image quality metrics into the review
session, so a reviewer can examine measurements alongside the scan being rated.
It began as MRIQC reference-distribution plotting and became a general viewer
with per-task metric sources, distribution plots, scan-level tables, and
diffusion support. The core feature is merged in #22.

Related issues: [#12](https://github.com/nipoppy/qc-studio/issues/12) (IQM
distribution views, closed by the PR) and
[#23](https://github.com/nipoppy/qc-studio/issues/23) (documentation for the
MRIQC Web API reference-data script, also closed). Open follow-ups:
[#77](https://github.com/nipoppy/qc-studio/issues/77) (IQM rendering during
autoplay) and [#82](https://github.com/nipoppy/qc-studio/issues/82) (reference
filtering and display).

## How the Design Evolved

The first version did what #12 asked for: IQM distributions, a reference-data
fetching script, and reference-comparison plots, with a dropdown for choosing a
metric. Three problems that version exposed drove the rest of the design.

### 1. The reference data was not usable as delivered

Fetching the MRIQC Web API reference population ran for two full days without
completing, and the partial download was too large to commit to the repository.
I rewrote the lookup to read cached, filtered Parquet rather than querying the
API per view, which made the panel usable at all. Extreme values in the
population then stretched the plot axes far enough to make the dataset
distribution unreadable, so I added clipping for reference outliers.

The conclusion was that loading the data successfully is not the same as
displaying it honestly. Rather than ship a comparison I could not vouch for, I
filed [#82](https://github.com/nipoppy/qc-studio/issues/82), hid the
"Dataset + reference" mode, and left the infrastructure in place behind it. A
plausible-looking but wrong reference curve is worse for a reviewer than no
curve at all.

### 2. A metric dropdown did not match how reviewers work

Review feedback was that selecting metrics one at a time was cumbersome, while
an all-IQM view was too dense to read. I tried an overview/detail layout with a
scrollable overview, then simplified to a metric grid inside each source tab so
a reviewer sees the whole picture first.

Plot changes followed from the same concern. Box plots became violins, which
show the shape of a distribution rather than just its quartiles. Each tab got
one shared legend with consistent shapes and colours. The current-subject
marker was scoped to the active run, with hover text naming it — an unscoped
distribution tells a reviewer nothing about which measurement belongs to the
scan in front of them, and an early review caught exactly that case, where the
highlighted subject was not visible at all.

### 3. The configuration encoded MRIQC assumptions

The original top-level `iqm_distributions` setting hardcoded groupings such as
`t1w` and `bold`. Review raised that the modality should follow from the QC task
rather than being chosen in the UI, which pointed at the deeper problem: the
feature was structured as an MRIQC add-on instead of a property of a QC task.

I replaced it with a per-task `iqm_path` list, deliberately mirroring the
existing `svg_montage_path` pattern so the configuration would be predictable to
anyone who already knew the project. Each source became its own tab. MRIQC-like
sources use curated metric groups; others fall back to generic per-column
display, so a non-MRIQC pipeline is degraded rather than unsupported. DWI
support then followed from extending the group registry and inferring the
modality, with no new display code — the practical test of whether the refactor
was worth doing.

### Module boundaries

Review also moved the boundaries. I had put the reference maps and download
helpers in `ui/utils/data_loaders.py`; the argument against it was that a loader
module should not import `streamlit` and `requests` or hold configuration maps.
I agreed and moved both into a dedicated `ui/utils/reference_data.py`. A later
round caught that `URL_PARENT` was resolved at import time, before the app's
environment was in place, so resolution moved to call time.

## Viewer Behavior

- Multi-row tables show metric distributions; single-row CSV or JSON inputs show
  a metrics table.
- MRIQC-like sources use curated metric groups; others fall back to per-column
  display.
- Violin plots replace box plots, with one shared legend per IQM tab. The
  reference-comparison design used split violins, but that control is hidden in
  the merged UI pending #82.
- Markers identify the current subject, scoped to the active run or task, with
  hover text naming the run or source.
- The MRIQC group registry covers diffusion metrics, with DWI group tests, a
  sample input, the configuration `pipelines/mriqc/dwi_iqm_test_qc.json`, and a
  launch script `ui/dwi_iqm_test.sh`.

## Implementation

The main component is `ui/components/iqm_viewer.py`. Metric groups live in
`ui/utils/iqm_distribution_config.py` and reference loading and download helpers
in `ui/utils/reference_data.py`. Reference acquisition uses
`scripts/fetch_mriqc_reference.py` (documented in
`scripts/fetch_mriqc_reference.md`).

Several fixes concerned whether the viewer showed evidence for the correct scan:
scanner-metadata sidecar lookup (wrong-subject fallback, derivatives paths,
`dataset_dir` propagation), subject-marker scoping, widget-state resets when
switching subjects, and duplicate MRIQC tab labels. Others fixed stale viewer
tests, a display-mode radio hidden by the scrollable montage container, runtime
reference URL resolution, and lazy creation of the reference cache.

## Reference Data

During development the reference data proved large, API access slow, and extreme
values stretched plot axes, so loading it successfully was not enough to make
its display trustworthy. The helpers and fetching script exist, but the
comparison control is hidden until #82 is resolved. The evidence-bundle
experiments use the same reference percentiles and z-scores and depend on the
same cleaning.

## Tests

I added or updated tests for the viewer, metric-group configuration, diffusion
groups, and loading, mainly in `ui/tests/test_iqm_viewer.py` and
`ui/tests/test_iqm_distribution_config.py`, covering grouping, source loading,
widget state, and plot rendering paths. I have not re-run the full suite while
revising this document.

## Outcome and Remaining Work

Delivered: an IQM panel integrated with QC tasks, supporting multiple sources,
dataset distributions, scan-level tables, diffusion examples, subject markers,
and shared legends. Remaining: the autoplay interaction (#77), reference-data
cleaning (#82), handling of dense metric grids, more realistic matching sample
data, and refinement of reference-source management.

See the [GSoC Final Report](GSoC_FINAL_REPORT.md) and
[appendices](GSoC_REPORT_APPENDIX.md).
