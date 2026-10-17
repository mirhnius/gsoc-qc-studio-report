# QC-Studio GSoC Report: Appendices

**Contributor:** Niusha ([mirhnius](https://github.com/mirhnius))  
**Contribution status checked:** October 12, 2026

These appendices contain the detailed contribution records supporting the
[GSoC Final Report](GSoC_FINAL_REPORT.md). Review work and maintenance are
credited separately from implementation by other contributors. Counts overlap;
created issues and issues closed under my account are different categories.

## Appendix A: All Authored Pull Requests

| PR | Contribution | Branch | Status |
| --- | --- | --- | --- |
| [#14](https://github.com/nipoppy/qc-studio/pull/14) | Early IQM/reference-distribution prototype; informed later IQM work. | `grouped-iqm-distributions-12` | Closed without merge |
| [#22](https://github.com/nipoppy/qc-studio/pull/22) | IQM viewer, source configuration, plots, DWI support, reference helpers, and tests. | `iqm-viewer-fixes` | Merged |
| [#34](https://github.com/nipoppy/qc-studio/pull/34) | Broader 2D montage terminology in the UI and supporting material. | `update-panel-name-to-montage` | Merged |
| [#39](https://github.com/nipoppy/qc-studio/pull/39) | Basic CI, formatter/linter/spelling configuration, and local pre-commit checks. | `add-ci` | Merged |
| [#61](https://github.com/nipoppy/qc-studio/pull/61) | Draft extraction of task/run metadata from montage paths. | `extract-task-run-from-montage` | Open draft |
| [#62](https://github.com/nipoppy/qc-studio/pull/62) | SVG montage loading cache. | `cache-svg-data` | Merged |
| [#66](https://github.com/nipoppy/qc-studio/pull/66) | Broken documentation-link repairs. | `fix-links-md` | Merged |
| [#76](https://github.com/nipoppy/qc-studio/pull/76) | Immediate rating persistence near autoplay advance, guards, and regression tests. | `fix-73-play-mode-rating-persistence` | Merged |
| [#88](https://github.com/nipoppy/qc-studio/pull/88) | Browser-test harness, behavior specifications, workflow tests, and agent guidance. | `e2e` | Open draft |

PRs #36, #37, and #38 were authored by the Copilot agent. My work on them is
recorded under review and maintenance below.

## Appendix B: All Issues I Created in QC-Studio

| Issue | My contribution | Status |
| --- | --- | --- |
| [#11](https://github.com/nipoppy/qc-studio/issues/11) | Reported QC notes not persisting when saving TSV. | Closed |
| [#23](https://github.com/nipoppy/qc-studio/issues/23) | Tracked documentation for the MRIQC reference-data script. | Closed |
| [#24](https://github.com/nipoppy/qc-studio/issues/24) | Proposed the evidence-bundle architecture and evidence sources. | Open |
| [#33](https://github.com/nipoppy/qc-studio/issues/33) | Defined the format-validation sandbox and documented results and caveats. | Closed |
| [#35](https://github.com/nipoppy/qc-studio/issues/35) | Proposed CI checks and automated testing; basic checks delivered in #39. | Closed |
| [#40](https://github.com/nipoppy/qc-studio/issues/40) | Explored multimodal image representations and documented model experiments. | Open |
| [#41](https://github.com/nipoppy/qc-studio/issues/41) | Proposed lightweight logging for UI-state and workflow debugging. | Open |
| [#68](https://github.com/nipoppy/qc-studio/issues/68) | Requested accessible, compact controls for long cohorts; reviewed #74. | Closed |
| [#69](https://github.com/nipoppy/qc-studio/issues/69) | Proposed rating the current subject without forcing navigation. | Open |
| [#70](https://github.com/nipoppy/qc-studio/issues/70) | Requested display-setting persistence when leaving and returning to review. | Open |
| [#71](https://github.com/nipoppy/qc-studio/issues/71) | Reported inconsistent visibility of pass/fail controls. | Open |
| [#72](https://github.com/nipoppy/qc-studio/issues/72) | Reported autoplay scrolling and loading time consuming review time. | Open |
| [#73](https://github.com/nipoppy/qc-studio/issues/73) | Reported lost autoplay ratings and implemented #76. | Closed |
| [#75](https://github.com/nipoppy/qc-studio/issues/75) | Proposed separately run browser tests; opened draft #88. | Open |
| [#77](https://github.com/nipoppy/qc-studio/issues/77) | Reported IQM rendering problems during automatic advance. | Open |
| [#78](https://github.com/nipoppy/qc-studio/issues/78) | Reported autoplay continuing after returning to the landing page. | Open |
| [#81](https://github.com/nipoppy/qc-studio/issues/81) | Reported loss of partial progress and reviewed the save/checkpoint response. | Closed |
| [#82](https://github.com/nipoppy/qc-studio/issues/82) | Tracked reference-distribution filtering and display correctness. | Open |
| [#84](https://github.com/nipoppy/qc-studio/issues/84) | Reported notes being discarded on forced navigation. | Closed |
| [#101](https://github.com/nipoppy/qc-studio/issues/101) | Reported the minimum Streamlit version mismatch for the new `st.iframe` call. Fixed by #95. | Closed |
| [#112](https://github.com/nipoppy/qc-studio/issues/112) | Reproduced export merge behavior retaining older rows despite an overwrite warning; requested consistent wording or behavior. | Open |

The additional upstream issue is
[niivue/niivue-vscode #314](https://github.com/niivue/niivue-vscode/issues/314),
described in [Section 7 of the main report](GSoC_FINAL_REPORT.md#7-upstream-niivue-issue).

## Appendix C: Review and Comment Activity on Others' PRs

| PR and author | Code or behavior reviewed | My contribution |
| --- | --- | --- |
| [#10](https://github.com/nipoppy/qc-studio/pull/10), `nikhil153` | Viewer layout and overlay controls. | Asked about height/configuration behavior, reported colorbar stacking, checked the update, and approved. |
| [#17](https://github.com/nipoppy/qc-studio/pull/17), `laurapiro17` | Documentation typo. | Explained that prior changes had addressed it and closed the superseded PR. |
| [#20](https://github.com/nipoppy/qc-studio/pull/20), `nikhil153` | Initial pagination work: viewer, pagination, panel layout, and session-state managers. | My first review on the project. Seven inline comments across six files on duplicated functions and helpers, session state initialized twice, a missing `dataset_dir` argument, `panels_data` type handling, empty ratings and notes passed through pagination, and a proposed `normalize_selected_panels()` helper to unify panel keys. Written April 6–8, 2026 and [submitted](https://github.com/nipoppy/qc-studio/pull/20#pullrequestreview-4064027658) October 12, 2026. |
| [#25](https://github.com/nipoppy/qc-studio/pull/25), `GhazalehManj` | Session manager, cohort helpers, landing/completion pages, and sidebar navigation. | [Submitted review](https://github.com/nipoppy/qc-studio/pull/25#pullrequestreview-4397400200) with six inline comments: [silent duplicate compaction](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3330745664), [baseline/session assumptions](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370144815), [display-length constants](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370388945), [completed-upload navigation](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370573536), [Continue QC widget reset](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370631538), and [Start Over retaining records](https://github.com/nipoppy/qc-studio/pull/25#discussion_r3370687500). Also commented on [smaller, behavior-focused PRs](https://github.com/nipoppy/qc-studio/pull/25#issuecomment-4587917864) and [the dependency on this merge for planned CI/test work](https://github.com/nipoppy/qc-studio/pull/25#issuecomment-4587074842). I merged it after the follow-up changes. |
| [#38](https://github.com/nipoppy/qc-studio/pull/38), Copilot agent | Montage fixture/documentation terminology. | Approved cleanup and later closed the PR. |
| [#53](https://github.com/nipoppy/qc-studio/pull/53), `SRSteinkamp` | Keyboard controls and narrow-screen layout. | Raised notes-field shortcut conflicts, Play/Pause coverage, and wrapped button labels. |
| [#58](https://github.com/nipoppy/qc-studio/pull/58), `manonheff` | Review-duration statistics and plots. | Requested tests, questioned duration correctness, and suggested average-per-subject and grouped-histogram presentation. |
| [#67](https://github.com/nipoppy/qc-studio/pull/67), `nikhil153` | Documentation cleanup. | Reviewed, approved, and merged. |
| [#74](https://github.com/nipoppy/qc-studio/pull/74), `GhazalehManj` | Sidebar filtering, navigation, compact labels, and autoplay completion. | Identified filter/completion problems, shared-helper placement, session assumptions, and configuration concerns; approved after the congratulations-page bug and other comments were addressed. |
| [#83](https://github.com/nipoppy/qc-studio/pull/83), `nikhil153` | Export, checkpoint, upload/resume, filenames, counts, and notes. | Reported path/case mismatches, subject-ID preservation, misleading labels, checkpoint confusion, and autoplay notes behavior. |
| [#95](https://github.com/nipoppy/qc-studio/pull/95), `nikhil153` | Cohort helpers, session manager, rating widgets, checkpoint lookup, task navigation, filenames, and wildcard paths. | Identified saved ratings masked by all-`None` widgets, facet-name changes breaking saved matching, checkpoint lookup crossing raters and tasks, and stale task navigation. [Follow-up review](https://github.com/nipoppy/qc-studio/pull/95#pullrequestreview-5408580458) on inconsistent filename casing, `all_tasks` checkpoint lookup, identifiers containing underscores, and absolute wildcard paths rejected by `Path.glob`. On October 6 I tested finishing one task, switching tasks, and uploading a wrong-task checkpoint, then [approved](https://github.com/nipoppy/qc-studio/pull/95#pullrequestreview-5436647028). Only the first task had image data in the samples, limiting image-level checks of the others, so I suggested a visible missing-image warning. Merged October 8. |
| [#96](https://github.com/nipoppy/qc-studio/pull/96), `nikhil153` | Citation metadata. | Reviewed and approved. Merged October 8. |
| [#105](https://github.com/nipoppy/qc-studio/pull/105), `michellewang` | Sphinx docs, README/Quickstart, CLI and export conventions, CI, and pre-commit scope. | [Submitted review](https://github.com/nipoppy/qc-studio/pull/105#pullrequestreview-5480252267) on October 10 recording a warning-free Sphinx build with `-n -W --keep-going`, demo startup, and checks of CLI flags and export columns against the code. Reported a README startup command missing required arguments, suggested independent documentation CI, and raised pre-commit scope reaching sample data; also flagged access to the built HTML, [saved-filename inconsistencies](https://github.com/nipoppy/qc-studio/pull/105#discussion_r4238816575), missing references, and unused Sphinx extensions. On October 11 asked for the PyBIDS mention to be removed from the session-list docs, since #111 removes PyBIDS session guessing. A structural review, not a cover-to-cover audit. Open. |
| [#111](https://github.com/nipoppy/qc-studio/pull/111), `nikhil153` | NIfTI and montage caching for memory use, and removal of PyBIDS-based session guessing. | [Submitted review](https://github.com/nipoppy/qc-studio/pull/111#pullrequestreview-5481379082) on October 11 with three design points: two caching styles in two modules; asymmetric invalidation, since the NIfTI cache key includes file modification time and size while the montage cache is keyed only on paths and configuration, so a regenerated figure can stay stale until evicted; and a shared cached dict that is read-only only by docstring, for which I asked for an immutable return value or a test guarding against mutation. A design review, not a test of memory usage. Open, marked WIP. |

These rows record my review and maintenance role. Implementations belong to the
PRs' authors.

## Appendix D: Merge and Closure Activity

### Pull Requests Merged Under My Account

- [#22](https://github.com/nipoppy/qc-studio/pull/22): my IQM viewer work.
- [#25](https://github.com/nipoppy/qc-studio/pull/25): pagination compatibility;
  review and merge contribution.
- [#39](https://github.com/nipoppy/qc-studio/pull/39): my basic CI work.
- [#62](https://github.com/nipoppy/qc-studio/pull/62): my SVG caching work.
- [#66](https://github.com/nipoppy/qc-studio/pull/66): my documentation-link fixes.
- [#67](https://github.com/nipoppy/qc-studio/pull/67): documentation cleanup;
  review and merge contribution.
- [#76](https://github.com/nipoppy/qc-studio/pull/76): my autoplay persistence fix.

### Pull Requests Closed Without Merge Under My Account

- [#14](https://github.com/nipoppy/qc-studio/pull/14): early IQM prototype.
- [#17](https://github.com/nipoppy/qc-studio/pull/17): already addressed
  documentation typo.
- [#36](https://github.com/nipoppy/qc-studio/pull/36),
  [#37](https://github.com/nipoppy/qc-studio/pull/37), and
  [#38](https://github.com/nipoppy/qc-studio/pull/38): Copilot-authored cleanup PRs.

### Issue Closures Recorded Under My Account

| Issue | Closure context |
| --- | --- |
| [#12](https://github.com/nipoppy/qc-studio/issues/12) | IQM distribution feature delivered through #22. |
| [#23](https://github.com/nipoppy/qc-studio/issues/23) | Reference-data script documentation completed with the IQM work. |
| [#33](https://github.com/nipoppy/qc-studio/issues/33) | Initial evidence-format sandbox completed. |
| [#35](https://github.com/nipoppy/qc-studio/issues/35) | Linked basic CI PR #39 merged. |
| [#43](https://github.com/nipoppy/qc-studio/issues/43) | SVG caching delivered through #62. |
| [#50](https://github.com/nipoppy/qc-studio/issues/50) | Test failure considered addressed by #25; closed with follow-up. |
| [#56](https://github.com/nipoppy/qc-studio/issues/56) | Not-rated state considered addressed by #25. |
| [#57](https://github.com/nipoppy/qc-studio/issues/57) | Re-tested re-rating, checked exported row uniqueness, and closed as likely fixed by #25. |
| [#63](https://github.com/nipoppy/qc-studio/issues/63) | Documentation cleanup delivered through #67. |
| [#64](https://github.com/nipoppy/qc-studio/issues/64) | Documentation links repaired through #66. |
| [#73](https://github.com/nipoppy/qc-studio/issues/73) | Autoplay rating persistence fixed through #76. |

## Appendix E: Additional Issue Discussion

| Issue | Contribution to discussion or triage |
| --- | --- |
| [#4](https://github.com/nipoppy/qc-studio/issues/4) | Suggested closure after the relevant QSIPrep compatibility work. |
| [#33](https://github.com/nipoppy/qc-studio/issues/33) | Explained package format, public-data caveats, semantic-context review needs, and experiment completion. |
| [#40](https://github.com/nipoppy/qc-studio/issues/40) | Reported cloud/local model observations and distinguished response-format validity from reasoning quality. |
| [#50](https://github.com/nipoppy/qc-studio/issues/50) | Followed up on resolution of the failing tests. |
| [#56](https://github.com/nipoppy/qc-studio/issues/56) | Followed up on the not-rated state fix. |
| [#57](https://github.com/nipoppy/qc-studio/issues/57) | Documented re-rating verification against a two-participant cohort and exported records. |
| [#75](https://github.com/nipoppy/qc-studio/issues/75) | Explained why browser tests need separate setup and should run separately from fast tests. |
| [#89](https://github.com/nipoppy/qc-studio/issues/89#issuecomment-5674629745) | Proposed immediate recording, separate export/recovery behavior, simpler navigation, and no assumed default rating. |

## Appendix F: Supporting Branches and Documents

- [IQM implementation branch](https://github.com/nipoppy/qc-studio/tree/iqm-viewer-fixes):
  feature development associated with merged PR #22.
- [E2E branch](https://github.com/nipoppy/qc-studio/tree/e2e): current browser-test
  work associated with draft PR #88.
- [Evidence-bundle branch](https://github.com/nipoppy/qc-studio/tree/evidence_bundle):
  sandbox code and recorded experiment artifacts; published tip
  `57c6ca8928db9949508b5c057cf929fa7a7915c1` at verification.
- [IQM explanation](IQM_VIEWER_BRANCH_EXPLANATION.md).
- [Playwright explanation](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md).
- [Evidence-bundle explanation](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md).

How this inventory was checked is described in
[About This Report](GSoC_FINAL_REPORT.md#about-this-report).
