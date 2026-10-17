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

### From MRIQC-only to pipeline-agnostic

The first version supported MRIQC across its modalities, driven by a top-level
`iqm_distributions` setting that hardcoded groupings such as `t1w` and `bold`.
Review raised that the modality should follow from the QC task rather than being
chosen in the UI, which pointed at the deeper problem: the feature was
structured as an MRIQC add-on instead of a property of a QC task.

I replaced that setting with a per-task `iqm_path` list, deliberately mirroring
the existing `svg_montage_path` pattern so the configuration would be
predictable to anyone who already knew the project. The list points at
individual JSON files, which is what makes the panel pipeline-agnostic: a task
can declare several sources, each rendered in its own tab, so a pipeline other
than MRIQC is supported with its own metrics. Across tasks and sources the
viewer accommodates different metric sets, version-suffixed pipeline names
(matched in `is_mriqc_pipeline`), and different modalities.

MRIQC-like sources use curated metric groups; anything else falls back to
generic per-column display, so an unfamiliar pipeline is degraded rather than
unsupported. Multi-row tables render as distributions, while a single-row CSV or
JSON renders as a metrics table. DWI support then followed from extending the
group registry and inferring the modality, with no new display code — the
practical test of whether the refactor was worth doing.

### Reference data

Comparing a scan against a population needs that population, so I wrote the
code to collect reference distributions from the MRIQC endpoint
(`scripts/fetch_mriqc_reference.py`, documented in
`scripts/fetch_mriqc_reference.md`).

Collecting it was the first problem. The download ran for two full days without
finishing and the result was large, so I used the partial data as-is and only
for the structural modality. The loader in `ui/utils/reference_data.py` reads
filtered Parquet and caches it with a seven-day TTL
(`CACHE_TTL_SECONDS = 7 * 24 * 60 * 60`), capped at `MAX_REFERENCE_ROWS =
50_000`. It also normalizes scanner metadata before filtering — manufacturer
aliases (`siemens healthineers` → `siemens`, `ge medical systems` → `ge`) and
field-strength aliases (`3.0T` → `3`) — so a comparison is drawn against
similar acquisitions rather than against everything. The host itself is not
committed; `REFERENCE_DATA_URL` comes from the environment and is resolved at
call time rather than import time so it cannot go stale relative to `.env`
loading.

The real problem was what the data showed. The MRIQC population distribution
was markedly different from the distribution of the dataset being reviewed. A
comparison between two distributions that far apart is not informative, and is
actively misleading: a reviewer could doubt a perfectly good scan because it sits
outside a population it was never comparable to. Extreme values also stretched
the plot axes enough to make the dataset distribution itself hard to read, which
clipping only partly addressed.

So the capability is built and disabled. The "Dataset + reference" mode is
hidden in the merged UI, the infrastructure stays behind it, and
[#82](https://github.com/nipoppy/qc-studio/issues/82) tracks the cleanup and
testing needed before it returns. Part of that fix is a rule rather than more
data: if the two distributions differ that much, the viewer should not show the
comparison at all. Loading reference data successfully is not the same as
displaying it honestly.

### Display

Choosing one metric at a time from a dropdown was cumbersome, and an all-IQM
view was too dense to read. I tried an overview/detail layout with a scrollable
overview, then simplified to a metric grid inside each source tab so a reviewer
sees the whole picture first.

Plot changes followed the same concern. Box plots became violins, which show
the shape of a distribution rather than just its quartiles. Each tab got one
shared legend with consistent shapes and colours. The reference-comparison
design used split violins with dataset and reference on opposing sides. The
current-subject marker was scoped to the active run, with hover text naming it
— an unscoped distribution tells a reviewer nothing about which measurement
belongs to the scan in front of them, and an early review caught exactly that,
where the highlighted subject was not visible at all.

### Module boundaries

Review also moved the boundaries. I had put the reference maps and download
helpers in `ui/utils/data_loaders.py`; the argument against it was that a loader
module should not import `streamlit` and `requests` or hold configuration maps.
I agreed and moved both into a dedicated `ui/utils/reference_data.py`, which is
uncached by design — the caching lives with the viewer, in
`iqm_viewer.load_reference_iqm_for_subject`.

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
