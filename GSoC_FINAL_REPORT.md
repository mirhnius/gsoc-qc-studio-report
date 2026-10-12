# Google Summer of Code 2026: QC-Studio Contribution Report

**Contributor:** Niusha ([mirhnius](https://github.com/mirhnius))  
**Project:** [QC-Studio](https://github.com/nipoppy/qc-studio)  
**Status:** Contribution status checked on October 12, 2026

This report covers my recorded contributions to QC-Studio through October 12,
including early exploratory work. Open work is described at its current stage.

## Summary

- **Delivered and merged:** the IQM viewer (#22), the fix for ratings lost
  during autoplay (#76), SVG montage caching (#62), basic CI checks (#39),
  and terminology and documentation fixes (#34, #66).
- **In progress:** a draft Playwright browser-test framework (#88) and an
  evidence-bundle sandbox for LLM-assisted QC.
- **Reviews:** 14 PRs by other contributors, covering saving, checkpoints,
  filtering, multifacet ratings, caching, and documentation.
- **Not delivered from the original proposal:** full Nipoppy layout and
  manifest integration, production outlier detection, and the AI-summary
  dashboard panel.

## Project Overview

QC-Studio is a Streamlit application for reviewing neuroimaging data. It brings
MRI images, 2D montages, image quality metrics (IQMs), and structured ratings into
one review workflow. For this kind of application, showing the data is only part
of the problem: a reviewer also needs their ratings and notes to survive
navigation, automatic advance, saving, and resuming a session.

My contribution centered on making QC-Studio dependable for real review sessions.
I developed the IQM viewer, fixed ratings being lost during automatic advance,
and improved montage loading and development checks. These implementation
contributions form the core of my delivered work.

I also followed problems across the wider review workflow: whether the displayed
evidence belongs to the current subject, whether ratings and notes survive
navigation, and whether a saved session can be recovered correctly. This led to
detailed reviews of other contributors' changes, reproducible issue reports,
and a draft Playwright framework for checking real browser interactions. In
parallel, I explored evidence bundles for future LLM-assisted QC. Together, this
work connects feature development, reliability, testing, collaboration, and
research around the same application and its users.

## How the Scope Evolved

My original proposal aimed to turn the prototype into a robust,
Nipoppy-integrated application through three deliverables:

- A config-driven architecture using Nipoppy's `DatasetLayout` API, dynamic
  subject loading, unified pagination, and `manifest.tsv` integration.
- An IQM panel comparing subject metrics with local and MRIQC reference
  distributions, including rule-based outlier detection.
- An experimental AI panel displaying pre-generated summaries, with an
  evaluation of their factual accuracy and effect on the review workflow.

As the project progressed, development priorities shifted toward making the
existing application reliable for real QC sessions. Rating persistence,
navigation, filtering, saving, checkpoints, and notes required attention.
Implementation, code review, regression testing, and browser-test infrastructure
therefore became a larger part of my contribution. These changes supported the
proposal's broader goal of reducing the burden of manual QC and producing
reliable, documented ratings.

| Original deliverable | My verified contribution | Outcome relative to the proposal |
| --- | --- | --- |
| Config-driven, Nipoppy-integrated architecture | Added per-task IQM source configuration, adapted the viewer to pagination, and reviewed cohort navigation, record identity, and resume behavior in other contributors' PRs. | Delivered configuration and integration work within the existing app. A dedicated implementation of `DatasetLayout` and `manifest.tsv` integration is not established in my contribution record. Pagination implementations by other authors are credited to them. |
| IQM distributions and rule-based outlier detection | Merged the IQM viewer with dataset distributions, scan-level tables, multiple sources, DWI support, and tests. Developed reference-data infrastructure and explored metric highlighting in the evidence sandbox. | Delivered the dataset IQM panel. Reference comparison remains hidden pending #82. Sandbox metric highlighting is separate from production subject-level outlier detection, which is not claimed as complete. |
| AI summaries in the dashboard and evaluation | Built structured evidence packages, model experiment scripts, response-schema validation, and recorded qualitative findings on a small set of acquisitions. | Reached an experimental sandbox stage. Dashboard integration and a systematic evaluation of factual accuracy or workflow impact were not completed. |

The revised work also included deliverables that were not explicit in the
original proposal: the autoplay persistence fix, CI and local checks, SVG
caching, the draft Playwright framework, substantial workflow reviews, and the
upstream Niivue loading-event report.

The emphasis moved toward the complete review session, from inspecting a scan
and its metrics to recording a decision and recovering saved progress. That
explains why the final contribution includes work across several components and
PRs. The IQM implementation, persistence fix, workflow reviews, and testing
framework each address a different part of making that session dependable.

This report presents the original scope alongside the actual outcomes.

## Main Outcomes

- **Quantitative evidence in the viewer:** merged an IQM panel that supports
  multiple metric sources per QC task, distribution plots, individual-scan
  tables, and diffusion metrics.
- **Protection against lost ratings:** fixed ratings being dropped near
  automatic page advance and added regression tests.
- **Maintenance and performance:** merged SVG caching, clearer montage
  terminology, documentation repairs, and basic CI checks.
- **Browser-test infrastructure:** opened a draft PR with a real Streamlit
  test harness, readable behavior specifications, and initial workflow tests.
- **Review and issue reporting:** identified saving, checkpoint, filtering,
  completion, notes, and multifacet-rating problems in other contributors' work;
  retested fixes and reviewed documentation, onboarding, and runtime compatibility.
- **Research groundwork:** built an evidence-bundle sandbox and documented
  limitations of model responses on a small set of MRI QC examples.
- **Upstream contribution:** reported missing Niivue loading events; a maintainer
  confirmed that support was implemented upstream.

## How the Contributions Connect

| Role in the project | Concrete contribution | Purpose in the review workflow |
| --- | --- | --- |
| Feature development | IQM viewer in #22, including per-task sources, plots, and tests. | Bring quantitative evidence into the same application as visual review. |
| Reliability and performance | Rating-persistence fix in #76 and SVG caching in #62. | Preserve a reviewer's decision during automatic advance and avoid repeated montage loading. |
| Testing infrastructure | Basic checks in #39 and the draft Playwright framework in #88. | Check code changes consistently and make browser workflows repeatable to test. |
| Code review and collaboration | Findings on record identity, saving, filtering, and completion in #25, #74, #83, and #95; caching design review in #111; documentation and onboarding review in #105. | Check full-session behavior, retest fixes, and verify that setup instructions and development checks support new users and contributors. |
| Research and upstream investigation | Evidence-bundle sandbox and Niivue issue #314. | Explore structured QC assistance and identify dependency support needed for correct viewer timing. |

A concrete example of this connection is the autoplay rating bug: I reported
the observed failure in #73, traced the timing problem, implemented the fix in
#76, and added regression tests. Other findings became review feedback or browser
specifications, with their implementation status recorded separately. The
evidence-bundle experiments provide research groundwork alongside these changes
to the human review application.

## Contribution Snapshot

| Activity | Status checked on October 12, 2026 |
| --- | --- |
| PRs authored by my account | 9: 6 merged, 2 open drafts, 1 closed prototype |
| Issues created in QC-Studio | 21: 9 closed, 12 open |
| Review or comment contributions on others' PRs | 14 PRs: 13 with published activity, plus review work on merged PR #20 without public submission |
| PR merges recorded under my account | 7 |
| PRs closed without merging under my account | 5 |
| Issue closures recorded under my account | 11 |
| Upstream dependency issue | 1 in niivue/niivue-vscode, closed |

These categories overlap. For example, a PR I authored and merged appears in
both categories. Closure records also include issues closed automatically by
merging linked PRs. The [separate appendices](GSoC_REPORT_APPENDIX.md) give the
individual records and my role in each.

The review total includes my account of substantial time spent reviewing #20.
Its feedback was not publicly submitted; that publication status is recorded
separately from the review work and the PR's merged status.

## 1. IQM Viewer Development

**Delivered:** [PR #22](https://github.com/nipoppy/qc-studio/pull/22), merged
September 10, 2026. Related work:
[#12](https://github.com/nipoppy/qc-studio/issues/12) and
[#23](https://github.com/nipoppy/qc-studio/issues/23).

I developed an IQM panel so reviewers can inspect quantitative measurements
alongside the images they are rating. The work began with MRIQC reference
distributions and evolved into a more general viewer after feedback and the
pagination refactor.

- Replaced the MRIQC-specific top-level configuration with a per-task
  `iqm_path` list, following the existing montage configuration pattern.
- Made each metric source its own tab. Multi-row sources display distributions;
  single-row CSV or JSON sources display a metrics table.
- Added curated MRIQC metric groups, generic handling for other metric sources,
  and DWI support.
- Redesigned plots around violin distributions, shared legends, and markers
  scoped to the subject and run being reviewed.
- Fixed scanner-metadata lookup, subject-switch widget state, and reference-data
  loading behavior. Separated reference-data helpers from rendering code.
- Added or updated viewer, metric-group, and loader tests, with sample inputs
  and a DWI test configuration.

The merged feature provides dataset IQM viewing. Reference-population comparison
is hidden while reference cleaning and display problems are addressed in
[#82](https://github.com/nipoppy/qc-studio/issues/82). The IQM/autoplay interaction
remains tracked in [#77](https://github.com/nipoppy/qc-studio/issues/77).

Detailed explanation: [IQM Viewer Branch](IQM_VIEWER_BRANCH_EXPLANATION.md).

## 2. Rating Persistence and Workflow Reliability

**Delivered:** [PR #76](https://github.com/nipoppy/qc-studio/pull/76), merged
September 2, 2026, fixing
[#73](https://github.com/nipoppy/qc-studio/issues/73).

During autoplay, a rating selected near the automatic-advance boundary could be
lost. The timer previously read the widget state later, after navigation could
already have changed the active widget key.

- Saved ratings immediately through the widget's change callback.
- Guarded against an empty widget read replacing an existing rating.
- Added a grace period around automatic advance.
- Added regression tests for the persistence behavior.

I also reported related problems with notes and partial progress:
[#11](https://github.com/nipoppy/qc-studio/issues/11),
[#81](https://github.com/nipoppy/qc-studio/issues/81), and
[#84](https://github.com/nipoppy/qc-studio/issues/84). All three issues are now
closed. My contribution to those issues was reporting and follow-up; other
contributors also implemented the associated fixes.

## 3. Performance and Development Checks

- **SVG loading:** [PR #62](https://github.com/nipoppy/qc-studio/pull/62) added
  caching for SVG montage data to avoid repeated loading during widget
  interactions. It closed [#43](https://github.com/nipoppy/qc-studio/issues/43).
- **CI and local checks:** [PR #39](https://github.com/nipoppy/qc-studio/pull/39)
  added GitHub Actions checks for Black, Flake8, and codespell, their
  configuration, and pre-commit setup. The local pre-commit configuration also
  includes a UI test hook. A separate CI test workflow was left as follow-up in
  the PR description.
- **Terminology:** [PR #34](https://github.com/nipoppy/qc-studio/pull/34)
  updated SVG-specific wording to montage terminology for broader 2D support.
- **Documentation:** [PR #66](https://github.com/nipoppy/qc-studio/pull/66)
  repaired broken documentation links. I also reviewed and merged documentation
  cleanup in [PR #67](https://github.com/nipoppy/qc-studio/pull/67).

## 4. Playwright Browser Testing

**In progress:** [PR #88](https://github.com/nipoppy/qc-studio/pull/88), open
draft on the `e2e` branch. Related issue:
[#75](https://github.com/nipoppy/qc-studio/issues/75).

Many bugs I encountered involved several steps: select a rating, navigate,
return, resume, or let autoplay advance. Mocked unit tests can check the
underlying logic, but browser tests are needed to check how real Streamlit
widgets and reruns connect that logic to the user workflow.

- Built pytest fixtures that launch the real app, allocate a port, use temporary
  output directories, and clean up the server.
- Added shared widget selectors and interaction helpers, with explicit handling
  for Streamlit reruns.
- Configured Chromium and Firefox runs.
- Created 11 plain-English specifications covering validation, ratings,
  navigation, overwrite protection, upload/resume, checkpoints, notes, and
  autoplay.
- Added six test scenarios across four files: two smoke checks, two rater-form
  validation checks, notes persistence, and notes pausing autoplay.
- Added project-specific guidance for writing and reviewing tests with coding
  agents, including a specification template and shared conventions.

The specifications make expected behavior reviewable before a test is written.
They also give future contributors and coding agents a concrete target for new
tests. The 11 specifications do not mean all 11 workflows have implemented tests.

The PR description records successful initial Chromium smoke tests. Both browser
engines are configured, but I am not claiming that the full suite has passed on
both from this report preparation. The draft still needs follow-up on rerun
synchronization, tracking overwritten exports, and documentation consistency.
Some workflow tests disable Niivue to focus on notes and autoplay behavior, so
they do not validate volume rendering.

Detailed explanation: [Playwright E2E Branch](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md).

## 5. Evidence Bundles for LLM-Assisted QC

**Experimental work:**
[`evidence_bundle` branch](https://github.com/nipoppy/qc-studio/tree/evidence_bundle).
Related issues:
[#24](https://github.com/nipoppy/qc-studio/issues/24),
[#33](https://github.com/nipoppy/qc-studio/issues/33), and
[#40](https://github.com/nipoppy/qc-studio/issues/40).

I investigated how to give a model a structured package of the evidence used in
MRI quality review, then inspect whether its response follows the requested
format and refers to the supplied evidence.

- Built shared helpers to load BIDS/scanner metadata, MRIQC metrics, reference
  statistics, provenance warnings, and metric explanations.
- Rendered evidence packages with metric values, percentiles, z-scores, and
  instructions for a structured response.
- Required fields such as decision, confidence, flagged metrics, reasons, and
  recommended follow-up; added Pydantic validation in the IQM-only experiment.
- Organized reusable inputs separately from prompts, model responses, and
  validation outputs for each experiment.
- Tested cloud-based IQM-only prompting and local LLaVA-OneVision experiments,
  including 0.5B and 4B model setups. Explored 2D image representations for
  multimodal input.
- Recorded experiment findings and limitations in the related issues.

The IQM-only packages included three MR-ART acquisitions from the same subject:
standard, headmotion1, and headmotion2. The saved Claude responses were
`uncertain`, `uncertain`, and `fail`, respectively. Other experiments showed that
a response can have valid JSON while still interpreting metrics incorrectly.

These were small exploratory experiments, with no established accuracy benchmark.
The dataset is public, so training-data overlap is possible. The generated metric
explanations need expert review, and reference-data quality remains a dependency.
The contribution is a reusable experimental structure and documented findings;
production integration and broader validation remain future work.

Detailed explanation: [Evidence Bundle Branch](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md).

## 6. Code Review and User-Workflow Testing

I combined code reading with manual checks of real review sessions, focusing on
whether ratings and notes survived navigation, filtering, saving, and resuming.
My feedback explained concrete failure cases and suggested changes; where
possible, I retested the fixes before approval.

| PR | What I Did | Outcome |
| --- | --- | --- |
| [#10](https://github.com/nipoppy/qc-studio/pull/10): viewer overlays | Reviewed viewer-height/configuration behavior and reported overlay colorbar stacking. | Checked the update and approved; merged. |
| [#17](https://github.com/nipoppy/qc-studio/pull/17): documentation typo | Explained that earlier changes had already addressed the issue. | Closed the superseded PR. |
| [#20](https://github.com/nipoppy/qc-studio/pull/20): initial pagination | Spent substantial time reviewing the pagination code. | Merged; my feedback was not publicly submitted. |
| [#25](https://github.com/nipoppy/qc-studio/pull/25): pagination compatibility | Submitted six inline comments covering completed uploads, stale Continue QC widgets, Start Over records, duplicate handling, session assumptions, and display constants. Also suggested smaller PRs and coordinated the dependency on this merge for planned CI/test work. | Follow-up changes explicitly acknowledged my review; I merged the PR. |
| [#38](https://github.com/nipoppy/qc-studio/pull/38): montage terminology | Reviewed and approved fixture/documentation cleanup. | Later closed the generated PR. |
| [#53](https://github.com/nipoppy/qc-studio/pull/53): keyboard controls | Raised shortcut conflicts with notes, missing Play/Pause shortcuts, and narrow-screen label wrapping. | Open; feedback submitted. |
| [#58](https://github.com/nipoppy/qc-studio/pull/58): review statistics | Questioned whether recorded duration reflected actual review time, requested tests, and suggested average-per-subject and grouped-histogram views. | Open; changes requested. |
| [#67](https://github.com/nipoppy/qc-studio/pull/67): documentation cleanup | Reviewed and approved the cleanup. | Merged the PR. |
| [#74](https://github.com/nipoppy/qc-studio/pull/74): filtering and navigation | Found autoplay leaving the active filter and completion logic skipping the congratulations page for a fully reviewed cohort. Suggested checking full-cohort completion first and improving shared-helper placement. | Reviewed and approved; merged. |
| [#83](https://github.com/nipoppy/qc-studio/pull/83): saving and checkpoints | Reported filename/path inconsistencies, checkpoint confusion, altered leading-zero subject IDs, lost autoplay notes, and page counts mislabeled as QC records. | Review feedback contributed to the save/resume work; merged. |
| [#95](https://github.com/nipoppy/qc-studio/pull/95): multifacet ratings | Identified saved-rating masking, facet identity changes, checkpoint boundaries, stale task navigation, aggregate labels, filename parsing/casing, and wildcard-path handling. | Retested task switching and wrong-task checkpoints, then approved; merged October 8. |
| [#96](https://github.com/nipoppy/qc-studio/pull/96): citation metadata | Reviewed and approved the citation information. | Merged. |
| [#105](https://github.com/nipoppy/qc-studio/pull/105): documentation website | Verified a strict Sphinx build, demo startup, CLI flags, and export columns. Flagged README startup failures, CI coupling, pre-commit scope, filename inconsistencies, and documentation references. On October 11, asked for the PyBIDS mention to be removed from the session-list docs, since #111 removes PyBIDS session guessing. | Open; structural review and follow-up comment submitted. |
| [#111](https://github.com/nipoppy/qc-studio/pull/111): memory optimization | Reviewed the NIfTI and montage caches and raised three design points: two caching styles in two modules, different invalidation rules (the NIfTI key includes file mtime and size, the montage key does not), and a shared cached dict that callers are only asked in a docstring not to mutate. | Open; review submitted October 11. A later commit by the author, "added shared cache module and decoupled load_montage_data to address PR comments", responds to the review. |

The common thread was following a decision through the whole review session,
rather than checking a control in isolation. This work also informed the
Playwright framework: notes persistence and autoplay/notes interactions have
implemented tests, while save/resume and filtered completion remain candidates
for further coverage. Fixes in other contributors' PRs are credited to their
implementers.

Detailed comments, follow-up responses, and test limitations are recorded in the
[review appendix](GSoC_REPORT_APPENDIX.md#appendix-c-review-and-comment-activity-on-others-prs)
and [contribution dossier](GSoC_CONTRIBUTION_DOSSIER.md). Related runtime and
export reports are listed in the issue appendix:
[#101](https://github.com/nipoppy/qc-studio/issues/101), closed after the
Streamlit-version fix in #95, and
[#112](https://github.com/nipoppy/qc-studio/issues/112), open for the mismatch
between an overwrite warning and retained existing export rows.

## 7. Upstream Niivue Issue

**Reported and closed upstream:**
[niivue/niivue-vscode #314](https://github.com/niivue/niivue-vscode/issues/314).

QC-Studio needs to know when a volume is visible before starting a review timer.
I traced a missing signal to the Streamlit Niivue component and filed an upstream
report explaining both the autoplay problem and its effect on browser-test waits.

- Requested separate `base_loaded` and `fully_loaded` events back to Python.
- Explained why iframe initialization does not prove that the scan is visible.
- Connected the request to review time and slow readiness checks in e2e tests.
- A maintainer confirmed on September 29 that support was implemented in master
  and planned for the next beta release.

My contribution was the technical issue report and proposed API behavior. The
upstream maintainer implemented the change; adoption in QC-Studio is a separate
step. [Maintainer response](https://github.com/niivue/niivue-vscode/issues/314#issuecomment-5885125915).

## Collaboration and Learning

- Learned to trace Streamlit behavior across widgets, reruns, session state,
  navigation, and saved records.
- Developed plots and metric-source configuration while adapting the IQM
  feature to feedback and changes elsewhere in the repository.
- Learned Playwright and built a shared structure for browser tests instead of
  treating each scenario as an isolated script.
- Turned manual observations into reproducible issues, regression tests, and
  readable test specifications.
- Helped clarify expected behavior in
  [issue #89](https://github.com/nipoppy/qc-studio/issues/89#issuecomment-5674629745):
  immediate rating recording, deliberate export, recovery snapshots, and
  simpler navigation. Automatic snapshots were a proposal, not a delivered
  feature.
- Practiced open-source maintenance through review follow-up, issue verification,
  merging, and closing resolved or superseded work.
- Extended reviews to onboarding and documentation builds, checking that CLI
  examples, saved-file conventions, and CI behavior matched the application.
- Learned to connect feature implementation with its behavior over a full
  review session, and to distinguish my own fixes from findings that other
  contributors would implement.

## Remaining Work and Handoff

| Work | Current state and next step |
| --- | --- |
| Playwright framework, #88 / #75 | Open draft. Resolve synchronization and export-tracking feedback, validate the current tests, and complete high-value save/resume scenarios. |
| IQM reference distributions, #82 | Open. Clean and validate reference data before restoring the comparison control. |
| IQM/autoplay interaction, #77 | Open. Investigate rendering and automatic-advance behavior with realistic matching inputs. |
| Montage task/run metadata, #61 | Open draft. Review its current scope and either finish it or document what remains for another contributor. |
| Documentation website, #105 | Open. Follow up on startup instructions, documentation CI, sample-data hook scope, and filename consistency. |
| Export semantics, #112 | Open. Clarify whether saving merges existing records or replaces the file, and align the warning with that policy. |
| Niivue loading events | Upstream support confirmed. Verify release availability and integrate the events into QC-Studio timing and tests. |
| Evidence-bundle experiments, #24 / #40 | Sandbox work. Expert-review metric context, improve reference data, and define a larger evaluation before production integration. |

The original proposal also included full Nipoppy layout/manifest integration,
production outlier detection, an AI-summary dashboard panel, and a workflow-impact
evaluation. These are not reported as delivered.

The remaining issues already provide entry points for follow-up development.

## Acknowledgements

Thank you to Nikhil Bhagwat and the QC-Studio contributors for feedback on the
implementation, discussions about review workflows, and collaboration on testing
and stabilizing the application. Thank you also to the Niivue maintainers for
responding to the loading-event request.

## Supporting Evidence

The detailed PR and issue inventories, review activity, merge and closure
records, and supporting branch links are collected in the separate
[Contribution Appendices](GSoC_REPORT_APPENDIX.md). The
[full contribution dossier](GSoC_CONTRIBUTION_DOSSIER.md) provides additional
context for individual contributions.

The contribution inventory was checked against GitHub PR authors, issue authors,
submitted reviews, comments, merge actors, closure events, and published branch
contents. The original proposal summary was supplied by the contributor during
report preparation, as was the account of substantial review work on #20 whose
feedback was not publicly submitted. Activity totals combine that account with
the repository record available on October 12, 2026; they are not an estimate of
hours or a claim that every item occurred within the official GSoC coding period.
