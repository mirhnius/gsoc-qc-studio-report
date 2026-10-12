# Playwright End-to-End Testing

**Primary PR:** [#88](https://github.com/nipoppy/qc-studio/pull/88), open draft  
**Branch:** [`e2e`](https://github.com/nipoppy/qc-studio/tree/e2e)  
**Branch snapshot:** October 5, 2026

## Overview

I introduced a Playwright framework that launches the real QC-Studio app, drives
it through a browser, and checks workflows involving forms, navigation, notes,
and autoplay. It complements the mocked unit tests by exercising the connection
between application logic and the controls a reviewer uses. Each planned
workflow starts as a readable specification, followed by an automated test where
implemented. The PR is an open draft, not complete coverage.

## Why Browser Tests

Many bugs found in review involved a sequence: record a decision, advance,
return, save, or let autoplay move on. Helpers can behave correctly in isolation
while real widgets, session state, and Streamlit reruns interact incorrectly.
`ui/tests/` mocks Streamlit and stays useful for fast logic checks; browser
tests add real form submissions, widget state, reruns, and elapsed-time
behavior without replacing them.

[Issue #75](https://github.com/nipoppy/qc-studio/issues/75) proposed this as a
separate layer. It is marked and run independently of the fast tests because it
needs browser binaries and a real Streamlit server.

## Harness and Helpers

- `e2e/conftest.py`: pytest fixtures allocate a port, launch the app with
  `streamlit run`, wait for its health endpoint, and provide a loaded page.
  Outputs go to temporary directories, and the server is cleaned up.
- `e2e/shared/app_utils.py`: shared widget selectors and interactions, including
  differences between ordinary and form-submit buttons and alert text where
  leading emojis are stripped. Tests use constants from `ui/constants.py`.
- `e2e/shared/waits.py`: rerun synchronization. Streamlit reruns the script
  after interactions, so an assertion right after a click can inspect the
  previous render, and negative assertions are especially prone to this. The
  helpers wait for the script state to become `notRunning`. Reviewers have
  raised synchronization concerns, so this still needs refinement.
- `e2e/pytest.ini`, `e2e/README.md`, `e2e/specs/`, and `e2e/tests/` hold the
  configuration, setup and conventions, specifications, and tests.

## Specifications

Each specification records the screen, category, priority, reason for browser
coverage, Given/When/Then steps, relevant copy and constants, and implementation
notes. The branch has 11:

- Entry: `001-rater-form-validation`, `002-rater-form-panel-validation`,
  `006-valid-rater-form-opens-viewer`.
- Upload: `005-upload-resumes-session`, `007-upload-rejects-unknown-participants`.
- Decisions and navigation: `003-rating-survives-navigation`,
  `008-next-preserves-recorded-rating`.
- Saving and recovery: `004-final-export-overwrite-guard`,
  `009-checkpoint-during-qc`.
- Notes and autoplay: `010-notes-are-saved`, `011-notes-pause-autoplay`.

These are 11 planned behaviors, not 11 automated tests.

## Implemented Tests

Six scenarios in four files:

- `test_smoke.py`: the app boots and the rater form is reachable.
- `test_rater_form.py`: an empty rater ID and a submission without selected
  panels are rejected.
- `test_notes_persistence.py`: a note survives navigating away and back.
- `test_viewer_autoplay.py`: opening Add notes pauses autoplay so the subject
  does not advance.

Chromium and Firefox are configured. The PR description records successful
initial Chromium smoke tests; I have not run the full suite on both engines for
this report. Some tests disable Niivue to focus on forms, notes, and autoplay,
so they do not check that a scan loads or renders.

## Contributor and Agent Guidance

The branch includes `.claude/skills/` files describing the suite architecture,
Playwright conventions, code review, and test generation
(`qcs-understanding-e2e`, `qcs-playwright-testing`, `qcs-reviewing-code`,
`qcs-generating-tests`), two agent definitions (`qcs-code-reviewer`,
`qcs-test-generator-runner`), and a repo-level `AGENTS.md`. The maintainer still
needs to agree on their scope; they could stay with the framework or move to a
separate PR.

## Review Feedback and Next Steps

Open review points: rerun-wait synchronization, export tracking that may miss
overwritten deterministic filenames, "CSV" wording where the app writes TSV, and
smoke assertions that should use shared helper paths.

Before merging: address those points, agree the guidance-file scope, correct
documentation inconsistencies, and validate the implemented tests. Higher-value
coverage to add next: upload/resume, the final-export overwrite guard,
checkpoint creation, and ratings surviving navigation. The suite should remain
separately marked from the fast tests.

See the [GSoC Final Report](GSoC_FINAL_REPORT.md) and
[appendices](GSoC_REPORT_APPENDIX.md).
