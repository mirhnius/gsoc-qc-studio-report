# Playwright End-to-End Testing

**Primary PR:** [#88](https://github.com/nipoppy/qc-studio/pull/88), open for review  
**Branch:** [`e2e`](https://github.com/nipoppy/qc-studio/tree/e2e)  
**Branch snapshot:** October 5, 2026

## Overview

I introduced a Playwright framework that launches the real QC-Studio app, drives
it through a browser, and checks workflows involving forms, navigation, notes,
and autoplay. It complements the mocked unit tests by exercising the connection
between application logic and the controls a reviewer uses. Each planned
workflow starts as a readable specification, followed by an automated test where
implemented. The suite now covers 13 specifications with 19 tests across 10
files, and runs in CI on request.

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
| `shared/flows.py` | A rater's journey rather than a single widget: everything a test needs in order to *reach* the scenario it is about, described once so each test opens with one readable line. |
| `tests/` | Scenarios, which know about none of the above. |

`flows.py` also holds the decision to leave Niivue off by default
(`SLOW_PANELS`). Niivue renders a real 3D volume client-side, which takes tens
of seconds, and `wait_for_app_run` blocks on its loading skeleton on every
rerun. A test that is not about the 3D viewer should not pay that cost; the one
spec that *is* about panel rendering overrides it.

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

I was curious about spec-driven development, and this is the concept rather
than the full method: a Markdown spec is written before any test code.
`e2e/specs/TEMPLATE.md` asks for the screen, category and priority, and then
the question that does the most work — **what wiring would break and go
unnoticed if this test did not exist?** It says outright that if a unit test in
`ui/tests/` covers it just as well, write that instead, and that pure logic
belongs there rather than here.

The Given/When/Then section insists that "Then" is something visible on screen,
not state a test cannot observe, and asks which `ui/constants.py` keys the test
should assert on so strings are imported rather than hardcoded. It also asks
for an optional second consequence — usually "and the app did NOT move on" —
noting that negative assertions are the most valuable and the easiest to get
wrong.

**The intended contributor workflow:** fill in the spec with the scenario and
what you think matters, hand it to your AI assistant of choice to review and
improve the spec, let it implement the test, then run it yourself and judge
whether it does what you wanted. Writing the spec first makes the expected
behaviour reviewable while disagreements are still cheap, and the human stays
the one who decides the test is right.

The thirteen specifications are in `e2e/specs/`, covering entry and form
validation, upload and resume, ratings surviving navigation, the final-export
overwrite guard, checkpoints, notes, autoplay, multi-task rating, and the
subject list.

## Implemented Tests

Nineteen tests across ten files:

| File | Covers |
| --- | --- |
| `test_smoke.py` | The app boots; the rater form is reachable. |
| `test_rater_form.py` | An empty rater ID and a submission with no panels selected are both rejected. |
| `test_viewer_rating.py` | A valid form opens the viewer; a rating survives navigating away and back; and two tests describing how the configured default rating behaves — including that a default of `None` leaves subjects unrated. |
| `test_subject_list.py` | Rating a subject marks it complete in the list; the search box filters the list. |
| `test_notes_persistence.py` | A note survives navigating away and back. |
| `test_viewer_autoplay.py` | Opening Add notes pauses autoplay so the subject does not advance. |
| `test_upload.py` | Uploading a previous QC file loads its records; unknown participants are rejected. |
| `test_checkpoint.py` | A checkpoint writes a snapshot of the ratings so far. |
| `test_final_export.py` | The export writes the results file; exporting over an existing file needs confirmation. |
| `test_multi_task.py` | Several tasks render on one page; and the `xfail` below. |

Three specifications were dropped during implementation rather than
implemented, as redundant once the tests around them existed.

Some tests leave Niivue off so they exercise forms, notes, and ratings without
waiting on volume rendering. That is a deliberate boundary: those scenarios
check the surrounding workflow, not whether a scan loaded.

## What the Suite Found

Writing the tests surfaced behaviour that is a product question rather than a
test problem, recorded in
[`e2e/FINDINGS.md`](https://github.com/nipoppy/qc-studio/blob/e2e/e2e/FINDINGS.md).
Nothing in that list is fixed; each item needs a decision from whoever owns the
behaviour. In short:

- **Next does not require every task to be rated**, though the tooltip says it
  does — the untouched tasks never reach the export.
- **The default rating is applied inconsistently**, reaching subjects arrived
  at by navigation but not the first one a rater lands on.
- **The viewer's overwrite guard is never called**, and `_save_qc_record` takes
  an `allow_overwrite` parameter its body never reads.
- **Two constants no longer render anywhere**, so a test written against either
  looks correct and can never pass.
- **One known flake**, in the only test whose timing depends on wall-clock
  rather than on the app.

The first two are pinned by `xfail(strict=True)` tests, so CI stays green and
the test fails loudly the moment the behaviour changes — at which point the
marker comes off.

## Continuous Integration

The suite runs through its own
[workflow](https://github.com/nipoppy/qc-studio/blob/e2e/.github/workflows/e2e-tests.yml),
separate from the basic checks, and deliberately not on every push: starting a
real Streamlit server and driving two browsers costs minutes rather than
seconds, and most changes cannot break it.

It is asked for in one of two ways — pressing **Run workflow** on the Actions
tab, with a choice of Chromium, Firefox or both, or adding a **`run-e2e`
label** to a pull request. The label is the useful one during review: the run
reports back on the PR itself, and the `synchronize` trigger keeps it running
on later pushes to an already-labelled PR instead of needing the label
reapplied.

Playwright browser binaries are cached, keyed on the pinned requirements file
so a version bump fetches fresh ones rather than reusing incompatible
binaries. `--with-deps` runs every time regardless, because the cache restores
the browsers but not the system packages they need. A failing run uploads
screenshots and video as artifacts, which is the point of running in CI at all:
seeing what broke on a machine you cannot open a browser on.

## An Environment Lesson

`requirements.txt` asks only for `streamlit>=1.56`. A local environment on 1.58
and a CI runner on 1.65 produced different markup for radio widgets, and
sixteen tests that passed locally failed in CI.

The fix was on the suite side: widgets are now located through their ARIA role
and accessible name rather than the component library's internal attributes,
which is stable across both versions. That is precisely what the shared
selector layer exists to absorb — one file changed, no test bodies touched.
The unpinned dependency is still there and will produce the same class of
surprise again.

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

## Current Status and Next Steps

The PR is open for review. What remains is less about the suite than about
decisions around it: agreeing the scope and placement of the committed agent
skills, and acting on the product questions above — each needs someone who owns
the behaviour to say what it should be, after which the two `xfail` markers come
off or the tests change.

Earlier review points have been addressed: the rerun waits, export tracking for
overwritten deterministic filenames, "CSV" wording where the app writes TSV, and
smoke assertions that bypassed the shared helpers. The suite stays separately
marked from the fast tests, and `--browser` now replaces the configured defaults
rather than adding to them.

See the [GSoC Final Report](GSoC_FINAL_REPORT.md) and
[appendices](GSoC_REPORT_APPENDIX.md).
