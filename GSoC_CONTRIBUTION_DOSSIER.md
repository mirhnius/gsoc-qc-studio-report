# QC-Studio GSoC: Review and Triage Detail

As of: October 12, 2026  
Repository: `nipoppy/qc-studio`  
GitHub account: `mirhnius` / Niusha

This file gives the comment-level detail behind section 6 of the
[GSoC Final Report](GSoC_FINAL_REPORT.md). Authored PRs, issues created, merges,
and closures are listed in the [Report Appendices](GSoC_REPORT_APPENDIX.md).
Fixes made in other contributors' PRs are credited to their authors.

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

My first substantial review on the project. I went through the pagination code
in detail and wrote many comments on it, but the review is still pending on
GitHub, so those comments were never published on the PR; at the time I was not
yet comfortable with GitHub's review workflow.

The maintainer's response records that the input reached him:
[Nikhil thanked me by name](https://github.com/nipoppy/qc-studio/pull/20#issuecomment-4413781991),
alongside `ThomasHMAC` and `GhazalehManj`, for the comments on this PR and said
he had addressed them all before asking for approval to merge. The pagination
implementation is credited to him.

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
- On October 11, asked for the PyBIDS mention to be removed from the
  session-list docs, because #111 removes PyBIDS-based session guessing and
  `--session_list` becomes required unless the participant TSV has a
  `session_id` column.
- Explicitly limited the review to structural checks, not a cover-to-cover
  documentation audit. The PR remains open; these are review findings, not
  implementation changes attributed to me.

Evidence:

- [Submitted review](https://github.com/nipoppy/qc-studio/pull/105#pullrequestreview-5480252267).
- [Filename consistency comment](https://github.com/nipoppy/qc-studio/pull/105#discussion_r4238816575).

### #111: [WIP] Memory optimization

Link: https://github.com/nipoppy/qc-studio/pull/111  
Author: `nikhil153`  
State: open

Review contribution, submitted October 11:

- Pointed out two caching styles in `qc_viewer.py` and
  `niivue_viewer_manager.py` and suggested a small shared module for limits, key
  rules, and documentation.
- Noted that the NIfTI cache key includes `mtime_ns` and size, while
  `_load_montage_data_cached` is keyed only on paths and configuration, so a
  regenerated montage could stay stale until evicted.
- Noted that `cache_resource` returns the same dict to every session and that
  read-only use was enforced only by a docstring; suggested an immutable return
  type or a test that fails on mutation.
- The author's later commit, "added shared cache module and decoupled
  load_montage_data to address PR comments", responds to this review. The
  implementation is credited to the author.
- [Submitted review](https://github.com/nipoppy/qc-studio/pull/111#pullrequestreview-5481379082).

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
