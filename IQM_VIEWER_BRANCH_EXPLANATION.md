# IQM Viewer Development

**Primary PR:** [#22](https://github.com/nipoppy/qc-studio/pull/22), merged  
**Primary branch:** [`iqm-viewer-fixes`](https://github.com/nipoppy/qc-studio/tree/iqm-viewer-fixes)  
**Branch snapshot:** October 5, 2026. Narrative revised October 10, 2026.

## Overview

I developed the IQM viewer to bring quantitative image-quality evidence into
QC-Studio's review workflow. Reviewers could already inspect images and montages,
but the measurements produced by MRIQC were not integrated into that experience.
The new panel lets a reviewer examine those measurements alongside the scan,
without navigating a separate set of outputs.

The work began as MRIQC reference-distribution plotting and evolved into a more
general viewer with per-task metric sources, distribution plots, scan-level
tables, and diffusion support. The core feature landed in
[PR #22](https://github.com/nipoppy/qc-studio/pull/22). Reference-data comparison
and autoplay reliability remain distinct follow-up areas rather than being
presented as fully resolved.

## Motivation and Related Issues

[Issue #12](https://github.com/nipoppy/qc-studio/issues/12) requested IQM
distribution views and comparison with MRIQC reference data to support scan
interpretation. It was closed by the IQM PR.
[Issue #23](https://github.com/nipoppy/qc-studio/issues/23) tracked documentation
for the MRIQC Web API reference-data script and was also closed during this work.

Two follow-up issues define the remaining boundaries.
[Issue #77](https://github.com/nipoppy/qc-studio/issues/77) tracks IQM rendering
problems during autoplay and automatic subject advance.
[Issue #82](https://github.com/nipoppy/qc-studio/issues/82) concerns reference-data
filtering and display correctness. Malformed or extreme reference values can
distort plots, so the merged direction retained dataset IQM viewing while hiding
reference comparison until that data handling is ready.

## How the Design Evolved

The initial version added IQM distributions, a reference-data fetching script,
and reference-comparison plots. Feedback showed that a metric dropdown was
cumbersome and that an all-IQM view could become too dense. I explored an
overview/detail layout, then simplified the display to a metric grid within
each source tab.

The architecture also changed as the feature was adapted to the pagination
refactor. The earlier top-level `iqm_distributions` setting was MRIQC-specific
and assumed groupings such as `t1w` and `bold`. I replaced it with a per-task
`iqm_path` list, following the existing `svg_montage_path` configuration
pattern. This made the metrics part of the configured QC task instead of a
separate hardcoded MRIQC feature.

A task can now refer to multiple IQM sources, and each source is displayed in its
own tab. This accommodates different pipeline inputs and allows non-MRIQC
sources to appear as generic plots or tables. The final cleanup separated
reference-data helpers from rendering, resolved the reference URL at call time,
and removed unnecessary imports, aliases, comments, and docstring clutter.

## Viewer and Plot Behavior

The viewer adapts its presentation to the input. Multi-row tables show metric
distributions, while single-row CSV or JSON inputs show a metrics table.
MRIQC-like sources use curated metric groups; other sources fall back to
per-column display. This keeps the panel useful for both dataset-level
comparison and individual-scan measurements.

I replaced box plots with violin plots and added a shared legend for each IQM
tab. The reference-comparison design used split violins, with dataset and
reference values on opposing sides, although that comparison control is hidden
in the merged UI pending reference cleanup. Dataset distributions remain part
of the delivered feature.

Markers identify the current subject where suitable data are available, with
scoping to the active run/task and hover text identifying the relevant run or
source. This matters because a distribution alone does not tell the reviewer
which measurement belongs to the scan they are currently rating.

## Diffusion Support and Sample Inputs

The work extended the MRIQC group registry to diffusion metrics and added DWI
group tests, sample input, a configuration, and a launch script. These additions
provided a concrete way to exercise diffusion IQMs rather than limiting the
viewer to the initial anatomical and functional examples.

The DWI configuration is `pipelines/mriqc/dwi_iqm_test_qc.json`, with a launch
script at `ui/dwi_iqm_test.sh`. Sample MRIQC inputs also support testing of
source loading and display behavior.

## Implementation and Reliability Fixes

The main viewer component is `ui/components/iqm_viewer.py`. Metric groups and
display configuration live in `ui/utils/iqm_distribution_config.py`, while
`ui/utils/reference_data.py` contains the separated reference-data loading
and download helpers. Reference acquisition is supported by
`scripts/fetch_mriqc_reference.py` and its documentation,
`scripts/fetch_mriqc_reference.md`.

Several fixes concerned whether the viewer showed evidence for the correct scan.
I corrected scanner-metadata sidecar lookup, including wrong-subject fallback,
derivatives-path support, and propagation of `dataset_dir`. I also corrected
subject-marker scoping, widget-state resets when switching subjects, and
duplicate MRIQC tab labels.

Other fixes addressed integration details: stale viewer tests, a display-mode
radio hidden by the scrollable montage container, runtime reference URL
resolution, and lazy creation of the reference cache. Moving reference loading
and caching out of unrelated loader modules made its responsibility clearer.

## Reference-Data Work

I added infrastructure for fetching and using MRIQC Web API reference
distributions. During development, the reference data proved large, API access
was slow, and extreme values stretched plot axes. Those observations showed that
loading the data successfully was not enough to make its display trustworthy.

The final state therefore separates infrastructure from the enabled feature.
Reference helpers and the fetching script exist, but the comparison control is
hidden pending [#82](https://github.com/nipoppy/qc-studio/issues/82). This
limitation also matters to the evidence-bundle experiments, which use reference
percentiles and z-scores and would need the same validated data handling.

## Testing and Validation

I added or updated tests for the viewer, metric-group configuration, diffusion
groups, and loading behavior. The principal test files include
`ui/tests/test_iqm_viewer.py` and
`ui/tests/test_iqm_distribution_config.py`. Sample inputs and the DWI test
script provide additional ways to inspect the feature.

The testing focus included metric grouping, source loading, widget state, plot
rendering paths, and diffusion support. These contributions establish test
coverage for the feature; this document does not claim a fresh full-suite test
run during its editorial revision.

## Outcome and Remaining Work

The delivered contribution is an IQM panel integrated with QC tasks, supporting
multiple sources, dataset distributions, individual-scan tables, diffusion
examples, subject markers, and shared legends. It spans configuration, interface
design, plotting, data loading, and tests, with the core work merged in #22.

The remaining work is explicit: investigate the IQM/autoplay interaction in #77,
clean and validate reference distributions in #82, improve handling of dense
metric grids, and use more realistic matching sample data. Reference-source
management may also need refinement. These follow-ups do not change what
landed, but they prevent the report from treating exploratory reference
comparison as a finished production capability.

My contribution brought quantitative evidence into the same review session as
visual inspection. It also showed how the original reference-plot idea could be
adapted into a more general, maintainable per-task component while recording the
parts that still require validation.

This work is summarized in the [GSoC Final Report](GSoC_FINAL_REPORT.md), with
linked contribution records in the [report appendices](GSoC_REPORT_APPENDIX.md).
