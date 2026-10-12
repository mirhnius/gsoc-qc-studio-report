# Google Summer of Code 2026: QC-Studio Contribution Report

**Contributor:** Niusha ([mirhnius](https://github.com/mirhnius))  
**Project:** [QC-Studio](https://github.com/nipoppy/qc-studio)  
**Status checked:** October 12, 2026

## Summary

QC-Studio is a Streamlit application for reviewing neuroimaging data. It brings
MRI images, 2D montages, image quality metrics (IQMs), and structured ratings
into one review workflow. My work made that workflow more informative and more
dependable: I built the panel that puts quantitative quality metrics next to the
images, fixed a bug that silently lost reviewers' ratings, set up the project's
automated checks, started a browser-level test framework, and reviewed a large
share of the code other contributors added.

- **Features and fixes merged:** the IQM viewer (#22), the fix for ratings lost
  during autoplay (#76), SVG montage caching (#62), basic CI checks (#39), and
  terminology and documentation fixes (#34, #66).
- **Testing infrastructure:** a Playwright framework with 11 behavior
  specifications and six working tests, in an open draft PR (#88).
- **Research:** an evidence-bundle sandbox that packages MRI QC evidence for
  LLMs, with cloud and local model experiments and documented findings.
- **Code review:** 14 PRs by other contributors, including detailed reviews of
  saving, checkpoints, filtering, multifacet ratings, caching, and
  documentation. Contributors acted on these reviews, and several issues I
  reported were fixed and closed.
- **Upstream impact:** a Niivue issue (#314) I filed led to the loading events
  I asked for being implemented in master.
- **Validated by real use:** new users ran QC on the PPMI dataset on October 7–9,
  2026, successfully. Making the application dependable for sessions like that
  one, and reviewing the PRs that changed the paths they depended on, was the
  work the project most needed from me.
- **Project direction:** two things reset the priorities mid-project. The PPMI
  onboarding made the app's reliability for real review sessions and the review
  of incoming PRs the urgent work; and the
  infrastructure needed to experiment with open-weight models was not available,
  so the AI panel could not progress past the sandbox. Full Nipoppy
  `DatasetLayout` and manifest integration, production outlier detection, and the
  AI-summary dashboard panel are left as future work, with the sandbox and IQM
  infrastructure as the starting point.

| Activity | Count (October 12, 2026) |
| --- | --- |
| PRs authored | 9: 6 merged, 2 open drafts, 1 closed prototype |
| Issues created in QC-Studio | 21: 9 closed, 12 open |
| PRs reviewed or commented on | 14 |
| PRs merged / closed without merge under my account | 7 / 5 |
| Issues closed under my account | 11 |
| Upstream issues | 1 in niivue/niivue-vscode, closed |

Categories overlap: a PR I authored and merged is counted in two rows, and
closures include issues closed automatically by merging a linked PR. Individual
records are in the [appendices](GSoC_REPORT_APPENDIX.md).

## How the Project Evolved

My proposal had three deliverables:

- A config-driven architecture using Nipoppy's `DatasetLayout` API, dynamic
  subject loading, unified pagination, and `manifest.tsv` integration.
- An IQM panel comparing subject metrics with local and MRIQC reference
  distributions, including rule-based outlier detection.
- An experimental AI panel displaying pre-generated summaries, with an
  evaluation of their factual accuracy and effect on the review workflow.

Two developments during the project changed which of these mattered most.

**Real users arrived.** New users were preparing to run QC on the PPMI dataset,
which turned reliability from a quality concern into a blocking one. An AI
summary panel on top of an application that could drop a reviewer's rating would
not have been worth building. Rating persistence, navigation, filtering, saving,
checkpoints, and notes all needed attention, and contributors were adding
features in those same areas at the same time. Reviewing those PRs carefully
became a priority in its own right, since a defect merged into the save or
resume path would reach real raters working on a real dataset. That is why
implementation, code review, regression testing, and browser-test infrastructure
became the largest share of my work.

Those sessions ran on October 7–9, 2026, and they went successfully. That is the
clearest measure I have of whether this shift was the right one: the application
held up for real raters on a real dataset. The timing also shows how close the
margin was. I retested the multifacet ratings work on October 6, covering task
switching and wrong-task checkpoint uploads, and that PR merged on October 8,
in the middle of the QC window.

**The compute for open-weight models was not there.** The AI deliverable assumed
I could experiment with open-weight models locally. In practice the
infrastructure for that was not available: the models I could actually run on my
laptop were too small for the task, and the larger ones would not run at all.
Section 5 describes what this allowed and what it ruled out. The work therefore
stopped at a sandbox, which is an honest finding about what this line of research
needs rather than a result I could extend by trying harder.

| Original goal | What I built | Where it stands |
| --- | --- | --- |
| Nipoppy-integrated architecture | Per-task IQM source configuration, adaptation of the viewer to pagination, and reviews of cohort navigation, record identity, and resume behavior in others' PRs. | Configuration and integration work is in the app. Full `DatasetLayout` and `manifest.tsv` integration is future work; pagination implemented by others is credited to them. |
| IQM distributions and outlier detection | Merged IQM viewer with dataset distributions, scan-level tables, multiple sources, DWI support, and tests; reference-data infrastructure; metric highlighting in the evidence sandbox. | Dataset IQM panel is live. Reference comparison is built and will be re-enabled once reference data is cleaned (#82). Production outlier detection builds on the sandbox rules. |
| AI summaries and evaluation | Structured evidence packages, model experiment scripts, response-schema validation, and findings on cloud and local models. | A working sandbox with documented findings. Blocked from going further by the lack of infrastructure for open-weight models; dashboard integration and a larger evaluation need that compute first. |

Work added beyond the original proposal: the autoplay persistence fix, CI and local
checks, SVG caching, the draft Playwright framework, workflow reviews, and the
upstream Niivue report.

## 1. IQM Viewer: From a Metric Dropdown to a Per-Task Panel

**Delivered:** [PR #22](https://github.com/nipoppy/qc-studio/pull/22), merged
September 10, 2026, after 45 commits across 112 files. Related:
[#12](https://github.com/nipoppy/qc-studio/issues/12),
[#23](https://github.com/nipoppy/qc-studio/issues/23).
Details: [IQM Viewer Branch](IQM_VIEWER_BRANCH_EXPLANATION.md).

Reviewers could see images but not the numbers MRIQC produces for them. Issue
#12 asked for IQM distributions with MRIQC reference comparison, and my first
implementation did exactly that: a metric dropdown, a dataset distribution, and
a reference population fetched from the MRIQC Web API. Getting that working was
the easy part. Nearly all of the design work came from three problems it
exposed.

**Problem 1: the reference data could not be trusted or even downloaded.**
Fetching from the MRIQC Web API ran for two full days without completing, and
the partial result was too large to commit. I rewrote the lookup to read
cached, filtered Parquet instead of querying the API per view, which made the
panel usable. But extreme values in the population stretched the plot axes
enough to make the dataset distribution unreadable, so clipping became
necessary. At that point I concluded that loading the data successfully was not
the same as displaying it honestly. Rather than ship a comparison I could not
vouch for, I filed [#82](https://github.com/nipoppy/qc-studio/issues/82),
hid the "Dataset + reference" mode, and kept the infrastructure in place behind
it. Hiding a feature I had already built was the right call: a plausible-looking
but wrong reference curve is worse for a reviewer than no curve at all.

**Problem 2: one dropdown per metric did not match how reviewers work.** In
review, Nikhil pointed out that selecting metrics one at a time was cumbersome.
I tried an overview/detail layout with a scrollable overview, then simplified to
a metric grid inside each source tab, so a reviewer sees the whole picture
first. I also replaced box plots with violins (which show the shape of a
distribution, not just its quartiles), gave each tab one shared legend with
consistent shapes and colors, and scoped the current-subject marker to the
active run with hover text naming it. That last detail mattered more than it
sounds: a distribution with an unscoped marker tells a reviewer nothing about
which measurement belongs to the scan in front of them, and an early review
comment caught exactly that case, where the highlighted subject was not visible
at all.

**Problem 3: the configuration encoded MRIQC assumptions.** The original design
had a top-level `iqm_distributions` setting with hardcoded `t1w` and `bold`
groupings. Nikhil observed that the modality should follow from the QC task
rather than being picked in the UI, and that pointed at the deeper issue: the
feature was structured as an MRIQC add-on instead of a property of a QC task. I
replaced it with a per-task `iqm_path` list, deliberately mirroring the existing
`svg_montage_path` pattern so the configuration would be predictable to anyone
who already knew the project. Each source became its own tab; MRIQC-like
sources use curated metric groups, while others fall back to generic per-column
display, so a non-MRIQC pipeline is degraded rather than unsupported. DWI
support then followed from extending the group registry and inferring the
modality, not from new display code, which is the practical test of whether that
refactor was worth doing.

The review also pushed the module boundaries. I had put the reference maps and
download helpers in `ui/utils/data_loaders.py`; Nikhil argued that a loader
module should not import `streamlit` and `requests` or hold configuration maps.
I agreed and moved both into a dedicated `ui/utils/reference_data.py`. A final
review round caught that `URL_PARENT` was resolved at import time, before the
app's environment was in place, so I moved the resolution to call time.

Alongside the feature I added tests for metric grouping, source loading, widget
state, plot rendering paths, and DWI groups, plus sample MRIQC inputs and a DWI
test configuration, since a viewer with no fixtures cannot be checked by anyone
else.

## 2. Rating Persistence: Reading the Widget Too Late

**Delivered:** [PR #76](https://github.com/nipoppy/qc-studio/pull/76), merged
September 2, 2026, fixing [#73](https://github.com/nipoppy/qc-studio/issues/73).

A reviewer reported that some ratings chosen during autoplay were missing from
the saved output. The report was intermittent, which is the hardest kind of bug
to act on, and the first useful step was framing it as a timing problem rather
than a save problem: I wrote #73 around the hypothesis that clicks near the
automatic-advance boundary were the ones being lost.

Reading the code confirmed it and explained why it was intermittent. A rating
click never saved itself. It only reached the saved record when the autoplay
timer next polled `session_state`. Each page advance rotated the widget key, so
a click that landed after the timer had already advanced was read from a key
that no longer belonged to the subject being rated, and the rating vanished with
no error. Nothing was broken in the save path, which is why the bug survived
ordinary testing: it depended entirely on when the click happened.

That diagnosis made the fix structural rather than defensive. The rating now
saves itself through the widget's `on_change` callback, so persistence no longer
depends on the timer at all. Two safeguards back it up: a guard so an empty
widget read cannot overwrite an existing rating, and a 0.3-second grace period
(`AUTOPLAY_ADVANCE_GRACE_SECONDS`) before the poll treats the interval as
elapsed, giving a click made right at the boundary time to reach the server and
save itself. The PR is roughly 870 added lines against 15 removed, because most
of it is regression tests: a test that fails under the original conditions is
the only way to show a timing bug is actually gone.

The same investigation surfaced related gaps in notes and partial progress,
which I reported as [#11](https://github.com/nipoppy/qc-studio/issues/11),
[#81](https://github.com/nipoppy/qc-studio/issues/81), and
[#84](https://github.com/nipoppy/qc-studio/issues/84). All three are now closed;
other contributors implemented those fixes.

## 3. Performance and the Project's First Automated Checks

**SVG caching** ([PR #62](https://github.com/nipoppy/qc-studio/pull/62), closed
[#43](https://github.com/nipoppy/qc-studio/issues/43)). Montages were being
re-read and re-rendered on every widget interaction. Caching with
`@st.cache_data` is the obvious move; making the cache key correct is the real
work. The path list had to be normalized to a tuple before Streamlit could use
it as a stable key, and I kept `load_svg_data()` as the public entry point while
moving the expensive file loading, image conversion, and montage assembly into a
cached helper, so callers did not change. I also decided *against* adding
`st.fragment` in the same pass, and said so in the PR: fragmenting the
rating/notes panel risked widget state going stale when the participant or
session changed, which is precisely the class of bug I had just fixed in #76. I
recorded my uncertainty rather than hiding it, and noted that `load_svg_data()`
was doing too much and deserved a follow-up refactor separating path
normalization, loading, conversion, and montage creation.

**CI and local checks** ([PR #39](https://github.com/nipoppy/qc-studio/pull/39),
from my own proposal in [#35](https://github.com/nipoppy/qc-studio/issues/35)).
The project had no automated checks, so the first decision was how much to
introduce at once. I deliberately staged it: formatting, linting, and spelling
(Black, Flake8, codespell) in the first PR, with the test workflow left as an
explicit follow-up in the checklist, because which tests belong on every PR is a
question for the maintainers, not a thing to decide unilaterally in a
configuration PR. Introducing a formatter to an existing codebase also means a
large diff, which is a bad thing to bundle with a behavioral change.

The local setup matters as much as the CI side. Pre-commit runs the same tools
with the same configuration files, so the hooks and the CI job cannot disagree,
and the file filters are scoped to `ui/` and `pipelines/` Python plus specific
config files, so whitespace hooks do not start rewriting sample data. I added a
local pytest hook so UI tests run before a commit rather than after a push. The
staged plan also recorded what I was choosing not to do yet: separating
lightweight from slow or data-dependent tests, and coverage reporting. Later,
reviewing [#105](https://github.com/nipoppy/qc-studio/pull/105), I argued for
keeping a documentation build in its own workflow for the same reason, so a docs
failure cannot block code tests.

Two smaller maintenance PRs came out of working in the code:
[#34](https://github.com/nipoppy/qc-studio/pull/34) replaced SVG-specific
wording with montage terminology once the panel handled 2D images generally, and
[#66](https://github.com/nipoppy/qc-studio/pull/66) repaired broken
documentation links.

## 4. Playwright Testing: Writing the Specification First

**In progress:** [PR #88](https://github.com/nipoppy/qc-studio/pull/88), open
draft, 13 commits across 38 files. Related:
[#75](https://github.com/nipoppy/qc-studio/issues/75).
Details: [Playwright E2E Branch](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md).

Three of the bugs I had worked on (#73, #81, #84) shared a shape: the logic was
correct in isolation, and the failure lived in the interaction between real
widgets, session state, and Streamlit's rerun cycle. The existing `ui/tests/`
suite mocks Streamlit, so by construction it cannot see that class of bug. That
is the argument for browser tests, and I made it in #75 rather than starting to
write tests and justifying them afterwards.

The first design decision was scope: these tests are marked and run separately
from the fast suite, because they need browser binaries and a real server, and a
test layer that makes the quick feedback loop slow gets abandoned.

The harness is built so a test does not have to know how the app starts. Pytest
fixtures allocate a free port, launch the app with `streamlit run`, wait on its
health endpoint, point output at a temporary directory so a test run cannot touch
the repository's `output/`, and tear the server down afterwards. Widget
selectors and interactions live in one shared module, with the awkward details
handled once: ordinary buttons behave differently from form-submit buttons, and
alert text may arrive with the leading emoji stripped. Tests reference
user-facing strings through `ui/constants.py` instead of copying them, so a
label change does not quietly break the suite.

The hardest part is synchronization. Streamlit reruns the script after an
interaction, so an assertion made immediately after a click can inspect the
previous render. Negative assertions are the dangerous case, because "the
element is gone" and "the new render has not arrived" look identical and the
test passes for the wrong reason. The waits module centralizes this, waiting for
the script state to reach `notRunning`, which is also the one part reviewers
have pushed back on and where I expect to keep iterating.

I also chose to write a specification before each test. Each one records the
screen, category, priority, why the behavior needs a browser, Given/When/Then
steps, and the relevant copy and constants. The point is that expected behavior
becomes reviewable before any Playwright code exists, which is where
disagreements are cheap to settle, and the 11 specifications now describe the
risky paths (validation, ratings surviving navigation, upload and resume,
overwrite protection, checkpoints, notes, autoplay) whether or not each has an
implemented test yet. Six scenarios are implemented in four files, and they were
not chosen at random: alongside two smoke checks and two rater-form validation
checks, notes persistence across navigation and notes pausing autoplay are the
browser-level versions of bugs #84 and #72. Some tests disable Niivue so they
test the workflow rather than waiting on volume rendering.

Because the specification-first workflow only helps if others follow it, the PR
also includes the Claude Code skills and agents that encode those conventions
for future contributors. Their scope is still open for discussion with the
maintainer and could reasonably be split into a separate PR.

Status: the PR description records passing Chromium smoke tests against a real
browser. Next steps are tightening the rerun waits, tracking overwritten
exports, and implementing the save/resume scenarios the specifications already
describe.

## 5. Evidence Bundles for LLM-Assisted QC

**Experimental:** [`evidence_bundle` branch](https://github.com/nipoppy/qc-studio/tree/evidence_bundle).
Related: [#24](https://github.com/nipoppy/qc-studio/issues/24),
[#33](https://github.com/nipoppy/qc-studio/issues/33),
[#40](https://github.com/nipoppy/qc-studio/issues/40).
Details: [Evidence Bundle Branch](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md).

The proposal's AI panel assumed pre-generated summaries, but it left the harder
question open: what does a model actually need to see in order to say anything
defensible about a scan? I treated that as the research question and worked
backwards from how a human reviews, which is never from a bare list of numbers.
A reviewer reads a metric against context: what the metric means, how this scan
compares with a population, and how much weight the measurement deserves.

So the evidence package carries that context explicitly. Shared helpers load
BIDS and scanner metadata, the scan's MRIQC metrics, and reference tables, filter
the reference rows by scanner metadata, and compute percentiles and z-scores.
Each metric is then paired with an interpretation and its caveats, and the
package records direction of concern and evidence strength per metric. Rather
than ask the model to decide what to look at, the builder highlights metrics
flagged as important or sitting beyond the 10th or 90th percentile or an absolute
z-score of 1.5. Those are deliberately simple experimental rules, chosen so that
what the model was shown is reconstructable afterwards instead of hidden in a
prompt.

The second decision was to constrain the output and validate it. The response
schema requires a decision, confidence, summary, flagged metrics, reasons, and
recommended follow-up, checked with Pydantic, and every run keeps both the
package and the response (`PACKAGE.md`, `result.json`, `result_error.json`) so a
failure can be traced to the evidence that produced it. I chose Markdown for the
packages because being able to read what the model read mattered more during
development than compactness, while noting JSON as the likely production format.

The limiting factor turned out to be infrastructure rather than design. The
proposal assumed I could iterate on open-weight models locally, and that was not
possible on the hardware available to me. Apple MPS hit memory and allocation
failures, so the 4B model had to run on CPU, which made each iteration slow
enough to discourage the repeated runs an evaluation needs; larger open-weight
models were out of reach entirely. What remained in range were models small
enough to test the pipeline but too small to do the task, plus cloud models for
the IQM-only text experiments. Setting up or obtaining GPU infrastructure was
outside what I could do within the project, so I treated the limit as a result
to record: this line of work needs that infrastructure in place before an
evaluation is meaningful.

That setup is what made the findings interpretable. Across three MR-ART
acquisitions of one subject, the responses were `uncertain`, `uncertain`, and
`fail`, with the clearly degraded scan the easiest to call. The more useful
result came from the local LLaVA-OneVision runs: schema-valid JSON that still
confused metric values with percentiles, flagged metrics that did not match its
own prose, and used `direction_of_concern` inconsistently. Format validity and
correct reasoning are separate problems, and only the first is cheap to check
automatically. That is the main thing this sandbox establishes, along with a
reusable structure for the larger labeled evaluation it would take to say
anything about accuracy. The dataset is public, so training overlap is possible,
and the generated metric explanations still need expert review.

## 6. Code Review and User-Workflow Testing

I combined code reading with manual checks of real review sessions, focusing on
whether ratings and notes survive navigation, filtering, saving, and resuming. I
gave concrete failure cases and suggestions and, where possible, retested fixes
before approving. Fixes in others' PRs are credited to their authors. The
[review appendix](GSoC_REPORT_APPENDIX.md#appendix-c-review-and-comment-activity-on-others-prs)
and [review detail](GSoC_CONTRIBUTION_DOSSIER.md) list every comment.

**Larger reviews**

- [#20](https://github.com/nipoppy/qc-studio/pull/20) (initial pagination): my
  first in-depth review on the project, with many comments on the pagination
  code. GitHub still shows that review as pending, so the comments were never
  published there. Nikhil
  [thanked me by name for them](https://github.com/nipoppy/qc-studio/pull/20#issuecomment-4413781991)
  alongside the other reviewers and said he had addressed them all before the
  PR was approved for merge. The pagination implementation is his.
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

Two related reports: [#101](https://github.com/nipoppy/qc-studio/issues/101)
(Streamlit minimum version for `st.iframe`, closed after #95) and
[#112](https://github.com/nipoppy/qc-studio/issues/112) (overwrite warning
versus retained export rows, open).

Reviewing #95 and #83 mattered on a schedule, not just in principle. Both
changed the save, checkpoint, and resume paths, and the PPMI QC sessions on
October 7–9 depended on those paths; #95 merged on October 8, after I retested
task switching and wrong-task checkpoint uploads on October 6.

This work also informed the Playwright framework: notes persistence and
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

- **Streamlit's execution model is the source of most of these bugs.** Reruns,
  widget keys and session state interact in ways that unit tests with mocks
  cannot see, which is why #76 was a timing fix rather than a save fix and why I
  argued for a browser-test layer.
- **Where code lives changes whether it can be reused.** The IQM refactor from a
  global MRIQC setting to a per-task `iqm_path`, and moving reference handling
  out of the loader module, were both prompted by review and both made later
  work (DWI support, the evidence-bundle percentiles) cheaper.
- **Deciding what not to ship is part of the work.** Hiding the reference
  comparison pending #82, declining `st.fragment` in #62, and staging CI in two
  PRs were all judgment calls I recorded with reasons so they could be argued
  with.
- **Reproducibility is what makes a finding useful to someone else.** Turning
  intermittent reports into reproducible issues, regression tests that fail
  under the original conditions, and readable specifications is what let other
  contributors act on what I found.
- **Review is a design conversation.** The strongest parts of #22 came out of
  Nikhil's review comments, and my own reviews aimed to do the same thing: name
  the concrete failure case, then retest it.
- Open-source maintenance in practice: follow-up, verification, merging, and
  closing superseded work. I also helped clarify expected save and navigation
  behavior in [issue #89](https://github.com/nipoppy/qc-studio/issues/89#issuecomment-5674629745)
  (automatic snapshots were a proposal, not a delivered feature).

## Next Steps

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

Beyond these, the proposal's full Nipoppy layout and manifest integration,
production outlier detection, AI-summary dashboard panel, and workflow-impact
evaluation are natural follow-ons to the IQM viewer and evidence-bundle work.

## About This Report

Counts and status are as of October 12, 2026, checked against the repository's
PR, issue, review, merge, and closure history. The original proposal summary,
the PPMI onboarding and its outcome, and the hardware limits on open-weight
model experiments come from my own records rather than that history. Where a fix
was implemented by another contributor, it is credited to them throughout.

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
