# Google Summer of Code 2026: QC-Studio Contribution Report

**Contributor:** Niusha ([mirhnius](https://github.com/mirhnius))  
**Project:** [QC-Studio](https://github.com/nipoppy/qc-studio)  
**Status checked:** October 12, 2026

## Background

Neuroimaging workflows produce a wide variety of artifacts — volumes, surfaces,
networks, tables, figures, and summary metrics — and these often need to be
reviewed for every participant and every visit. In current practice that means
opening files by hand, in one tool per output type, inspecting them visually,
and noting quality ratings in a spreadsheet. That process is slow, error-prone,
and hard to scale to a modern dataset. It is also the step that stands between a
processed dataset and an analysis-ready one, so when it becomes a bottleneck,
the whole study waits.

[QC-Studio](https://github.com/nipoppy/qc-studio) addresses this directly. It is
a web-based quality control application, part of the
[Nipoppy](https://github.com/nipoppy) ecosystem, that gives raters one portable
interface — a browser, used locally or over SSH — for reviewing raw BIDS data,
processed pipeline derivatives, and image quality metrics together, and for
exporting their ratings as a tab-separated table. Because no single QC protocol
fits every study, reviews are defined as configurable "QC tasks" in a `qc.json`
file, which can be specific to a study, datatype, or pipeline; ready-made
configurations ship for fMRIPrep, FreeSurfer, fsqc, QSIPrep, XCP-D, and
NODDIreg. A QC page can show up to three panels: a 3D MRI viewer built on
NiiVue, a 2D image montage, and a QC metrics panel showing IQM distributions.
The project's own framing is that neuroimagers should be able to *QC anything,
anywhere, all at once*.

The application's stated goals are to visualize both raw and processed
neuroimaging data in an interactive web app, to support 3D MRI (NIfTI), 2D
image montages, and IQM metrics, and to make structured QC ratings possible
through a clean, intuitive interface.

## Project Goals

Two things follow from that motivation and shaped this GSoC project. First, a
reviewer needs quantitative evidence next to the images, because judging a scan
by eye alone discards the measurements a pipeline such as MRIQC has already
produced. Second, and less obvious until the application is used on a real
dataset, the review session itself has to be dependable: a rating or a note that
does not survive navigation, automatic advance, saving, or resuming is worse than
no tool at all, because the reviewer cannot tell that anything was lost.

My proposal set out three deliverables against that motivation:

- A config-driven architecture using Nipoppy's `DatasetLayout` API, dynamic
  subject loading, unified pagination, and `manifest.tsv` integration.
- An IQM panel comparing subject metrics with local and MRIQC reference
  distributions, including rule-based outlier detection.
- An experimental AI panel displaying pre-generated summaries, with an
  evaluation of their factual accuracy and effect on the review workflow.

The aim behind all three was to reduce the burden of manual QC and to produce
ratings that are reliable and documented. Which of them mattered most changed
during the project, for reasons described in
[How the Project Evolved](#how-the-project-evolved).

## Summary

My work made QC-Studio's review workflow more informative and more
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

Two developments during the project changed which of the three proposed
deliverables mattered most.

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

## 1. IQM Viewer

**Merged:** [PR #22](https://github.com/nipoppy/qc-studio/pull/22), September 10,
2026. Closes [#12](https://github.com/nipoppy/qc-studio/issues/12) and
[#23](https://github.com/nipoppy/qc-studio/issues/23).
Full detail: [IQM Viewer Branch](IQM_VIEWER_BRANCH_EXPLANATION.md).

Reviewers could see images but not the numbers MRIQC produces for them.

**From MRIQC-only to any pipeline.** The first version supported MRIQC across
its modalities, configured by a top-level setting that hardcoded groupings such
as `t1w` and `bold`. That made the panel an MRIQC add-on rather than a property
of a QC task, so I redesigned it around a per-task `iqm_path` list — mirroring
the existing `svg_montage_path` pattern — pointing at individual JSON files.
A task can now declare several sources, each shown in its own tab, so a pipeline
other than MRIQC is supported with its own metrics, and different metric sets,
version-suffixed pipeline names, and modalities can be displayed. MRIQC-like
sources use curated metric groups; others fall back to generic per-column
display, so an unfamiliar pipeline is degraded rather than unsupported. DWI
support then followed from extending the group registry, with no new display
code — the practical test of whether the refactor was worth doing.

**Reference data: built, but not trustworthy yet.** To compare a scan against a
population, I wrote the code to collect reference distributions from the MRIQC
endpoint. It ran for two days without finishing, so I used the partial download
as-is, and only for the structural modality. The loader caches filtered Parquet
with a seven-day refresh and normalizes scanner metadata, so a comparison is
drawn against similar acquisitions rather than everything.

The problem was what the data showed: the MRIQC population distribution was
markedly different from the dataset being reviewed. A comparison like that is
not informative, and could mislead a reviewer into doubting a perfectly good
scan. So the capability exists but is disabled, and
[#82](https://github.com/nipoppy/qc-studio/issues/82) tracks the cleanup and
testing it needs. Part of the fix is a rule rather than more data: if the two
distributions are that far apart, the viewer should not show the comparison at
all.

**Display.** Choosing one metric at a time from a dropdown was cumbersome, and
showing everything at once was too dense, so each source tab holds a metric
grid. Violin plots replaced box plots to show distribution shape, each tab has
one shared legend, and the current-subject marker is scoped to the active run —
an unscoped distribution tells a reviewer nothing about which measurement is
theirs.

## 2. Rating Persistence

**Merged:** [PR #76](https://github.com/nipoppy/qc-studio/pull/76), September 2,
2026, fixing [#73](https://github.com/nipoppy/qc-studio/issues/73).

A reviewer reported that ratings chosen during autoplay sometimes went missing.
The report was intermittent, so the first useful step was framing it as a timing
problem: I wrote #73 around the hypothesis that clicks near the automatic-advance
boundary were the ones being lost.

The code confirmed it. A rating click never saved itself; it only reached the
record when the autoplay timer next polled `session_state`. Each advance rotated
the widget key, so a click landing after an advance was read from a key that no
longer belonged to the subject being rated, and the rating disappeared with no
error. Nothing in the save path was broken, which is why ordinary testing missed
it.

That made the fix structural. The rating now saves through the widget's
`on_change` callback, so persistence no longer depends on the timer. Two
safeguards back it up: a guard preventing an empty read from overwriting an
existing rating, and a 0.3-second grace period before the poll treats the
interval as elapsed. Most of the PR is regression tests, since a test that fails
under the original conditions is the only way to show a timing bug is gone.

The same investigation surfaced related gaps in notes and partial progress,
reported as [#11](https://github.com/nipoppy/qc-studio/issues/11),
[#81](https://github.com/nipoppy/qc-studio/issues/81), and
[#84](https://github.com/nipoppy/qc-studio/issues/84). All three are closed;
other contributors implemented those fixes.

## 3. Performance and the Project's First Automated Checks

**Montage caching** ([PR #62](https://github.com/nipoppy/qc-studio/pull/62),
closes [#43](https://github.com/nipoppy/qc-studio/issues/43)). Montages were
re-read and re-rendered on every widget interaction. Caching is the obvious move;
the real work is the cache key, which required normalizing the path list to a
tuple so Streamlit could use it. I kept `load_svg_data()` as the public entry
point and moved the expensive loading, conversion, and assembly into a cached
helper, so callers did not change. I deliberately left `st.fragment` out of the
same pass and said why in the PR: fragmenting the rating and notes panel risked
widget state going stale when the participant changed, which is the bug class I
had just fixed in #76.

**CI and local checks** ([PR #39](https://github.com/nipoppy/qc-studio/pull/39),
from my proposal in [#35](https://github.com/nipoppy/qc-studio/issues/35)). The
project had no automated checks, so the question was how much to introduce at
once. I staged it: Black, Flake8, and codespell first, with the test workflow
left as an explicit follow-up, because which tests gate every PR is a maintainer
decision rather than something to settle inside a configuration PR. Adding a
formatter to an existing codebase also produces a large diff, which should not
ride along with a behavioral change.

Pre-commit runs the same tools against the same configuration files as CI, so
hooks and CI cannot disagree, and the file filters are scoped to `ui/` and
`pipelines/` Python plus named config files so whitespace hooks do not rewrite
sample data. A local pytest hook runs the UI tests before a commit instead of
after a push.

Two smaller maintenance PRs came out of working in the code:
[#34](https://github.com/nipoppy/qc-studio/pull/34) replaced SVG-specific
wording with montage terminology once the panel handled 2D images generally, and
[#66](https://github.com/nipoppy/qc-studio/pull/66) repaired broken
documentation links.

## 4. Playwright Browser Testing

**Open draft:** [PR #88](https://github.com/nipoppy/qc-studio/pull/88), from
[#75](https://github.com/nipoppy/qc-studio/issues/75).
Full detail: [Playwright E2E Branch](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md).

Three bugs I had worked on (#73, #81, #84) shared a shape: the logic was correct
in isolation, and the failure lived in the wiring between real widgets, session
state, and Streamlit's rerun cycle. `ui/tests/` mocks Streamlit out, so by
construction it cannot see that class of bug. We needed end-to-end tests that
run automatically, and I argued that in #75 before writing any.

Playwright was new to me, so instead of inventing a structure I studied how
Streamlit's own `e2e_playwright` suite is organised — particularly how it stops
tests breaking when Streamlit's internals change, by wrapping the fragile parts
so there is one place to fix rather than many. The suite has four layers, each
hiding its mechanics from the one above: a fixture that launches the real app, a
fixture that hands over a loaded page, a shared layer owning every selector and
every rerun wait, and the tests themselves, which know about none of it.

The hard part is synchronization, because Streamlit reruns the whole script on
every interaction and a negative assertion can pass against the stale render —
a green test over a shipped bug. The waits ask the app for its script state
rather than guessing with timeouts.

I was also curious about spec-driven development, and this is the concept rather
than the full method. A contributor fills in a Markdown spec with the scenario
and what matters about it, hands it to their AI assistant to review and improve,
lets it implement the test, then runs it themselves and judges whether it does
what they wanted. The spec makes expected behavior reviewable while
disagreements are still cheap, and the human stays the one who decides the test
is right. For that to work across contributors there has to be shared agreement,
so the branch commits skills for writing tests and reviewing code — committed
rather than local, so everyone's assistant behaves the same way.

The 11 specifications cover the risky paths; six scenarios are implemented, two
of them the browser-level versions of #84 and #72. Next: tighten the rerun
waits, track overwritten exports, and implement the save/resume specs.


## 5. Evidence Bundles: An LLM Companion for QC

**Experimental:** [`evidence_bundle` branch](https://github.com/nipoppy/qc-studio/tree/evidence_bundle),
issues [#24](https://github.com/nipoppy/qc-studio/issues/24),
[#33](https://github.com/nipoppy/qc-studio/issues/33),
[#40](https://github.com/nipoppy/qc-studio/issues/40).
Full detail: [Evidence Bundle Branch](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md).

**The purpose was a companion, not an automated rater.** Deciding whether a scan
passes is a hard problem, and the right tool would be a model trained for it — a
classifier or deep-learning model — not a general language model. Even
purpose-trained models are not reliably successful at it, so asking an LLM for
the final verdict is the wrong use of it. What an LLM can usefully do is point
out the possible issues and say what to follow up on: what to look at next, and
what the reviewer should judge for themselves. The decision stays with the
human, and that framing shaped the whole design.

**The package.** For one subject and session, the builder assembles a Markdown
file holding the IQMs that matter — statistical outliers plus those important
for that modality — the statistics around each, and the instructions telling the
model how to behave. The per-metric context comes from a dictionary I built by
extracting the MRIQC documentation with Codex: what each metric means, which
direction is concerning, when it is informative. A metric therefore reaches the
model already interpreted rather than as a bare number.

**The intended product.** On starting a session, a batch of subjects would be
sent to the model, and each subject's summary of possible problems and suggested
next steps would be shown as a pop-up when the reviewer reaches them.
Summarization first, with a chat interface for follow-up questions later.

**Where it stopped.** The text stage worked: the Claude API read the package and
returned a structured response, validated with Pydantic. Images were the next
step, and that needed the local vision model the project intended to use. I
could not run it on my own machine, and the server for it was not yet available,
so the work stopped there rather than at a design dead end.

**What the experiments showed.** The small local models returned schema-valid
JSON that still confused metric values with percentiles and flagged metrics
contradicting their own prose. Format validity and correct reasoning are
separate problems, and only the first is cheap to check automatically — which is
the strongest argument for keeping this a companion that surfaces things to
check rather than anything that produces a rating.


## 6. Code Review and Workflow Testing

I combined code reading with manual checks of real review sessions, focusing on
whether ratings and notes survive navigation, filtering, saving, and resuming.
Where possible I retested fixes before approving. Implementations in other
contributors' PRs are theirs; these are my review findings. Comment-level records
are in [Appendix C](GSoC_REPORT_APPENDIX.md#appendix-c-review-and-comment-activity-on-others-prs).

| PR | What I found |
| --- | --- |
| [#20](https://github.com/nipoppy/qc-studio/pull/20) pagination | My first review here, written in April before I had authored any code in the repo: a duplicated function, session state initialized twice, a missing `dataset_dir` argument, `panels_data` type handling, and a proposed `normalize_selected_panels()` helper to unify two sets of panel keys. |
| [#95](https://github.com/nipoppy/qc-studio/pull/95) multifacet ratings | Saved ratings masked by all-`None` widgets, facet-name changes breaking saved matching, checkpoint lookup crossing raters and tasks, stale task navigation, filename and wildcard-path edge cases. Retested task switching and wrong-task checkpoints before approving. |
| [#83](https://github.com/nipoppy/qc-studio/pull/83) saving and checkpoints | Filename and path inconsistencies, checkpoint confusion, altered leading-zero subject IDs, lost autoplay notes, page counts mislabeled as QC records. |
| [#74](https://github.com/nipoppy/qc-studio/pull/74) filtering and navigation | Autoplay leaving the active filter and moving to hidden subjects; a fully reviewed cohort skipping the congratulations page. |
| [#25](https://github.com/nipoppy/qc-studio/pull/25) pagination compatibility | Completed uploads sent back to page 1, stale Continue QC widgets, Start Over retaining records, silent duplicate handling, session assumptions, magic display constants. |
| [#111](https://github.com/nipoppy/qc-studio/pull/111) memory optimization | Two caching styles in two modules; asymmetric invalidation, since the NIfTI key includes file mtime and size but the montage key does not, so a regenerated montage can stay stale; a shared cached dict read-only only by docstring. |
| [#105](https://github.com/nipoppy/qc-studio/pull/105) documentation site | Verified a strict Sphinx build, demo startup, CLI flags, and export columns. README startup command missing required arguments, docs CI coupled to code tests, pre-commit scope reaching sample data, inconsistent saved-filename examples. |
| [#10](https://github.com/nipoppy/qc-studio/pull/10), [#53](https://github.com/nipoppy/qc-studio/pull/53), [#58](https://github.com/nipoppy/qc-studio/pull/58) | Overlay colorbar stacking; shortcut conflicts with typing in notes and wrapped button labels; review duration measuring time since last rerun rather than actual review time. |
| [#17](https://github.com/nipoppy/qc-studio/pull/17), [#38](https://github.com/nipoppy/qc-studio/pull/38), [#67](https://github.com/nipoppy/qc-studio/pull/67), [#96](https://github.com/nipoppy/qc-studio/pull/96) | Documentation, fixture, and citation PRs: reviewed and approved, or closed as superseded. |

Two related reports: [#101](https://github.com/nipoppy/qc-studio/issues/101)
(Streamlit minimum version for `st.iframe`) and
[#112](https://github.com/nipoppy/qc-studio/issues/112) (overwrite warning
versus retained export rows, open).

Reviewing #95 and #83 mattered on a schedule. Both changed the save, checkpoint,
and resume paths the PPMI sessions depended on, and #95 merged on October 8,
after I retested task switching and wrong-task checkpoint uploads on October 6.

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
closures) ·
[IQM viewer](IQM_VIEWER_BRANCH_EXPLANATION.md) ·
[Playwright](PLAYWRIGHT_E2E_BRANCH_EXPLANATION.md) ·
[Evidence bundles](EVIDENCE_BUNDLE_BRANCH_EXPLANATION.md)
