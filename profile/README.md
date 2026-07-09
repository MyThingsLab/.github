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

## The tools

Each is its own public repo, scaffolded from
[`my-template`](https://github.com/MyThingsLab/my-template) and built on
`my-things-core`.

**Deciding what to do next**

| Tool | Does |
|---|---|
| [`my-orchestrator`](https://github.com/MyThingsLab/my-orchestrator) | Picks the single next unit of work across the whole fleet. |
| [`my-planner`](https://github.com/MyThingsLab/my-planner) | Produces a priority-ordered plan of what the fleet should do next. |
| [`my-projector`](https://github.com/MyThingsLab/my-projector) | Keeps the fleet's GitHub Project board and tracking issue in sync. |
| [`my-searcher`](https://github.com/MyThingsLab/my-searcher) | Indexes a repo's files and ranks the ones most relevant to an issue. |
| [`my-guard`](https://github.com/MyThingsLab/my-guard) | Rule engine — evaluates an `Action` to allow / ask / deny. |

**Working the code**

| Tool | Does |
|---|---|
| [`my-tester`](https://github.com/MyThingsLab/my-tester) | Finds one uncovered unit and opens a PR adding a test for it. |
| [`my-changelogger`](https://github.com/MyThingsLab/my-changelogger) | Folds ledger `ship`/`fix`/`build` entries into a `CHANGELOG.md` section. |
| [`my-docs`](https://github.com/MyThingsLab/my-docs) | Keeps a docs site in sync with each tool's README / CLAUDE.md. |
| [`my-todo`](https://github.com/MyThingsLab/my-todo) | Curates a `TODO.md` from live fleet signals. |

**Reporting & comms**

| Tool | Does |
|---|---|
| [`my-reporter`](https://github.com/MyThingsLab/my-reporter) | Digests the ledger into a report and posts it as a comment. |
| [`my-dashboard`](https://github.com/MyThingsLab/my-dashboard) | Renders the fleet's status as a shareable dashboard. |
| [`my-server`](https://github.com/MyThingsLab/my-server) | Surfaces the fleet over HTTP. |
| [`my-telegram-bot`](https://github.com/MyThingsLab/my-telegram-bot) | Pushes ledger notifications over Telegram; relays a Policy `ASK` to a human. |

**Research & knowledge**

| Tool | Does |
|---|---|
| [`my-uni`](https://github.com/MyThingsLab/my-uni) | Decomposes a "field of study" issue into researchable topic issues. |
| [`my-researcher`](https://github.com/MyThingsLab/my-researcher) | Discovery + synthesis — turns a topic issue into a cited brief PR. |
| [`my-archivist`](https://github.com/MyThingsLab/my-archivist) | Maintains a unified catalog of a book / materials collection. |
| [`my-librarian`](https://github.com/MyThingsLab/my-librarian) | Given a task, discovers the right tool and format to produce the output. |
| [`my-scraper`](https://github.com/MyThingsLab/my-scraper) | Given a URL and a question, extracts structured data from the page. |

**Documents & media**

| Tool | Does |
|---|---|
| [`my-typster`](https://github.com/MyThingsLab/my-typster) | Turns a document-drafting issue into a Typst document. |
| [`my-presentation`](https://github.com/MyThingsLab/my-presentation) | Turns a talk-request issue into a drafted presentation. |
| [`my-site`](https://github.com/MyThingsLab/my-site) | Drafts Jekyll content for a site. |
| [`my-raytracer`](https://github.com/MyThingsLab/my-raytracer) | A Monte Carlo path tracer — a plain-Python project the fleet builds out. |

**Foundation**

| Repo | Does |
|---|---|
| [`my-things-core`](https://github.com/MyThingsLab/my-things-core) | The SDK: the five contracts every tool imports and no tool reimplements. |
| [`my-template`](https://github.com/MyThingsLab/my-template) | Scaffold for a new `My[X]` tool — copy it, replace `template`. |
| [`fleet-dispatch`](https://github.com/MyThingsLab/fleet-dispatch) | Workspace-level dispatcher that chains the tools into one autonomous cycle. |

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
