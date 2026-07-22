# MyThingsLab

A fleet of small tools that develop a GitHub repository on their own —
calling an LLM only for the one step that genuinely needs judgment, and
running everything else as deterministic code.

## The pattern

Every `My[X]` tool follows the same loop:

1. read a unit of work from a backlog (a GitHub issue),
2. run deterministic pre-work — lint, index, checks — with **no** model call,
3. call an LLM **once**, only for the step that needs judgment,
4. open a pull request, never a merge — a human (or a permission-scoped
   GitHub App) always holds merge authority,
5. append a structured outcome to a shared ledger so the next run can trust it.

[`my-things-core`](https://github.com/MyThingsLab/my-things-core) is the SDK
this is built on: five contracts (`ledger`, `policy`, `engine`, `github`,
`isolation`) that every tool imports and no tool reimplements. It calls no
LLM itself — `engine` is the single seam where a model is ever invoked, and
it's a `Protocol`, so a tool runs and tests its whole skeleton for free
before any real backend is wired in.

## Stability: v1 core vs. v0 tools

Most of the fleet is **v0** — build freely, float on `@main`, no version
discipline. A small core has graduated to **v1**: real semver
(`MAJOR.MINOR.PATCH`), a `CHANGELOG.md` entry per bump, a deprecate-before-remove
rule, and v1-to-v1 dependencies pinned to a tagged release instead of `@main`.
The contract lives in `my-things-core`'s
[`release.md`](https://github.com/MyThingsLab/my-things-core/blob/main/src/mythings/release.md).

| Repo | Does |
|---|---|
| [`my-things-core`](https://github.com/MyThingsLab/my-things-core) | The SDK: the five contracts every tool imports and no tool reimplements. |
| [`my-guard`](https://github.com/MyThingsLab/my-guard) | Rule engine — evaluates an `Action` to allow / ask / deny. |
| [`my-director`](https://github.com/MyThingsLab/my-director) | Human-in-the-loop end-of-day session: decides the next objective, decomposes it into task-issues. |
| [`my-fleet`](https://github.com/MyThingsLab/my-fleet) | Orchestration/ops tooling that chains every tool's CLI into one autonomous cycle. |
| [`my-dashboard`](https://github.com/MyThingsLab/my-dashboard) | Renders the fleet's org-wide dashboard and single-repo status cards. |
| [`my-reporter`](https://github.com/MyThingsLab/my-reporter) | Digests the ledger into a report and posts it as a comment. |

**Next in line for v1.** `my-fleet`'s
[`fleet_cycle.py`](https://github.com/MyThingsLab/my-fleet/blob/main/src/myfleet/fleet_cycle.py)
chains these into the cycle by name, and `my-orchestrator` is imported
directly by `fleet_dispatch.py` to pick the next unit of work — that direct
wiring is what puts them ahead of the rest of the v0 backlog for promotion:

| Tool | Wired in as |
|---|---|
| [`my-orchestrator`](https://github.com/MyThingsLab/my-orchestrator) | picks the next unit of work — imported directly by `fleet_dispatch.py` |
| [`my-planner`](https://github.com/MyThingsLab/my-planner) | cycle step 1 — ranks the backlog `my-orchestrator` reads |
| [`my-researcher`](https://github.com/MyThingsLab/my-researcher) | cycle step 3 — briefs open study topics |
| [`my-tester`](https://github.com/MyThingsLab/my-tester) | cycle step 4 |
| [`my-changelogger`](https://github.com/MyThingsLab/my-changelogger) | cycle step 5 |
| [`my-docs`](https://github.com/MyThingsLab/my-docs) | cycle step 6 |
| [`my-projector`](https://github.com/MyThingsLab/my-projector) | cycle step 8 |
| [`my-telegram-bot`](https://github.com/MyThingsLab/my-telegram-bot) | cycle step 10 — the ASK-channel human gate |

**Open wiring question.** [`my-coder`](https://github.com/MyThingsLab/my-coder)
exists specifically to formalize the fleet's worker role (issue → headless
session → draft PR), but `fleet_dispatch.py` doesn't call it — it still spawns
its own inline generic `claude -p` session. Until that's resolved, the worker
logic lives in two places and `my-coder` isn't actually load-bearing for the
cycle, so it stays off the v1 candidate list for now.

## The tools (v0 — build freely)

Each is its own public repo, scaffolded from
[`my-template`](https://github.com/MyThingsLab/my-template) and built on
`my-things-core`.

**Deciding what to do next**

| Tool | Does |
|---|---|
| [`my-orchestrator`](https://github.com/MyThingsLab/my-orchestrator) | Picks the single next unit of work across the whole fleet. |
| [`my-planner`](https://github.com/MyThingsLab/my-planner) | Produces a priority-ordered plan of what the fleet should do next. |
| [`my-architect`](https://github.com/MyThingsLab/my-architect) | Decomposes an objective into an ordered, dependency-tagged backlog — the unattended counterpart to `my-director`. |
| [`my-projector`](https://github.com/MyThingsLab/my-projector) | Keeps the fleet's GitHub Project board and tracking issue in sync. |
| [`my-searcher`](https://github.com/MyThingsLab/my-searcher) | Indexes a repo's files and ranks the ones most relevant to an issue. |
| [`my-idea`](https://github.com/MyThingsLab/my-idea) | Explores a rough idea against the fleet and posts a structured brief. |
| [`my-scaffolder`](https://github.com/MyThingsLab/my-scaffolder) | Bootstraps a brand-new `My[X]` tool repo from a proposal issue. |
| [`my-pipeline`](https://github.com/MyThingsLab/my-pipeline) | Declares and drives cross-tool handoffs (workflow DAG) as labeled issues. |

**Working the code**

| Tool | Does |
|---|---|
| [`my-coder`](https://github.com/MyThingsLab/my-coder) | Formalizes the fleet's worker role: closes a picked issue as a draft PR via a full headless coding session. |
| [`my-tester`](https://github.com/MyThingsLab/my-tester) | Finds one uncovered unit and opens a PR adding a test for it. |
| [`my-changelogger`](https://github.com/MyThingsLab/my-changelogger) | Folds ledger `ship`/`fix`/`build` entries into a `CHANGELOG.md` section. |
| [`my-docs`](https://github.com/MyThingsLab/my-docs) | Keeps a docs site in sync with each tool's README / CLAUDE.md. |
| [`my-todo`](https://github.com/MyThingsLab/my-todo) | Curates a `TODO.md` from live fleet signals. |
| [`my-librarian`](https://github.com/MyThingsLab/my-librarian) | Given a task, discovers existing packages/CLIs live and recommends build-vs-buy. |

**Reporting & comms**

| Tool | Does |
|---|---|
| [`my-reporter`](https://github.com/MyThingsLab/my-reporter) | Digests the ledger into a report and posts it as a comment. |
| [`my-dashboard`](https://github.com/MyThingsLab/my-dashboard) | Renders the fleet's status as a shareable dashboard. |
| [`my-office`](https://github.com/MyThingsLab/my-office) | Renders the fleet as a physical-virtual raytraced office from the shared ledger. |
| [`my-server`](https://github.com/MyThingsLab/my-server) | Surfaces the fleet over HTTP. |
| [`my-telegram-bot`](https://github.com/MyThingsLab/my-telegram-bot) | Pushes ledger notifications over Telegram; relays a Policy `ASK` to a human. |
| [`my-guide`](https://github.com/MyThingsLab/my-guide) | The fleet's front door: a plain-language catalog, wish matching, and dry-run trials. |

**Research & knowledge**

| Tool | Does |
|---|---|
| [`my-uni`](https://github.com/MyThingsLab/my-uni) | Decomposes a "field of study" issue into researchable topic issues. |
| [`my-researcher`](https://github.com/MyThingsLab/my-researcher) | Discovery + synthesis — turns a topic issue into a cited brief PR. |
| [`my-cartographer`](https://github.com/MyThingsLab/my-cartographer) | Clusters a document corpus into named, prerequisite-ordered themes. |
| [`my-glossary`](https://github.com/MyThingsLab/my-glossary) | Defines a term from a document corpus, citing the exact source spans. |
| [`my-bibliography`](https://github.com/MyThingsLab/my-bibliography) | Resolves a DOI/arXiv-id/ISBN/free-text reference into BibTeX + CSL-JSON. |
| [`my-archivist`](https://github.com/MyThingsLab/my-archivist) | Maintains a unified catalog of a book / materials collection. |
| [`my-scraper`](https://github.com/MyThingsLab/my-scraper) | Given a URL and a question, extracts structured data from the page. |
| [`my-notes`](https://github.com/MyThingsLab/my-notes) | Freeform note capture: files a note as an issue, extracts tags + title. |

**Study loop** (spaced recall, quizzing, grading — see `mythings.mastery`)

| Tool | Does |
|---|---|
| [`my-syllabus`](https://github.com/MyThingsLab/my-syllabus) | Decomposes a course program into an ordered list of masterable topics. |
| [`my-professor`](https://github.com/MyThingsLab/my-professor) | Quizzes and grades one topic at a time against a document corpus. |
| [`my-flashcards`](https://github.com/MyThingsLab/my-flashcards) | Spaced-recall drilling: builds, reviews, and grades flashcard decks. |
| [`my-grader`](https://github.com/MyThingsLab/my-grader) | Summative assessment: grades a whole mock exam in one pass. |

**Documents & media**

| Tool | Does |
|---|---|
| [`my-typster`](https://github.com/MyThingsLab/my-typster) | Turns a document-drafting issue into a Typst document. |
| [`my-presentation`](https://github.com/MyThingsLab/my-presentation) | Turns a talk-request issue into a drafted, compiled Typst slide deck. |
| [`my-designer`](https://github.com/MyThingsLab/my-designer) | Design brief → self-contained HTML/CSS mockup. |
| [`my-site`](https://github.com/MyThingsLab/my-site) | Drafts Jekyll content for a personal site. |
| [`my-manim-editor`](https://github.com/MyThingsLab/my-manim-editor) | Translates an animation concept into a syntax-valid ManimCE Scene script. |
| [`my-figure`](https://github.com/MyThingsLab/my-figure) | Extracts every embedded figure from a PDF into a cross-referenced index. |
| [`my-tables`](https://github.com/MyThingsLab/my-tables) | Extracts every table from a PDF into structured (CSV + Markdown) form. |
| [`my-equations`](https://github.com/MyThingsLab/my-equations) | Extracts every displayed equation from a PDF into indexed LaTeX. |

**Data & signals**

| Tool | Does |
|---|---|
| [`my-data-analysist`](https://github.com/MyThingsLab/my-data-analysist) | Profiles a local CSV and narrates insights + a follow-up analysis. |
| [`my-signal-processor`](https://github.com/MyThingsLab/my-signal-processor) | FFT/statistics analysis of a local CSV time-series. |
| [`my-image-processor`](https://github.com/MyThingsLab/my-image-processor) | Extracts dimensions/histogram/EXIF from a local image and interprets them. |
| [`my-embedder`](https://github.com/MyThingsLab/my-embedder) | Open-source embedding backends for `mythings.embed`, with a local server. |

**Not a `My[X]` tool** — the fleet builds these out as real projects, exercising the loop end-to-end:

| Repo | Does |
|---|---|
| [`my-raytracer`](https://github.com/MyThingsLab/my-raytracer) | Monte Carlo path tracer — `my-coder`'s first real build target. |

**Foundation**

| Repo | Does |
|---|---|
| [`my-things-core`](https://github.com/MyThingsLab/my-things-core) | The SDK: the five contracts every tool imports and no tool reimplements. |
| [`my-template`](https://github.com/MyThingsLab/my-template) | Scaffold for a new `My[X]` tool — copy it, replace `template`. |
| [`my-fleet`](https://github.com/MyThingsLab/my-fleet) | Chains the tools into one autonomous cycle (formerly `fleet-dispatch`'s own scripts). |
| [`fleet-dispatch`](https://github.com/MyThingsLab/fleet-dispatch) | The workspace root; its dispatch/ops scripts now live in `my-fleet`. |
| [`typst-templates`](https://github.com/MyThingsLab/typst-templates) | Shared Typst style-anchor templates for `my-typster`/`my-presentation`. |
| [`mythingslab.github.io`](https://github.com/MyThingsLab/mythingslab.github.io) | Technical documentation for the fleet. |

## Status

Current status, last/next steps per tool, open decisions, and safety gaps
live on the fleet's [project board](https://github.com/orgs/MyThingsLab/projects/1)
(org members only). It's manually curated, not auto-generated — the
[org-wide tracking issue](https://github.com/MyThingsLab/fleet-dispatch/issues/1)
carries the full-detail write-up behind each card.

## Design rules

- **Deterministic-first.** An LLM is called only where judgment is
  irreplaceable; everything else is plain code, free to run and free to test.
- **GitHub-native.** Issues, Actions, PRs, and App identity *are* the
  substrate — no multi-forge abstraction, no scheduler beyond `schedule:`
  triggers.
- **PR, never merge.** No tool holds merge authority. A human or a
  permission-scoped GitHub App does.
- **One shared ledger.** An append-only JSONL history every tool writes to
  and reads back, so the fleet has one trustworthy audit trail instead of N
  private ones.

All tool repos are public and MIT-licensed.
