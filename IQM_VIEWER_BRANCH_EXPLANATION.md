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

The first version added IQM distributions, a reference-data fetching script, and
reference-comparison plots. Feedback showed a metric dropdown was cumbersome and
an all-IQM view too dense, so I simplified to a metric grid within each source
tab.

Adapting to the pagination refactor changed the architecture. The top-level
`iqm_distributions` setting was MRIQC-specific and assumed groupings such as
`t1w` and `bold`. I replaced it with a per-task `iqm_path` list, following the
existing `svg_montage_path` pattern, so metrics are part of the configured QC
task. Each source appears in its own tab; non-MRIQC sources show as generic
plots or tables. A final cleanup separated reference-data helpers from
rendering and resolved the reference URL at call time.

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
