# Google Summer of Code 2026: QC-Studio Contribution Report

**Contributor:** Niusha ([mirhnius](https://github.com/mirhnius))  
**Project:** [QC-Studio](https://github.com/nipoppy/qc-studio)  
**Status checked:** October 12, 2026

## Summary

QC-Studio is a Streamlit application for reviewing neuroimaging data. It brings
MRI images, 2D montages, image quality metrics (IQMs), and structured ratings
into one review workflow. My work focused on making that workflow dependable:
showing quantitative evidence next to the images, and making sure a reviewer's
ratings and notes survive navigation, automatic advance, saving, and resuming.

- **Merged:** the IQM viewer (#22), the fix for ratings lost during autoplay
  (#76), SVG montage caching (#62), basic CI checks (#39), and terminology and
  documentation fixes (#34, #66).
- **In progress:** a draft Playwright browser-test framework (#88) and an
  evidence-bundle sandbox for LLM-assisted QC.
- **Reviews:** 14 PRs by other contributors, covering saving, checkpoints,
  filtering, multifacet ratings, caching, and documentation.
- **Upstream:** a Niivue issue (#314) whose requested loading events a
  maintainer confirmed are implemented.
- **Not delivered from the original proposal:** full Nipoppy layout and
  manifest integration, production outlier detection, and the AI-summary
  dashboard panel.

| Activity | Count (October 12, 2026) |
| --- | --- |
| PRs authored | 9: 6 merged, 2 open drafts, 1 closed prototype |
| Issues created in QC-Studio | 21: 9 closed, 12 open |
| PRs reviewed or commented on | 14: 13 with published activity, plus #20 (see [Limits](#limits-of-this-report)) |
| PRs merged / closed without merge under my account | 7 / 5 |
| Issues closed under my account | 11 |
| Upstream issues | 1 in niivue/niivue-vscode, closed |

Categories overlap: a PR I authored and merged is counted in two rows, and
closures include issues closed automatically by merging a linked PR. Individual
records are in the [appendices](GSoC_REPORT_APPENDIX.md).

## How the Scope Evolved

My proposal had three deliverables:

- A config-driven architecture using Nipoppy's `DatasetLayout` API, dynamic
  subject loading, unified pagination, and `manifest.tsv` integration.
- An IQM panel comparing subject metrics with local and MRIQC reference
  distributions, including rule-based outlier detection.
- An experimental AI panel displaying pre-generated summaries, with an
  evaluation of their factual accuracy and effect on the review workflow.

As the project progressed, priorities shifted toward making the existing
application reliable for real QC sessions. Rating persistence, navigation,
filtering, saving, checkpoints, and notes needed attention, so implementation,
code review, regression testing, and browser-test infrastructure became a
larger share of my work.

| Original deliverable | What I contributed | Outcome |
| --- | --- | --- |
| Nipoppy-integrated architecture | Per-task IQM source configuration, adapting the viewer to pagination, and reviews of cohort navigation, record identity, and resume behavior in others' PRs. | Configuration and integration work within the existing app. `DatasetLayout` and `manifest.tsv` integration was not implemented by me. Pagination implemented by others is credited to them. |
| IQM distributions and outlier detection | Merged IQM viewer with dataset distributions, scan-level tables, multiple sources, DWI support, and tests; reference-data infrastructure; metric highlighting in the evidence sandbox. | Dataset IQM panel delivered. Reference comparison is hidden pending #82. Production outlier detection is not complete. |
| AI summaries and evaluation | Structured evidence packages, model experiment scripts, response-schema validation, and qualitative findings on a small set of acquisitions. | Experimental sandbox only. No dashboard integration or systematic evaluation. |

Work outside the original proposal: the autoplay persistence fix, CI and local
checks, SVG caching, the draft Playwright framework, workflow reviews, and the
upstream Niivue report.

## 1. IQM Viewer

**Delivered:** [PR #22](https://github.com/nipoppy/qc-studio/pull/22), merged
September 10, 2026. Related: [#12](https://github.com/nipoppy/qc-studio/issues/12),
[#23](https://github.com/nipoppy/qc-studio/issues/23).
Details: [IQM Viewer Branch](IQM_VIEWER_BRANCH_EXPLANATION.md).

The IQM panel lets reviewers inspect quantitative measurements alongside the
images they are rating. It began with MRIQC reference distributions and became a
more general viewer after feedback and the pagination refactor.

- Replaced the MRIQC-specific top-level configuration with a per-task
  `iqm_path` list, following the existing montage configuration pattern.
- Each metric source is its own tab: multi-row sources show distributions;
  single-row CSV or JSON sources show a metrics table.
- Added curated MRIQC metric groups, generic handling of other sources, and DWI
  support.
- Redesigned plots around violin distributions, shared legends, and markers
  scoped to the subject and run being reviewed.
- Fixed scanner-metadata lookup, subject-switch widget state, and reference-data
  loading; separated reference-data helpers from rendering code.
- Added viewer, metric-group, and loader tests, with sample inputs and a DWI
  test configuration.

Reference-population comparison is hidden while reference cleaning and display
problems are addressed in [#82](https://github.com/nipoppy/qc-studio/issues/82).
The IQM/autoplay interaction is tracked in
[#77](https://github.com/nipoppy/qc-studio/issues/77).

## 2. Rating Persistence

**Delivered:** [PR #76](https://github.com/nipoppy/qc-studio/pull/76), merged
September 2, 2026, fixing [#73](https://github.com/nipoppy/qc-studio/issues/73).

During autoplay, a rating selected near the automatic-advance boundary could be
lost: the timer read the widget state after navigation had already changed the
active widget key. The fix saves ratings immediately through the widget's change
callback, guards against an empty widget read replacing an existing rating, adds
a grace period around automatic advance, and includes regression tests.

I also reported related notes and progress problems:
[#11](https://github.com/nipoppy/qc-studio/issues/11),
[#81](https://github.com/nipoppy/qc-studio/issues/81), and
[#84](https://github.com/nipoppy/qc-studio/issues/84). All are closed; my part
was the report and follow-up, and other contributors implemented the fixes.

## 3. Performance and Development Checks

- **SVG loading:** [PR #62](https://github.com/nipoppy/qc-studio/pull/62) added
  caching of SVG montage data to avoid repeated loading during widget
  interactions (closed [#43](https://github.com/nipoppy/qc-studio/issues/43)).
- **CI and local checks:** [PR #39](https://github.com/nipoppy/qc-studio/pull/39)
  added GitHub Actions checks for Black, Flake8, and codespell, with
  configuration and pre-commit setup that includes a UI test hook. A separate CI
  test workflow was left as follow-up.
- **Terminology:** [PR #34](https://github.com/nipoppy/qc-studio/pull/34) moved
  SVG-specific wording to montage terminology for broader 2D support.
- **Documentation:** [PR #66](https://github.com/nipoppy/qc-studio/pull/66)
  repaired broken links; I also reviewed and merged cleanup in
  [PR #67](https://github.com/nipoppy/qc-studio/pull/67).

## 4. Playwright Browser Testing

**In progress:** [PR #88](https://github.com/nipoppy/qc-studio/pull/88), open
draft on the `e2e` branch. Related: [#75](https://github.com/nipoppy/qc-studio/issues/75).
Details: [Playwright E2E Branch](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md).

Many bugs I found involved several steps: select a rating, navigate, return,
resume, or let autoplay advance. Mocked unit tests check the underlying logic,
but browser tests are needed to check how real Streamlit widgets and reruns
connect that logic to the workflow.

- Pytest fixtures launch the real app on a free port with temporary output
  directories and clean up the server.
- Shared widget selectors and helpers, with explicit handling of Streamlit
  reruns.
- Chromium and Firefox configured.
- 11 plain-English specifications (validation, ratings, navigation, overwrite
  protection, upload/resume, checkpoints, notes, autoplay).
- Six implemented test scenarios in four files: two smoke checks, two
  rater-form validation checks, notes persistence, and notes pausing autoplay.
- Project guidance for writing and reviewing tests with coding agents.

Status: initial Chromium smoke tests passed per the PR description; I have not
re-run the full suite on both browsers. Review feedback on rerun
synchronization, tracking of overwritten exports, and documentation consistency
is still to be addressed. Some tests disable Niivue, so they do not validate
volume rendering.

## 5. Evidence Bundles for LLM-Assisted QC

**Experimental:** [`evidence_bundle` branch](https://github.com/nipoppy/qc-studio/tree/evidence_bundle).
Related: [#24](https://github.com/nipoppy/qc-studio/issues/24),
[#33](https://github.com/nipoppy/qc-studio/issues/33),
[#40](https://github.com/nipoppy/qc-studio/issues/40).
Details: [Evidence Bundle Branch](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md).

I investigated packaging the evidence used in MRI quality review (scanner
metadata, MRIQC metrics, reference statistics, metric explanations) for a model,
then checking whether its response follows the requested format and the supplied
evidence.

- Shared helpers load the evidence and render packages with values, percentiles,
  z-scores, and a required response structure (decision, confidence, flagged
  metrics, reasons, follow-up), validated with Pydantic in the IQM-only
  experiment.
- Tested cloud IQM-only prompting and local LLaVA-OneVision setups (0.5B and
  4B), and explored 2D image representations for multimodal input.

The IQM-only packages used three MR-ART acquisitions from one subject (standard,
headmotion1, headmotion2); the saved Claude responses were `uncertain`,
`uncertain`, and `fail`. Other runs showed that valid JSON can still contain
wrong interpretations of the metrics. These are small exploratory runs with no
accuracy benchmark; the dataset is public, so training-data overlap is possible,
and the generated metric explanations need expert review. The result is a
reusable experimental structure and documented findings; production integration
and broader validation remain future work.

## 6. Code Review and User-Workflow Testing

I combined code reading with manual checks of real review sessions, focusing on
whether ratings and notes survive navigation, filtering, saving, and resuming. I
gave concrete failure cases and suggestions and, where possible, retested fixes
before approving. Fixes in others' PRs are credited to their authors. The
[review appendix](GSoC_REPORT_APPENDIX.md#appendix-c-review-and-comment-activity-on-others-prs)
and [review detail](GSoC_CONTRIBUTION_DOSSIER.md) list every comment.

**Larger reviews**

- [#95](https://github.com/nipoppy/qc-studio/pull/95) (multifacet ratings):
  found saved ratings being masked by all-`None` widgets, facet-name changes
  that broke saved matching, checkpoint lookup across the wrong rater or task,
  stale task navigation, and filename and wildcard-path edge cases. Retested
  task switching and wrong-task checkpoints, then approved; merged October 8.
- [#83](https://github.com/nipoppy/qc-studio/pull/83) (saving and checkpoints):
  reported filename and path inconsistencies, checkpoint confusion, altered
  leading-zero subject IDs, lost autoplay notes, and mislabeled counts; merged.
- [#74](https://github.com/nipoppy/qc-studio/pull/74) (filtering and
  navigation): found autoplay leaving the active filter and a fully reviewed
  cohort skipping the congratulations page; approved; merged.
- [#25](https://github.com/nipoppy/qc-studio/pull/25) (pagination
  compatibility): six inline comments on completed uploads, stale Continue QC
  widgets, Start Over records, duplicate handling, session assumptions, and
  display constants. The follow-up changes acknowledged the review; I merged it.
- [#105](https://github.com/nipoppy/qc-studio/pull/105) (documentation site):
  verified a strict Sphinx build, demo startup, CLI flags, and export columns;
  flagged README startup failures, CI coupling, pre-commit scope, and filename
  inconsistencies; on October 11 asked to remove the PyBIDS mention from the
  session docs. Open.
- [#111](https://github.com/nipoppy/qc-studio/pull/111) (memory optimization):
  noted two caching styles in two modules, different invalidation rules (the
  NIfTI key includes file mtime and size, the montage key does not), and a
  shared cached dict that is read-only only by docstring. The author's later
  commit "added shared cache module and decoupled load_montage_data to address
  PR comments" responds to the review. Open.

**Smaller reviews**

- [#10](https://github.com/nipoppy/qc-studio/pull/10): viewer layout; reported
  overlay colorbar stacking, checked the update, approved; merged.
- [#53](https://github.com/nipoppy/qc-studio/pull/53): keyboard shortcuts;
  raised conflicts with typing in notes, missing Play/Pause shortcuts, and
  narrow-screen label wrapping. Open.
- [#58](https://github.com/nipoppy/qc-studio/pull/58): review statistics;
  questioned whether recorded duration reflected review time, requested tests
  and average-per-subject and grouped-histogram views. Open, changes requested.
- [#17](https://github.com/nipoppy/qc-studio/pull/17),
  [#38](https://github.com/nipoppy/qc-studio/pull/38),
  [#67](https://github.com/nipoppy/qc-studio/pull/67),
  [#96](https://github.com/nipoppy/qc-studio/pull/96): documentation, fixture
  and citation PRs; reviewed and approved or closed as superseded.
- [#20](https://github.com/nipoppy/qc-studio/pull/20): initial pagination; see
  [Limits](#limits-of-this-report).

Two related reports: [#101](https://github.com/nipoppy/qc-studio/issues/101)
(Streamlit minimum version for `st.iframe`, closed after #95) and
[#112](https://github.com/nipoppy/qc-studio/issues/112) (overwrite warning
versus retained export rows, open).

This work informed the Playwright framework: notes persistence and
autoplay/notes interaction have tests, while save/resume and filtered completion
remain candidates.

## 7. Upstream Niivue Issue

**Reported and closed:** [niivue/niivue-vscode #314](https://github.com/niivue/niivue-vscode/issues/314).

QC-Studio needs to know when a volume is visible before starting a review timer.
I traced a missing signal to the Streamlit Niivue component and requested
separate `base_loaded` and `fully_loaded` events back to Python, explaining the
effect on autoplay and on slow readiness checks in browser tests. A maintainer
confirmed on September 29 that support was implemented in master and planned for
the next beta
([response](https://github.com/niivue/niivue-vscode/issues/314#issuecomment-5885125915)).
The maintainer implemented the change; adopting it in QC-Studio is a separate
step.

## What I Learned

- Tracing Streamlit behavior across widgets, reruns, session state, navigation,
  and saved records.
- Building a shared structure for browser tests instead of isolated scripts, and
  turning manual observations into reproducible issues, regression tests, and
  readable specifications.
- Helping clarify expected save and navigation behavior in
  [issue #89](https://github.com/nipoppy/qc-studio/issues/89#issuecomment-5674629745)
  (automatic snapshots were a proposal, not a delivered feature).
- Open-source maintenance: review follow-up, issue verification, merging, and
  closing resolved or superseded work.

## Remaining Work

| Work | Next step |
| --- | --- |
| Playwright framework, #88 / #75 | Resolve synchronization and export-tracking feedback, validate the tests, add save/resume scenarios. |
| IQM reference distributions, #82 | Clean and validate reference data before restoring the comparison control. |
| IQM/autoplay interaction, #77 | Investigate rendering and automatic advance with realistic matching inputs. |
| Montage task/run metadata, #61 (draft) | Finish it or document what remains. |
| Documentation website, #105 | Follow up on startup instructions, docs CI, sample-data hook scope, filename consistency. |
| Export semantics, #112 | Clarify merge versus replace and align the warning. |
| Niivue loading events | Verify release availability and use the events for timing and tests. |
| Evidence-bundle experiments, #24 / #40 | Expert-review metric context, improve reference data, define a larger evaluation. |

The original proposal's full Nipoppy layout/manifest integration, production
outlier detection, AI-summary dashboard panel, and workflow-impact evaluation
are not reported as delivered.

## Limits of This Report

- The inventory was checked against GitHub PR authors, issue authors, submitted
  reviews, comments, merge actors, closure events, and published branch contents
  as of October 12, 2026. It is not an estimate of hours and does not claim that
  every item fell within the official GSoC coding period.
- The original proposal summary and the account of the review of #20 come from
  me. I spent substantial time reviewing #20, but did not submit the feedback
  publicly, so no individual findings are attributed; the PR is merged and the
  pagination code is credited to its author.
- Playwright and evidence-bundle results are preliminary, as described in
  sections 4 and 5.

## Acknowledgements

Thank you to Nikhil Bhagwat and the QC-Studio contributors for feedback,
discussion of review workflows, and collaboration on testing and stabilizing the
application, and to the Niivue maintainers for responding to the loading-event
request.

## More Detail

[Appendices](GSoC_REPORT_APPENDIX.md) (all PRs, issues, reviews, merges, and
closures) · [Review and triage detail](GSoC_CONTRIBUTION_DOSSIER.md) ·
[IQM viewer](IQM_VIEWER_BRANCH_EXPLANATION.md) ·
[Playwright](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md) ·
[Evidence bundles](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md)
