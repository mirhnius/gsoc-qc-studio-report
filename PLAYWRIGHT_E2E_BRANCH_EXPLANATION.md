# Playwright End-to-End Testing

**Primary PR:** [#88](https://github.com/nipoppy/qc-studio/pull/88), open for review  
**Branch:** [`e2e`](https://github.com/nipoppy/qc-studio/tree/e2e)

## Why Browser Tests

Many bugs found in review involved a sequence: record a decision, advance,
return, save, or let autoplay move on. Helpers behave correctly in isolation
while real widgets, session state, and Streamlit reruns interact incorrectly.
`ui/tests/` mocks Streamlit and stays useful for fast logic checks; this suite
launches the real app in a browser and tests the wiring between them.

[Issue #75](https://github.com/nipoppy/qc-studio/issues/75) proposed it as a
separate layer, marked and run independently of the fast tests because it needs
browser binaries and a real server. It now covers 13 specifications with 19
tests across 10 files, and runs in CI on request.

## Structure

Playwright was new to me, so rather than invent a structure I studied how
Streamlit's own `e2e_playwright` suite is organised — in particular how it keeps
tests from breaking when Streamlit's internals change, by wrapping the fragile
parts so there is one place to fix instead of many. The result is four layers,
each hiding its mechanics from the one above:

| Layer | Responsibility |
| --- | --- |
| `app_server` fixture | Launches a real QC-Studio process on a free port, polls `/_stcore/health` until it serves, and includes the server log in the failure if startup dies. Module-scoped, with a fresh page per test so no session state leaks. |
| `app` fixture | A browser page already loaded and settled on that server. |
| `shared/` | Every selector and wait: `app_utils.py` for driving widgets, `waits.py` for rerun synchronization. |
| `shared/flows.py` | A rater's journey rather than one widget — everything a test needs to *reach* its scenario, described once so each test opens with a single readable line. |
| `tests/` | Scenarios, which know about none of the above. |

Three decisions in the fixtures matter for correctness:

- **Niivue is off by default** (`SLOW_PANELS` in `flows.py`). It renders a real
  3D volume client-side, taking tens of seconds that `wait_for_app_run` blocks
  on every rerun. Tests not about the 3D viewer should not pay that; the one
  spec that is overrides it.
- **A two-subject configuration** exists because the default single participant
  cannot distinguish a real bug from an edge case: autoplay stopping at the end
  of a one-subject cohort looks identical to a pause working.
- **Tests own their output directory**, so asserting on an exported file is a
  legitimate end-to-end check. An `output_files` fixture snapshots it beforehand
  and returns only what that test wrote.

## Synchronization

This is what silently breaks a Streamlit browser suite. Streamlit reruns the
whole script on every interaction, so for a few hundred milliseconds the browser
still shows the previous render. Playwright's retrying hides this most of the
time, and the case where it does not is the dangerous one:

```python
expect(error_message).not_to_be_visible()
```

That passes instantly against the stale page, before the rerun that would have
produced the error — a green test over a shipped bug.

Rather than guessing with timeouts, the helpers ask the app. Streamlit publishes
`data-test-script-state` on its root element, cycling `initial → running →
notRunning` on every rerun, so waiting for `notRunning` means the script has
finished. The helper also waits for the websocket to be `CONNECTED` and for zero
loading skeletons. Every interaction helper calls it, so a test cannot forget
to. The approach is adapted from Streamlit's own `e2e_playwright/conftest.py`.

## Selectors and Streamlit's Quirks

Tests never call `page.locator()` directly. Every selector lives in
`shared/app_utils.py`, so when Streamlit changes its internals there is one file
to fix. The strategy, ordered by robustness: a widget's explicit `key=`, which
Streamlit stamps as a stable `st-key-<key>` class and which survives copy
changes; then a test ID narrowed by visible label, as Streamlit's own suite
does; raw CSS last.

The same layer absorbs Streamlit's quirks once rather than in every test:

- `st.text_area` commits on blur, not per keystroke, so the helper fills then
  presses Tab — `fill()` alone changes the DOM without notifying Streamlit.
- Checkboxes hide their real `<input>` off-screen, so the helper clicks the
  visible `<label>`; dispatching to the input changes `checked` without
  reaching session state.
- A button with a `help=` tooltip renders a second hidden `<button>`, so the
  locator filters to `:visible`.
- `st.error` extracts a leading emoji into a separate icon, so a constant
  authored with one never appears verbatim in the DOM.
- Sliders are driven from a known end of the range, so the helper works from any
  starting value.

Every test runs in both Chromium and Firefox by design: a suite leaning this
hard on Streamlit's DOM cannot assume a selector working in one engine works in
the other.

## Specifications

I was curious about spec-driven development, and this is the concept rather than
the full method: a Markdown spec is written before any test code.
`e2e/specs/TEMPLATE.md` asks for the screen, category and priority, and then the
question that does the most work — **what wiring would break and go unnoticed if
this test did not exist?** It says outright that if a unit test in `ui/tests/`
covers it just as well, write that instead. The Given/When/Then section insists
"Then" is something visible on screen, and asks which `ui/constants.py` keys the
test should assert on so strings are imported rather than hardcoded.

**The intended contributor workflow:** fill in the spec with the scenario and
what you think matters, hand it to your AI assistant of choice to review and
improve the spec, let it implement the test, then run it yourself and judge
whether it does what you wanted. The spec makes expected behaviour reviewable
while disagreements are cheap, and the human stays the one who decides the test
is right.

The thirteen specifications are in `e2e/specs/`. Three more were dropped during
implementation as redundant once the surrounding tests existed.

## Implemented Tests

| File | Covers |
| --- | --- |
| `test_smoke.py` | The app boots; the rater form is reachable. |
| `test_rater_form.py` | An empty rater ID and a submission with no panels selected are rejected. |
| `test_viewer_rating.py` | A valid form opens the viewer; a rating survives navigation; two tests describing how the configured default rating behaves. |
| `test_subject_list.py` | Rating a subject marks it complete; the search box filters the list. |
| `test_notes_persistence.py` | A note survives navigating away and back. |
| `test_viewer_autoplay.py` | Opening Add notes pauses autoplay. |
| `test_upload.py` | A previous QC file loads its records; unknown participants are rejected. |
| `test_checkpoint.py` | A checkpoint writes a snapshot of the ratings so far. |
| `test_final_export.py` | The export writes the results file; overwriting needs confirmation. |
| `test_multi_task.py` | Several tasks render on one page, plus the `xfail` below. |

## Continuous Integration

The suite has [its own workflow](https://github.com/nipoppy/qc-studio/blob/e2e/.github/workflows/e2e-tests.yml),
deliberately not run on every push — a real server and two browsers cost minutes
rather than seconds, and most changes cannot break it. It is asked for by
pressing **Run workflow**, with a choice of browser, or by adding a **`run-e2e`
label** to a pull request, which is the useful one during review: the result
reports on the PR itself and keeps running on later pushes.

Playwright binaries are cached, keyed on the requirements file so a version bump
fetches fresh ones. A failing run uploads screenshots and video, which is the
point of running in CI at all: seeing what broke on a machine you cannot open a
browser on.

## An Environment Lesson

`requirements.txt` asks only for `streamlit>=1.56`. A local environment on 1.58
and a CI runner on 1.65 produced different markup for radio widgets, and sixteen
tests that passed locally failed in CI.

The fix was on the suite side: widgets are now located through their ARIA role
and accessible name rather than the component library's internal attributes,
which is stable across both. One file changed and no test bodies were touched —
which is what the shared selector layer is for. The unpinned dependency remains.

## Shared Conventions for Contributors

The spec-first workflow only helps if contributors follow the same conventions,
so the branch commits them: `.claude/skills/` holds orientation for this suite
and for Playwright, plus general review and test-writing conventions for the
repository, with matching agents in `.claude/agents/` and a repo-level
`AGENTS.md` pointing other coding agents at the same guidance.

They are committed rather than kept local on purpose — anyone using an AI
assistant on this repository then gets the same behaviour instead of each
contributor's assistant inventing its own. By contrast `settings.local.json` is
deliberately not committed, since a personal permission allowlist is specific to
one machine.

## Status and Next Steps

The PR is open for review. What remains is less about the suite than about
decisions around it: agreeing the scope and placement of the committed agent
skills, and acting on the product questions the suite turned up, which are recorded
in `e2e/FINDINGS.md` and pinned where it matters by `xfail(strict=True)` tests.

Earlier review points have been addressed — the rerun waits, export tracking for
overwritten filenames, "CSV" wording where the app writes TSV, and smoke
assertions that bypassed the shared helpers — and `--browser` now replaces the
configured defaults rather than adding to them.

See the [GSoC Final Report](GSoC_FINAL_REPORT.md) and
[appendices](GSoC_REPORT_APPENDIX.md).
