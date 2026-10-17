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

## Structure: Four Layers

Playwright was new to me, so rather than invent a structure I studied how
Streamlit's own `e2e_playwright` suite is organised — in particular how it keeps
tests from breaking when Streamlit's internals change, by wrapping the fragile
parts so there is one place to fix instead of many. The suite ended up with four
layers, each hiding its mechanics from the one above:

| Layer | Responsibility |
| --- | --- |
| `app_server` fixture | Launches a real QC-Studio process on an OS-assigned free port, polls `/_stcore/health` until it serves, and includes the server log in the failure if startup dies. Module-scoped, with a fresh browser page per test so no session state leaks between them. |
| `app` fixture | A browser page already loaded and settled on that server. |
| `shared/` | Every selector and interaction: `waits.py` for rerun synchronization, `app_utils.py` for finding and driving widgets. |
| `tests/` | Scenarios, which know about none of the above. |

`build_launch_command()` is split out of the fixture so the `streamlit run`
command can be inspected without starting anything. `QCAppConfig` is a frozen
dataclass holding one launch configuration, overridable per test module to
exercise a different pipeline.

Two fixture details that matter for correctness:

- **Two-subject configuration.** The default has a single participant, which
  cannot distinguish a real bug from a one-subject edge case: autoplay stopping
  at the end of a one-subject cohort looks identical to a pause actually
  working. Specs needing genuine navigation rebind to a two-subject config.
- **Owned output directory.** Tests write to a temporary directory instead of
  the repository's `output/`, so asserting on an exported file is a legitimate
  end-to-end check. Because the server is module-scoped, an `output_files`
  fixture snapshots the directory before each test and returns only what that
  test created.

## Synchronization

This is the part that silently breaks a Streamlit browser suite. Streamlit
reruns the whole script on every interaction, so for a few hundred milliseconds
the browser still shows the previous render. Playwright's built-in retrying
hides this most of the time, and the case where it does not is the dangerous
one:

```python
expect(error_message).not_to_be_visible()
```

That passes instantly against the stale page, before the rerun that would have
produced the error. A green test over a shipped bug.

Rather than guessing with timeouts, the helpers ask the app. Streamlit publishes
its status on the root element:

```html
<div data-testid="stApp"
     data-test-connection-state="CONNECTED"
     data-test-script-state="notRunning">
```

`data-test-script-state` cycles `initial → running → notRunning` on every
rerun, so waiting for `notRunning` means the script has finished. The helper
also waits for the websocket to be `CONNECTED` and for zero `stSkeleton`
elements, so every element has actually rendered. Every interaction helper in
`app_utils.py` calls it, so a test cannot forget to. This approach is adapted
from Streamlit's own `e2e_playwright/conftest.py`.

## Selectors and Streamlit's Quirks

Tests never call `page.locator()` directly. Every selector lives in
`shared/app_utils.py`, so when Streamlit changes its internals there is exactly
one file to fix. The strategy is ordered by robustness:

1. `get_by_key(page, "rater_id")` — Streamlit stamps a stable `st-key-<key>`
   class on any widget given an explicit `key=`. Independent of label text, so
   it survives copy changes.
2. A widget test ID narrowed by its visible label, which is what Streamlit's own
   suite uses.
3. Raw CSS selectors — avoided.

The test IDs were read from Streamlit's own suite. The two that had to be
inferred are marked `UNVERIFIED` in the source with a note on what to re-check
against the installed version if they stop matching.

The same layer absorbs Streamlit's quirks once, instead of in every test:

- `st.text_area` commits `on_change` on blur or Ctrl+Enter, not per keystroke,
  so `fill()` alone changes the DOM without notifying Streamlit. The helper
  fills, then presses Tab to blur.
- Checkboxes render their real `<input>` off-screen behind a styled `<label>`,
  which fails Playwright's actionability checks even with `force=True`. The
  helper clicks the visible label and lets the browser forward the event;
  dispatching to the hidden input changes `checked` but never reaches session
  state.
- A button with a `help=` tooltip renders a second, hidden `<button>` beside
  the real one, so the locator filters to `:visible` to stay a single element
  under strict mode.
- `st.error` and friends extract a single leading emoji into a separate icon,
  stripping it from the text node, so a constant authored with that emoji never
  appears verbatim in the DOM and has to be stripped before asserting.
- Sliders are driven with the keyboard from a known end of the range, so the
  helper works from any starting value.

Every test runs in both Chromium and Firefox by design. A suite leaning this
hard on Streamlit's DOM cannot assume a selector that works in one engine works
in the other, and the browser appears in the test ID in the report
(`test_app_boots[firefox]`). Markers are `smoke`, `flow`, and `edge`.

## Specifications

I was curious about spec-driven development, and this is the concept rather than
the full method: a Markdown spec is written before any test code.
`e2e/specs/TEMPLATE.md` asks for the screen, category, and priority, and then
the question that does the most work — **what wiring would break and go
unnoticed if this test did not exist?** The template says outright that if a
unit test in `ui/tests/` covers it just as well, write that instead, because it
is faster and more precise, and that pure logic (string cleaning, config
parsing, dataframe filtering) belongs there rather than here.

The Given/When/Then section insists that "Then" is something visible on screen,
and contrasts the two directly:

```
BAD   the rating is saved to session state      <- invisible, untestable
GOOD  the PASS button renders as selected
GOOD  the progress counter reads "2 of 3"
```

The export is the one exception, since the tests own the output directory, so
"a CSV exists with one row per rated participant" is fully observable. The
template also asks for an optional second consequence — usually "and the app did
NOT move on" — noting that negative assertions are the most valuable and the
easiest to get wrong. Finally it asks which `ui/constants.py` keys the test
should assert on, so whoever writes it imports the strings instead of
hardcoding them.

**The intended contributor workflow:** fill in the spec with the scenario and
what you think matters, hand it to your AI assistant of choice to review and
improve the spec, let it implement the test, then run it yourself and judge
whether it does what you wanted. Writing the spec first makes the expected
behavior reviewable while disagreements are still cheap, and the human stays the
one who decides the test is right.

The branch has 11 specifications:

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

## Shared Conventions for Contributors

The spec-first workflow only helps if contributors follow the same conventions,
so the branch commits them. `.claude/skills/` holds `qcs-understanding-e2e` and
`qcs-playwright-testing` for this suite's architecture and Playwright
conventions, plus `qcs-reviewing-code` and `qcs-generating-tests` as general
review and test-writing conventions for the repository; `.claude/agents/` holds
the matching `qcs-code-reviewer` and `qcs-test-generator-runner`. A repo-level
`AGENTS.md` points other coding agents at the same guidance.

These are committed rather than kept local on purpose: anyone using an AI
assistant on this repository then gets the same behavior, instead of each
contributor's assistant inventing its own conventions. By contrast
`settings.local.json` is deliberately not committed, since a personal
permission allowlist is specific to one machine rather than a project
convention.

The maintainer still needs to agree on the scope and placement of these files.
They could stay with the framework or move to a separate PR without changing
the purpose of the browser tests.

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
