# Playwright End-to-End Testing

**Primary PR:** [#88](https://github.com/nipoppy/qc-studio/pull/88), open draft  
**Primary branch:** [`e2e`](https://github.com/nipoppy/qc-studio/tree/e2e)  
**Branch snapshot:** October 5, 2026. Narrative revised October 10, 2026.

## Overview

I introduced a Playwright browser-test framework for QC-Studio's Streamlit
interface. The framework launches the real application, drives it through a
browser, and checks workflows involving forms, navigation, notes, and autoplay.
It complements the existing mocked unit tests by exercising the connection
between application logic and the controls a reviewer actually uses.

The work follows a specification-first approach. Each planned workflow starts
with a readable description of the expected behavior, followed by an automated
test where implemented. The contribution is currently an open draft, not complete
end-to-end coverage or finished production test infrastructure.

## Why Browser Tests Were Needed

Many of the bugs found during QC-Studio review involved a sequence of actions:
record a decision, advance, return to the subject, save progress, or let autoplay
move the session forward. The underlying helpers could behave correctly in
isolation while the real widgets, session state, and Streamlit reruns interacted
incorrectly.

The existing `ui/tests/` suite mocks Streamlit and remains useful for fast
logic checks. Browser tests add coverage of real form submissions, widget state,
navigation, reruns, and elapsed-time behavior. Their role is to complement those
tests, rather than replace them or require every small logic change to launch
a browser.

[Issue #75](https://github.com/nipoppy/qc-studio/issues/75) proposed this separate
testing layer. An important decision was to mark and run the suite independently
of the fast tests, since Playwright needs browser binaries and a real Streamlit
server. [PR #88](https://github.com/nipoppy/qc-studio/pull/88) contains the
`e2e/` scaffold, specifications, initial tests, and project-specific guidance
for contributors and coding agents.

## Test Harness and Shared Helpers

The pytest fixtures in `e2e/conftest.py` allocate a port, launch QC-Studio with
`streamlit run`, wait for its health endpoint, and provide a loaded browser
page. Test outputs go to temporary directories rather than the repository's
normal `output/` folder, and the fixture handles server cleanup. This creates
an isolated environment for each test session.

Shared interactions and widget selectors are centralized in
`e2e/shared/app_utils.py`. Rerun synchronization lives in
`e2e/shared/waits.py`, while `e2e/pytest.ini` defines the suite's configuration
and markers. The setup and writing conventions are described in `e2e/README.md`,
with behavior descriptions in `e2e/specs/` and test implementations in
`e2e/tests/`.

Centralizing widget lookup keeps test bodies focused on the workflow rather than
repeating raw locator logic. Tests use application constants from
`ui/constants.py` for user-facing text where applicable. The helper layer also
handles differences between ordinary buttons and Streamlit form-submit buttons,
along with alert-text handling where leading emojis may be stripped.

## Streamlit Reruns and Synchronization

Streamlit reruns the application script after interactions. A browser assertion
immediately after a click can therefore inspect the previous render instead of
the resulting state. Negative assertions are particularly sensitive: something
may appear absent only because the new render has not arrived yet.

The framework adds explicit waits around these transitions, including waiting
for Streamlit's script state to become `notRunning`. This is an important part
of the harness, but synchronization remains an area raised in review and still
needs refinement. A wait helper's existence is not a claim that every possible
rerun timing issue has been resolved.

## Specification-First Workflow

Each specification records the screen, category, priority, reason for browser
coverage, and expected behavior using Given/When/Then steps. It also includes
relevant copy/constants and implementation notes. This makes the intended
behavior available for review before the Playwright implementation exists and
provides a consistent starting point for future contributors.

The branch contains 11 specifications. The first group covers entry into the
viewer: `001-rater-form-validation.md` checks empty rater IDs,
`002-rater-form-panel-validation.md` checks missing panel selections, and
`006-valid-rater-form-opens-viewer.md` describes successful entry. Upload
behavior is covered by `005-upload-resumes-session.md` and
`007-upload-rejects-unknown-participants.md`.

Recorded decisions and navigation are described in
`003-rating-survives-navigation.md` and
`008-next-preserves-recorded-rating.md`. Saving and recovery are covered by
`004-final-export-overwrite-guard.md` and
`009-checkpoint-during-qc.md`. Finally, `010-notes-are-saved.md` describes
notes surviving navigation, while `011-notes-pause-autoplay.md` describes
pausing automatic advance to write notes.

These are 11 planned behavior descriptions, not 11 completed automated tests.
Their value is to make expected behavior explicit and keep future coverage tied
to actual review-session risks.

## Implemented Tests and Browser Coverage

The initial implementation contains six test scenarios across four files.
`e2e/tests/test_smoke.py` checks that the application boots and that the rater
form is reachable. `e2e/tests/test_rater_form.py` checks rejection of an empty
rater ID and submission without selected panels. These establish basic startup
and validation coverage.

`e2e/tests/test_notes_persistence.py` checks that a note survives navigation
away from and back to a subject. `e2e/tests/test_viewer_autoplay.py` checks that
opening Add notes pauses autoplay and prevents the subject from advancing.
These tests exercise workflows connected to the persistence and timing problems
reported during manual review.

The suite is configured for Chromium and Firefox. Using both engines can help
identify browser-sensitive assumptions in selectors and Streamlit behavior.
The PR description records successful initial Chromium smoke tests, but this
document does not claim that the complete current suite has passed on both
engines during report preparation or this editorial revision.

Some tests disable Niivue to focus on forms, notes, and autoplay without waiting
for volume rendering. That is a deliberate test boundary: those scenarios check
the surrounding workflow, not whether a scan has loaded or rendered correctly.

## Contributor and Coding-Agent Guidance

The branch includes guidance intended to keep future tests and reviews consistent
with the repository. The files
`.claude/skills/qcs-understanding-e2e/SKILL.md` and
`.claude/skills/qcs-playwright-testing/SKILL.md` describe the suite architecture
and Playwright conventions. Related review and test-writing guidance lives in
`.claude/skills/qcs-reviewing-code/SKILL.md` and
`.claude/skills/qcs-generating-tests/SKILL.md`.

The agent definitions are
`.claude/agents/qcs-code-reviewer.md` and
`.claude/agents/qcs-test-generator-runner.md`. Repo-level `AGENTS.md` points
other coding agents toward the same project guidance and records contribution
conventions. These files support the test-writing workflow; the main engineering
contribution remains the browser harness, shared helpers, and tests.

The maintainer still needs to agree on the scope and placement of these guidance
files. They could remain with the framework or be separated into another PR
without changing the purpose of the browser tests.

## Review Feedback and Current Status

Review feedback identified rerun-wait synchronization concerns, export tracking
that may miss overwritten deterministic filenames, CSV wording where the
application writes TSV, and smoke assertions that should use shared helper paths.
These are outstanding refinement items rather than changes claimed as completed.

The PR remains a draft. Before merging, the framework needs those review points
addressed and its current tests validated. Additional high-value coverage includes
upload/resume, the final-export overwrite guard, checkpoint creation, and ratings
surviving navigation. Existing specifications provide a starting point, but
each workflow still requires implementation and verification.

## Contribution and Next Steps

My contribution was building a reusable browser-testing layer around the real
Streamlit application and connecting it to readable workflow specifications.
This provides a way to turn selected manual review findings into repeatable
checks across widgets, state, navigation, reruns, and saving behavior.

The immediate next steps are to agree on the guidance-file scope, strengthen
rerun waits and overwrite tracking, correct documentation inconsistencies, and
validate the implemented tests. The suite should remain separately marked so
contributors can run quick logic checks independently from browser workflows.

This work is summarized in the [GSoC Final Report](GSoC_FINAL_REPORT.md), with
linked contribution records in the [report appendices](GSoC_REPORT_APPENDIX.md).
