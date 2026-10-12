# Google Summer of Code Contribution Dossier: QC-Studio

As of: October 10, 2026  
Repository: `nipoppy/qc-studio`  
GitHub account: `mirhnius` / Niusha

Report draft: [GSoC Final Report](GSoC_FINAL_REPORT.md).

Linked inventory: [Report Appendices](GSoC_REPORT_APPENDIX.md).

## Executive Summary

My GSoC work on QC-Studio has covered implementation, testing infrastructure,
issue design, code review, and repository maintenance. The main contribution
areas are:

- QC review reliability and data safety: play-mode persistence, notes/save
  edge cases, checkpoint/export review, resume behavior, and completion logic.
- IQM visualization and reference-data work: IQM viewer development, MRIQC/DWI
  support, reference distribution experiments, and follow-up issues for IQM
  filtering/display reliability.
- Testing and developer infrastructure: CI checks, pre-commit/test gates, and a
  draft spec-driven Playwright e2e framework for Streamlit browser workflows.
- Upstream dependency feedback: reported a Niivue Streamlit-component issue
  needed for correct viewer timing, autoplay behavior, and e2e readiness checks.
- UI/UX workflow review: compact controls, autoplay/filter behavior,
  keyboard-shortcut layout, sidebar navigation, and save/checkpoint confusion.
- Evidence bundle / LLM-QC exploration: early evidence bundle format work,
  multimodal QC experiments, and model-evaluation notes.
- Project maintenance: documentation cleanup, issue triage, PR review, closing
  resolved issues, and merging/reviewing contributions from others, including
  documentation-site builds and onboarding instructions.

## Contribution Counts

- Authored PRs: 9 total
  - 6 merged: #22, #34, #39, #62, #66, #76
  - 2 open draft/in-progress: #61, #88
  - 1 closed WIP/prototype: #14
- Issues created: 21 total
  - 9 closed: #11, #23, #33, #35, #68, #73, #81, #84, #101
  - 12 open: #24, #40, #41, #69, #70, #71, #72, #75, #77, #78, #82, #112
- External PR review/comment contributions: 13
  - Published reviews/comments on 12 PRs: #10, #17, #25, #38, #53, #58, #67, #74, #83, #95, #96, #105
  - Review work on merged #20 without public submission: substantial time spent
    reviewing, based on my account of the work and the recorded review entry.
- Upstream dependency issue reported: niivue/niivue-vscode#314
- PRs merged by me: 7
  - #22, #25, #39, #62, #66, #67, #76
- PRs closed by me without merge: 5
  - #14, #17, #36, #37, #38
- Issues personally closed by me: 11
  - #12, #23, #33, #35, #43, #50, #56, #57, #63, #64, #73

## Main Technical Themes

### 1. QC Workflow Reliability

This includes bugs and reviews around saving ratings, preserving notes, avoiding
silent data loss, correct resume behavior, and making completion logic match the
real cohort state.

Related work:

- #76 fixed ratings being dropped near play-mode auto-advance.
- #73 tracked the play-mode save bug fixed by #76.
- #81 tracked partial QC progress loss and the need for save/checkpoint
  behavior.
- #83 review identified export/checkpoint/resume concerns, subject ID
  preservation problems, confusing checkpoint UX, save-path mismatches, and
  autoplay notes issues.
- #84 tracked notes being silently discarded on forced navigation.
- #95 review identified multi-facet rating persistence, checkpoint lookup,
  task-switching, slug casing, final rating labels, and export-path issues;
  retested fixes and approved before its October 8 merge.
- #101 reported a runtime dependency minimum incompatible with a newly used API;
  the maintainer marked it fixed by #95.
- #112 reports a mismatch between an overwrite warning and export behavior that
  retains existing rows outside the current batch.

### 2. IQM Viewer and Reference Data

This work focused on improving IQM plots, reference distributions, and MRIQC/DWI
support.

Related work:

- #14 prototyped IQM plots with web reference distributions.
- #22 merged IQM plots after pagination refactor.
- #23 tracked documentation for MRIQC Web API reference-data script.
- #82 tracks follow-up work for IQM reference-data distribution filtering and
  display.
- Local commits include work on IQM viewer redesign, DWI support, scanner
  metadata sidecar lookup, reference data caching, violin plots, legends, and
  subject-overlay markers.

### 3. Testing, CI, and E2E Infrastructure

This work adds project checks and starts browser-level testing for real
Streamlit workflows.

Related work:

- #35 proposed CI workflow for linting, spell check, and automated tests.
- #39 added CI formatting/lint/spelling checks and local pre-commit checks,
  including a UI test hook. A separate CI test workflow remained follow-up.
- #75 proposed Playwright end-to-end browser tests.
- #88 adds a draft spec-driven Playwright e2e scaffold for the real Streamlit
  app, including shared helpers, rerun waits, specs, smoke tests, and rater-form
  validation tests.
- niivue/niivue-vscode#314 reported the need for explicit volume-loaded events
  from the Streamlit Niivue component so QC-Studio can start autoplay timers
  only once an image is actually visible and avoid slow e2e readiness polling.

### 4. Upstream Niivue Integration Feedback

QC-Studio depends on the Niivue Streamlit component for image viewing, so some
workflow bugs require upstream support rather than only local QC-Studio changes.

Related work:

- niivue/niivue-vscode#314 requested `base_loaded` and `fully_loaded` events
  from the Streamlit component.
- The issue connected the upstream API gap to QC-Studio autoplay timing and e2e
  test readiness problems.
- A Niivue maintainer responded that the requested change was implemented in
  master and would be available in the next beta release.

### 5. Review, Triage, and Maintenance

This includes reviewing PRs from other contributors, catching workflow bugs,
helping close resolved issues, and merging approved changes.

Highlights:

- Reviewed #95 with multiple concrete code-level findings in multifacet rating
  behavior, followed up on filename/checkpoint/path edge cases, then retested
  task switching and wrong-task checkpoints before approving.
- Reviewed #105 documentation-site structure and onboarding; verified a strict
  Sphinx build and demo, and raised startup, CI, pre-commit, and filename issues.
- Reviewed #83 save/export/checkpoint/resume work and found user-facing and
  data-integrity issues.
- Reviewed #74 compact controls/autoplay/filter behavior and caught a filtered
  congratulations-page bug.
- Reviewed #25 pagination compatibility and test alignment, catching several
  completion/resume and stale state bugs.
- Spent substantial time reviewing the earlier pagination PR #20; its feedback
  was not publicly submitted while I was learning the GitHub review workflow.
  The PR is merged, and this is credited as review work rather than open work.
- Reviewed #58 detailed statistics and requested tests / timing correctness.
- Reviewed #53 hotkey support and raised UI/shortcut-layout concerns.
- Reviewed #10 overlay options and identified colorbar/UI behavior concerns.

## Authored Pull Requests

### #22: [ENH] IQM plots after pagination refactor

Link: https://github.com/nipoppy/qc-studio/pull/22  
State: merged  
Branch: `iqm-viewer-fixes`

Contribution:

- Continued and integrated IQM plot work after the pagination refactor.
- Connected IQM visualization to the cohort/page flow.
- Included supporting commits around IQM reference data, DWI support, viewer
  behavior, scanner metadata, plot layout, and tests.
- Closed #12 and #23 when the related IQM/reference documentation work landed.

### #34: [MAINT] Update Montage panel wording for 2D image support

Link: https://github.com/nipoppy/qc-studio/pull/34  
State: merged  
Branch: `update-panel-name-to-montage`

Contribution:

- Updated wording from SVG-specific language toward Montage panel language.
- Improved user-facing terminology for 2D image/montage support.
- Supported cleanup of related documentation and fixture terminology.

### #39: [ENH] Add basic CI checks

Link: https://github.com/nipoppy/qc-studio/pull/39  
State: merged  
Branch: `add-ci`

Contribution:

- Added initial CI/check infrastructure.
- Added Black/Flake8/codespell checks in CI, local pre-commit checks including
  a UI test hook, and README setup guidance.
- Left a separate CI test workflow as follow-up in the PR description.
- Closed #35.

### #62: [ENH] Cache SVG images

Link: https://github.com/nipoppy/qc-studio/pull/62  
State: merged  
Branch: `cache-svg-data`

Contribution:

- Added caching/refactoring for SVG montage loading.
- Reduced unnecessary reload/re-render behavior during UI interactions.
- Closed #43.

### #66: [MAINT] Fix broken documentation links

Link: https://github.com/nipoppy/qc-studio/pull/66  
State: merged  
Branch: `fix-links-md`

Contribution:

- Fixed broken links in project documentation.
- Closed #64.

### #76: [BUG] Fix play-mode ratings dropped near auto-advance

Link: https://github.com/nipoppy/qc-studio/pull/76  
State: merged  
Branch: `fix-73-play-mode-rating-persistence`

Contribution:

- Fixed a workflow/data-loss bug where ratings selected during play mode could
  be dropped near automatic advance.
- Added regression tests and clarified missing `qc_task` error behavior.
- Closed #73.

### #61: [ENH] Extract task and run metadata from SVG montage paths

Link: https://github.com/nipoppy/qc-studio/pull/61  
State: open draft  
Branch: `extract-task-run-from-montage`

Contribution:

- Draft work toward extracting `task` and `run` metadata from SVG montage
  paths.
- Includes helper/refactor work around BIDS IDs and montage identity.
- Supports future run/task-specific QC identity handling.

### #88: [MAINT] Spec-driven Playwright e2e testing + Claude Code skills/agents

Link: https://github.com/nipoppy/qc-studio/pull/88  
State: open draft  
Branch: `e2e`

Contribution:

- Added a Playwright-based e2e scaffold for testing the real Streamlit app in a
  browser.
- Added fixtures to launch the app, use free ports, isolate output directories,
  and wait for Streamlit reruns.
- Added shared selectors/helpers for Streamlit widgets.
- Added spec-first workflow under `e2e/specs/`.
- Added smoke tests, rater-form validation tests, notes-persistence tests, and
  an autoplay/notes interaction test.
- Added project-specific Claude/agent instructions for review and test-writing
  consistency.

Value:

- Complements `ui/tests/`, which mock Streamlit and cannot prove real UI wiring.
- Targets workflow bugs around forms, uploads, saving, resuming, filters,
  autoplay, and exports.

### #14: [WIP] IQM plots with web reference distributions

Link: https://github.com/nipoppy/qc-studio/pull/14  
State: closed WIP  
Branch: `grouped-iqm-distributions-12`

Contribution:

- Early prototype for IQM plots with web reference distributions.
- Later informed the more mature IQM work in #22.

## Issues Created

### #11: [BUG] Persist QC notes when saving TSV

Link: https://github.com/nipoppy/qc-studio/issues/11  
State: closed

Contribution:

- Identified a notes persistence/data-loss bug.
- Created an early workflow correctness issue around TSV saving.

### #23: [MAINT] Document MRIQC Web API reference-data script

Link: https://github.com/nipoppy/qc-studio/issues/23  
State: closed

Contribution:

- Tracked documentation work for MRIQC reference-data handling.
- Closed when related IQM/reference work landed.

### #24: Evidence Bundle

Link: https://github.com/nipoppy/qc-studio/issues/24  
State: open

Contribution:

- Opened the broader evidence-bundle idea for LLM-assisted QC.
- Anchors follow-up experiments around representing QC evidence for models.

### #33: Sandbox Experiment: Evidence Bundle Format Validation

Link: https://github.com/nipoppy/qc-studio/issues/33  
State: closed

Contribution:

- Defined and documented an initial evidence package format.
- Explored selected BIDS metadata, preselected IQMs, outliers, MRIQC flags,
  structured model-output expectations, and prompt rules.
- Discussed JSON vs Markdown tradeoffs and the need for experiment tracking.

### #35: [ENH] Add CI workflow for linting, spell check, and automated tests

Link: https://github.com/nipoppy/qc-studio/issues/35  
State: closed

Contribution:

- Proposed automated checks for code quality and tests.
- Closed by #39.

### #40: Sandbox Experiment: Multimodal Image Representation for Evidence Bundle

Link: https://github.com/nipoppy/qc-studio/issues/40  
State: open

Contribution:

- Documented experiments with local and online LLaVA models and Claude
  IQM-only reasoning.
- Compared model behavior on evidence bundles and IQM interpretation.
- Identified limitations in local small models and semantic consistency issues.

### #41: [ENH] Add lightweight logging for UI state transitions and QC workflow debugging

Link: https://github.com/nipoppy/qc-studio/issues/41  
State: open

Contribution:

- Proposed logging support for debugging Streamlit workflow/state transitions.

### #68: [ENH] Make QC review controls compact and accessible during long subject lists

Link: https://github.com/nipoppy/qc-studio/issues/68  
State: closed

Contribution:

- Tracked UI/UX improvements for long cohorts and compact controls.
- Related to #74.

### #69: [ENH] Add a confirm-only action for rating the current subject

Link: https://github.com/nipoppy/qc-studio/issues/69  
State: open

Contribution:

- Proposed a workflow improvement where users can rate without navigating.

### #70: [ENH] Remember QC display settings after leaving and returning to the review page

Link: https://github.com/nipoppy/qc-studio/issues/70  
State: open

Contribution:

- Tracked persistence of display preferences across review sessions.

### #71: [ENH] Keep pass/fail controls in a consistent visible location

Link: https://github.com/nipoppy/qc-studio/issues/71  
State: open

Contribution:

- Identified a usability issue around review controls moving out of view.

### #72: [BUG] Play mode should not scroll away from the montage or count loading time as review time

Link: https://github.com/nipoppy/qc-studio/issues/72  
State: open

Contribution:

- Raised workflow/timing concerns for play mode.

### #73: [BUG] Ratings selected during play mode are not always saved

Link: https://github.com/nipoppy/qc-studio/issues/73  
State: closed

Contribution:

- Identified the play-mode rating persistence bug.
- Closed by #76.

### #75: [ENH] Add Playwright end-to-end browser tests

Link: https://github.com/nipoppy/qc-studio/issues/75  
State: open

Contribution:

- Proposed e2e browser tests for real Streamlit workflows.
- Clarified that these should be opt-in/local or separately marked because
  browser setup and a real app server make them slower than unit tests.
- Related to #88.

### #77: [BUG] IQM panel is unreliable during autoplay / automatic subject advance

Link: https://github.com/nipoppy/qc-studio/issues/77  
State: open

Contribution:

- Tracked an IQM/autoplay reliability issue.

### #78: [BUG] Autoplay keeps running after navigating back to the landing page

Link: https://github.com/nipoppy/qc-studio/issues/78  
State: open

Contribution:

- Identified an autoplay state-management bug.

### #81: [BUG] Partial QC progress can be lost with no way to save it

Link: https://github.com/nipoppy/qc-studio/issues/81  
State: closed

Contribution:

- Raised data-loss/progress-loss concerns around save/checkpoint workflows.
- Related to the later save/export/checkpoint work in #83.

### #82: [ENH] Fix IQM reference data distribution filtering and display

Link: https://github.com/nipoppy/qc-studio/issues/82  
State: open

Contribution:

- Tracked correctness and display follow-up work for IQM reference data.

### #84: [BUG] Notes can be silently discarded on forced navigation

Link: https://github.com/nipoppy/qc-studio/issues/84  
State: closed

Contribution:

- Identified a forced-navigation notes loss bug.
- Related to #90.

### #101: [MAINT] Bump minimum Streamlit to 1.56 (st.iframe needs it)

Link: https://github.com/nipoppy/qc-studio/issues/101  
State: closed October 8, 2026

Contribution:

- Reported that `_render_image` now used `st.iframe` while the declared minimum
  Streamlit version still allowed installations without that API.
- Explained the resulting `AttributeError` risk and requested an appropriate
  dependency minimum.
- Nikhil marked the issue fixed by #95. My contribution was the compatibility
  report, not the dependency-change implementation or issue closure.
- [Maintainer resolution](https://github.com/nipoppy/qc-studio/issues/101#issuecomment-6066713911).

### #112: [BUG] Export warning says "overwritten" but rows from the old file are kept

Link: https://github.com/nipoppy/qc-studio/issues/112  
State: open

Contribution:

- Reported that the congratulations-page overwrite warning did not describe
  `save_qc_results_to_csv`, which merges existing rows and replaces duplicate
  record identities with their latest values.
- Gave a concrete example: saving participants A/B, then B/C to the same path,
  leaves A, updated B, and C rather than a fresh B/C-only export.
- Explained when this matters: changed participant lists or separate batches
  using the same rater/pipeline/task output path.
- Requested clearer merge wording or actual replacement, depending on the
  intended policy. This is a minor behavior/expectation mismatch, not a claim
  that the retained records are corrupt or that a fix has been implemented.

## Upstream Dependency Issue Reported

### niivue/niivue-vscode#314: Streamlit component: expose a "volume loaded" event back to Python

Link: https://github.com/niivue/niivue-vscode/issues/314  
State: closed

Contribution:

- Reported an upstream limitation in the Niivue Streamlit wrapper: Python only
  received `voxel_click` events and had no signal when the base volume or
  overlays had actually loaded.
- Explained the concrete QC-Studio impact: autoplay timers could start before
  the viewer was visible, reducing or eliminating real review time on slow
  loads.
- Connected the same upstream gap to e2e test reliability and slow readiness
  checks.
- Proposed two explicit event types: `base_loaded` and `fully_loaded`.
- A Niivue maintainer responded that the request was implemented in master and
  would be included in the next beta release.

## External PR Reviews and Code Review Contributions

### #10: added overlay options after streamlit update

Link: https://github.com/nipoppy/qc-studio/pull/10  
Author: `nikhil153`  
State: merged

Review contribution:

- Reviewed early overlay/UI behavior in `ui/layout.py`.
- Asked about the viewer-height slider behavior.
- Asked about `styled=True` for broader/faster config display.
- Reported the overlay colorbar stacking bug and confirmed later improvement.
- Approved after follow-up.

### #17: fix(docs): typo in dev_plan.md

Link: https://github.com/nipoppy/qc-studio/pull/17  
Author: `laurapiro17`  
State: closed

Review/maintenance contribution:

- Closed/commented that the issue had already been addressed in prior merged
  PRs.

### #20: started pagination

Link: https://github.com/nipoppy/qc-studio/pull/20  
Author: `nikhil153`  
State: merged

Review contribution:

I spent substantial time reviewing this early pagination PR. I did not submit
the feedback publicly because I was still learning how to post a GitHub review.
This is review work performed on a merged PR, not an open review task. GitHub's
unsubmitted review entry supports the existence of review activity, while my
account provides the context about the time invested and why it was not posted.
Individual findings are not attributed without supporting detail, and the
pagination implementation is credited to the PR author.

### #25: Fix pagination compatibility and test alignment

Link: https://github.com/nipoppy/qc-studio/pull/25  
Author: `GhazalehManj`  
State: merged

Review contribution:

- Reviewed duplicate QC record handling in `ui/managers/session_manager.py` and
  warned about silently keeping the latest duplicate.
- Questioned repeated `baseline -> ses-01` normalization in
  `ui/utils/cohort.py` and suggested documenting/centralizing it.
- Suggested replacing magic numbers with a named display-length constant in
  `ui/views/sidebar_cohort_nav.py`.
- Found a completion/resume bug in `ui/views/landing_page.py`: fully reviewed
  uploads could be sent back to page 1 instead of completed state.
- Found direct session-state mutation in `ui/views/congratulations_page.py`
  that could leave stale widgets after Continue QC.
- Raised that Start Over left `qc_records` intact, allowing stale ratings and
  notes to leak into a fresh run.
- Suggested smaller PRs grouped by behavior for future large changes.
- Explained how merging this PR would unblock planned CI/test workflow work,
  including a proposed split between lightweight and slower/data-dependent
  checks. That proposal is not credited as implemented in #25.
- Thomas explicitly acknowledged my review and reported corresponding changes
  to completion/resume, Continue QC resets, clearing records on Start Over,
  session handling, and duplicate-export reporting. Ghazaleh accepted the
  updates. Their implementation work is credited to them.
- Merged the PR.

Evidence:

- [Submitted review with six inline comments](https://github.com/nipoppy/qc-studio/pull/25#pullrequestreview-4397400200).
- [Completed-upload finding](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370573536).
- [Continue QC state-reset finding](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370631538).
- [Start Over record-retention finding](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370687500).
- [Follow-up implementation acknowledgement](https://github.com/nipoppy/qc-studio/pull/25#pullrequestreview-4460976114).
- [Author acceptance](https://github.com/nipoppy/qc-studio/pull/25#pullrequestreview-4461353279).
- [PR-scope feedback](https://github.com/nipoppy/qc-studio/pull/25#issuecomment-4587917864).
- [CI/test coordination](https://github.com/nipoppy/qc-studio/pull/25#issuecomment-4587074842).

### #38: Align remaining fixture docs terminology with 2D montage naming

Link: https://github.com/nipoppy/qc-studio/pull/38  
Author: Copilot agent  
State: closed

Review/maintenance contribution:

- Approved/managed cleanup related to montage terminology.
- Closed stale generated PR work.

### #53: [WIP] Hotkey support

Link: https://github.com/nipoppy/qc-studio/pull/53  
Author: `SRSteinkamp`  
State: open

Review contribution:

- Asked whether `Shift+1/2/3` was needed to avoid conflicts with typing in the
  Notes field.
- Asked why Play/Pause lacked keyboard shortcuts while arrows handled
  Prev/Next.
- Flagged narrow-window layout issues where shortcut hints caused button labels
  to wrap vertically.

### #58: UI detailed statistics

Link: https://github.com/nipoppy/qc-studio/pull/58  
Author: `manonheff`  
State: open

Review contribution:

- Requested tests.
- Tested locally and agreed with timing concerns: recorded duration appeared to
  measure time since last rerun/page change, not actual review time.
- Asked for average time per subject, consistent with the issue proposal.
- Gave feedback that grouped histogram visualization was more intuitive.
- Requested changes.

### #67: [DOC] Fix/llm doc slop

Link: https://github.com/nipoppy/qc-studio/pull/67  
Author: `nikhil153`  
State: merged

Review/maintenance contribution:

- Approved and merged documentation cleanup.
- Closed #63.

### #74: [ENH] Make QC review controls compact and accessible during long subject lists

Link: https://github.com/nipoppy/qc-studio/pull/74  
Author: `GhazalehManj`  
State: merged

Review contribution:

- Reviewed compact controls/sidebar behavior.
- Asked whether the search bar from earlier feedback was still needed.
- Noted longer-term coupling between sidebar code and private QC viewer helpers.
- Suggested optimizing repeated `qc.json` parsing later.
- Found that autoplay ignored the active subject/session filter and could move
  to hidden subjects.
- Suggested moving `_request_navigation_rerun()` into a neutral shared utility.
- Flagged suspicious `.env` removal from `.gitignore`.
- Suggested moving `_compact_session_label` into `ui/utils/cohort.py`.
- Questioned hard-coded `ses-01`.
- Flagged merge issues around `_display_iqm_panel`.
- Found a filtered-completion bug where the congratulations page could be
  skipped even when the full cohort became complete.
- Suggested a message when autoplay finishes the currently filtered subset but
  the full cohort is incomplete.
- Approved after asking for the congratulations bug and other comments to be
  addressed.

### #83: Fix QC export/save flows and resume loading edge cases

Link: https://github.com/nipoppy/qc-studio/pull/83  
Author: `nikhil153`  
State: merged

Review contribution:

- Flagged that “Save QC results to CSV” did not match the default `.tsv` path.
- Raised rater-ID filename capitalization/normalization concerns.
- Raised concern about saving output relative to script working directory
  instead of a stable `qc-studio/output` path.
- Tested checkpoint behavior and found checkpoint files were not appearing
  where the UI message suggested.
- Identified that changing the default path did not affect checkpoint save
  location.
- Reported checkpoint workflow confusion from a user perspective.
- Flagged subject ID zero stripping during upload/checkpoint.
- Noted a loading-record message was not very visible.
- Asked whether sidebar search was also being refactored in the PR.
- Reported notes not saving properly during autoplay after related changes.
- Review summary called out “QC records” wording even though values were
  cohort-page counts.

### #95: Enh 79 multifacet ratings

Link: https://github.com/nipoppy/qc-studio/pull/95  
Author: `nikhil153`  
State: merged October 8, 2026

Review contribution:

- Reviewed multifacet rating code deeply from both app behavior and code
  reading.
- In `ui/utils/cohort.py`, suggested handling missing facet values without
  converting `NaN` into the string `"nan"`.
- In `ui/views/landing_page.py`, suggested safer facet-mask handling for
  missing/blank facets.
- In `ui/managers/session_manager.py`, questioned how mixed values like
  `FAIL + UNCERTAIN` or all `UNCERTAIN` should be summarized.
- Flagged a local path in `ui/fsqc_test.sh`.
- Found a stale-widget guard bug in `ui/components/qc_viewer.py`: all-`None`
  multifacet ratings could be saved and hide real previous ratings.
- Found checkpoint lookup problems in `ui/components/qc_viewer.py`: globbing
  all checkpoint files could compare against another rater/task, and
  `qc_session_id` was accepted but unused.
- Found facet key sanitization could save `t1_snr` instead of original
  `T1 SNR`, breaking saved values/export/import matching.
- Found “Add notes” editable state could persist across later subjects.
- Flagged duplicated/identical code.
- Found task switching bug: after finishing one task and switching tasks, the
  old `current_page` could leave the app on the congratulations page instead
  of the first page of the new task.
- Summarized remaining issues: task page reset, case mismatch in save paths,
  inaccurate `All-Fail` labeling for mixed uncertainty, checkpoint lookup
  depending on records, and export-path handling.
- Follow-up review identified differing casing rules between filename helpers
  and requested a shared slug helper.
- Explained that `all_tasks` checkpoint filenames could be missed when lookup
  inferred a single task from records, while broad lookup could cross raters.
- Flagged underscore-delimited parsing of display labels: rater `test_user`
  and pipeline `freesurfer_v7` could produce an incorrect export name. Suggested
  passing explicit pipeline identity rather than parsing the label.
- Reported that absolute wildcard paths were rejected by `Path.glob`, despite
  relative patterns and absolute literal paths working.
- On October 6, retested finishing a task, switching tasks, and uploading a
  wrong-task checkpoint with `fsqc_test.sh`, then approved the PR.
- Recorded the test limitation: sample data only provided images for the first
  task, so image-level checks of other tasks were not possible. Suggested a
  visible missing-image warning rather than silently hiding the viewer.
- Nikhil reported fixing the remaining sanitized-naming and globbing issues
  before merging. Fix implementation belongs to him; my contribution was
  review and verification. The earlier Edit Notes behavior was not reproduced
  by the maintainer, and this report does not claim every suggestion was fixed.

Evidence:

- [Follow-up review](https://github.com/nipoppy/qc-studio/pull/95#pullrequestreview-5408580458).
- [Retesting and approval](https://github.com/nipoppy/qc-studio/pull/95#pullrequestreview-5436647028).
- [Naming/globbing resolution](https://github.com/nipoppy/qc-studio/pull/95#issuecomment-6066641684).

### #96: Add CITATION.cff

Link: https://github.com/nipoppy/qc-studio/pull/96  
Author: `nikhil153`  
State: merged October 8, 2026

Review contribution:

- Reviewed and approved citation metadata PR.

### #105: [DOC] Add documentation website

Link: https://github.com/nipoppy/qc-studio/pull/105  
Author: `michellewang`  
State: open

Review contribution, submitted October 10:

- Verified a warning-free Sphinx build using `-n -W --keep-going`, and checked
  that the Quickstart demo started with the documented configuration.
- Checked the CLI flags table and documented export columns against the code.
- Reported that the README's `streamlit run ui/main.py` command lacked required
  arguments and could not start the app; suggested the working demo entry point.
- Suggested a separate documentation workflow so a docs-build failure would not
  prevent code tests from running; raised the differing Python versions between
  Read the Docs and code checks, and suggested strict builds with path filters.
- Flagged removal of pre-commit file filters, which would extend whitespace/EOF
  hooks to staged sample-data TSV, SVG, and JSON files.
- Requested instructions for viewing the built HTML site.
- Identified missing README references and enabled but unused Sphinx extensions.
- Inline comments identified a stray backtick and inconsistent saved-filename
  examples between upload and saving instructions; noted CSV and TSV support.
- Explicitly limited the review to structural checks, not a cover-to-cover
  documentation audit. The PR remains open; these are review findings, not
  implementation changes attributed to me.

Evidence:

- [Submitted review](https://github.com/nipoppy/qc-studio/pull/105#pullrequestreview-5480252267).
- [Filename consistency comment](https://github.com/nipoppy/qc-studio/pull/105#discussion_r4238816575).

## Issue Triage / Issue Discussion Contributions

### #4: Address UI requirements

Link: https://github.com/nipoppy/qc-studio/issues/4

Contribution:

- Suggested closing after QSIPrep compatibility was handled in recent PRs.

### #33: Evidence Bundle Format Validation

Link: https://github.com/nipoppy/qc-studio/issues/33

Contribution:

- Documented evidence package format, dataset caveats, IQM semantic-file caveats,
  and experiment-management needs.
- Reported sandbox completion and commit reference.

### #40: Multimodal Image Representation for Evidence Bundle

Link: https://github.com/nipoppy/qc-studio/issues/40

Contribution:

- Documented LLaVA 0.5B, online LLaVA, Claude IQM-only, and LLaVA 4B findings.
- Compared JSON validity, QC reasoning quality, and metric-interpretation
  consistency.

### #50: test_utils.py fails

Link: https://github.com/nipoppy/qc-studio/issues/50

Contribution:

- Closed as addressed by PR #25.

### #56: Not rated yet state

Link: https://github.com/nipoppy/qc-studio/issues/56

Contribution:

- Closed as addressed by PR #25.

### #57: No overwriting of session record when re-rating

Link: https://github.com/nipoppy/qc-studio/issues/57

Contribution:

- Re-tested against the current app with a two-participant cohort.
- Verified repeated re-rating did not duplicate rows and exported one latest row
  per participant.
- Traced `SessionManager.add_qc_record` and dedup behavior.
- Closed as likely fixed by #25.

### #75: Add Playwright end-to-end browser tests

Link: https://github.com/nipoppy/qc-studio/issues/75

Contribution:

- Clarified that e2e tests should be opt-in/local or separately marked because
  Playwright browser setup and real Streamlit app launch make them slower and
  heavier than default tests.

### #89: Clarify Expected Navigation, Record, and Save Operations

Link: https://github.com/nipoppy/qc-studio/issues/89#issuecomment-5674629745

Contribution:

- Proposed keeping rating recording immediate on selection.
- Distinguished recording in session state from recovery snapshots and explicit
  export to disk.
- Suggested simplifying Next / Confirm and Next and avoiding a preselected
  rating.
- Automatic snapshots were a proposal; the maintainer left them for future work.

## Branches / Work Areas

Branches and related local/remote work areas visible in the repo:

- `add-ci`
- `cache-svg-data`
- `e2e`
- `evidence_bundle`
- `extract-task-run-from-montage`
- `fix-73-play-mode-rating-persistence`
- `fix-81-export-save-path`
- `fix-links-md`
- `grouped-iqm-distributions-12`
- `iqm-viewer-fixes`
- `iqm-viewer-fixes-multisession`
- `svg-montage-wording`
- `update-panel-name-to-montage`
- `enh-79-multifacet-ratings`

## Representative Commit Topics

Representative local commit topics authored by Niusha include:

- Playwright e2e scaffold, viewer specs/tests, Firefox run, rater-form tests.
- Claude/agent review and test-writing skills for repeatable project workflows.
- IQM viewer redesign and DWI support.
- IQM reference-data caching and reference-data URL handling.
- Scanner metadata sidecar lookup fixes.
- Plotly/requests/Pillow/pyarrow dependency additions.
- Violin plot and legend improvements.
- Autoplay rating persistence regression tests.
- Notes-only QC save fix.
- SVG montage caching and loader refactor.
- Documentation link fixes.
- CI/pre-commit/test gate work.
- Evidence bundle experiments and LLaVA/LLM QC exploration.

## GSoC Report Framing

A concise framing for the final report:

> My GSoC contribution focused on making QC-Studio more reliable for real QC
> workflows. I implemented fixes and infrastructure around rating persistence,
> IQM visualization, SVG/montage performance, documentation, CI, and e2e
> testing. In parallel, I created and triaged issues, reviewed PRs from other
> contributors, and caught several workflow bugs involving saving, checkpoints,
> filters, autoplay, completion logic, multifacet ratings, and export paths.

Stronger version:

> I contributed across both implementation and project quality: feature work in
> IQM visualization and montage handling, bug fixes for play-mode rating
> persistence, testing/CI infrastructure, a spec-driven Playwright e2e scaffold
> for Streamlit workflows, and detailed code review/triage that identified
> user-facing data-loss and workflow edge cases before release.

## Good Evidence to Mention

- Nikhil explicitly said the PR review comments were useful and hard to catch
  with tests.
- #95 shows mature code review: concrete failure modes, affected files,
  reproduction reasoning, suggested fixes/tests, and retesting before approval.
- #105 shows documentation review beyond prose: startup behavior, strict builds,
  CI isolation, hook scope, and consistency with CLI/export behavior.
- #83 shows user-workflow testing and UX review around save/checkpoint/export.
- #74 and #25 show early ability to catch completion/resume/filter bugs in
  other contributors' PRs.
- #88 shows independent technical growth: learning Playwright/Streamlit e2e
  patterns and building reusable test infrastructure.

## Near-Finish-Line Priorities

If there is still time before final submission, the highest-value next steps are:

1. Finish or narrow #88 so it is mergeable, even if the agent/skills part is
   split out from the e2e framework.
2. Add one or two high-value e2e tests around real workflow bugs:
   save/checkpoint, upload/resume, or filtered completion/autoplay.
3. Follow up on open documentation review #105 and export semantics issue #112.
   Multifacet ratings #95 is now merged after review and retesting; any remaining
   notes or missing-image feedback can be tracked separately if still relevant.
4. Close or update stale issues you created so the final report can say which
   items landed, which remain future work, and why.
