# MyThingsLab

A fleet of small tools that develop a GitHub repository on their own —
calling an LLM only for the one step that genuinely needs judgment, and
running everything else as deterministic code.

This page lists the **kernel**: the repos the autonomous loop actually
imports or invokes. The org contains more — study tooling, document and
media extractors, experiments — but they are not what the loop runs on, so
they are not here. Browse
[all repositories](https://github.com/orgs/MyThingsLab/repositories) if you
want the long tail.

## The idea

Most "AI agent" systems are one big model call wearing a trench coat: the
model is asked to plan, to search, to edit, to verify, and to decide. Every
one of those steps is then non-deterministic, unauditable, and billed.

MyThingsLab inverts that. A tool is mostly ordinary Python, and the model is
a **seam** — one narrow, typed call at the single point where judgment is
irreplaceable. Everything around it is deterministic: reading the backlog,
ranking files, running the suite, opening the PR, writing the outcome down.

The payoff is concrete. A tool's entire skeleton runs and tests for free
against `NoopEngine` before a real model is ever wired in, so the part that
can break silently is small enough to hold in your head, and the part that
costs money is small enough to price.

## The loop

1. **Read** a unit of work from a backlog — a labelled GitHub issue.
2. **Prepare** deterministically — index, rank, lint, check. No model call.
3. **Decide** with exactly one LLM call, for the one step that needs it.
4. **Propose** a pull request. Never a silent merge.
5. **Record** a structured outcome in a shared append-only ledger, so the
   next run can trust what the last one did.

Step 5 is the one people skip and the one that makes the rest work. The
ledger is how a fleet of independent processes gets a shared memory without
a database, and it is what every report, dashboard, and liveness check reads.

## The five contracts

[`my-things-core`](https://github.com/MyThingsLab/my-things-core) is the SDK.
It calls no LLM itself. Every tool imports these and no tool reimplements
them:

| Contract | What it is |
|---|---|
| `mythings.ledger` | Append-only JSONL run history — the fleet's shared memory. |
| `mythings.policy` | `Action` types and an allow / **ask** / deny `Decision`. `ask` is a real state: it suspends the run and routes to a human. |
| `mythings.engine` | The `Engine` protocol — the *one* seam where a model is invoked. `NoopEngine` makes the whole fleet runnable and testable at zero cost. |
| `mythings.github` | A thin `gh`-CLI adapter. GitHub is the substrate, not an abstraction over "some forge". |
| `mythings.isolation` | `Workspace` — a git-worktree sandbox, plus detection of CI, where the runner already *is* the sandbox. |

## The kernel

Two different kinds of wiring, and the difference matters more than the list
does. Some tools are **imported as libraries** — they are on the critical
path and their API breaking breaks the loop. Others are **invoked as CLIs**
by the cycle — they are stages, independently replaceable, and a failure
degrades a cycle rather than stopping it.

### Imported — on the critical path

| Repo | What it does |
|---|---|
| [`my-things-core`](https://github.com/MyThingsLab/my-things-core) | The SDK above. Everything depends on it; it depends on nothing but `git` and `gh`. |
| [`my-fleet`](https://github.com/MyThingsLab/my-fleet) | The orchestration layer: the dispatch loop, the multi-stage cycle, the merge gate, auth preflight, and the ops tooling around them. |
| [`my-orchestrator`](https://github.com/MyThingsLab/my-orchestrator) | Picks the single next unit of work across the whole fleet — one ranked answer, not a list. Imported directly by the dispatcher. |
| [`my-coder`](https://github.com/MyThingsLab/my-coder) | The worker. Takes one issue and closes it as a PR via a bounded headless coding session in a throwaway worktree. |
| [`my-searcher`](https://github.com/MyThingsLab/my-searcher) | Indexes a repo and ranks the files most relevant to an issue. Imported by `my-coder` to aim the session before it starts spending tokens. |
| [`my-guard`](https://github.com/MyThingsLab/my-guard) | The rule engine that turns an `Action` into allow / ask / deny — the thing standing between an autonomous run and a side effect. |
| [`my-telegram-bot`](https://github.com/MyThingsLab/my-telegram-bot) | The human channel. Relays a Policy `ask` to a phone and carries the answer back, so a human gate does not mean a human at a desk. |
| [`my-template`](https://github.com/MyThingsLab/my-template) | The scaffold every tool is cut from — which also means a fix here fans out to the whole fleet. |

**`my-coder` is the fleet's one deliberate exception** to the single-call
rule: its core action is an open-ended, tools-enabled session rather than one
narrow call. That is the point — it is the tool that *writes* the other
tools, and the exception is scoped to exactly one repo instead of leaking
into all of them.

### Invoked — the cycle stages

Chained by `my-fleet`'s cycle, each a standalone CLI you can run by hand.

| Repo | Stage |
|---|---|
| [`my-director`](https://github.com/MyThingsLab/my-director) | Human-in-the-loop: an end-of-day session that sets the next objective and decomposes it into task-issues. |
| [`my-planner`](https://github.com/MyThingsLab/my-planner) | Ranks the backlog into a priority-ordered build plan, grounded in dependencies and ledger velocity. |
| [`my-researcher`](https://github.com/MyThingsLab/my-researcher) | Turns an open topic into a cited brief from live sources (web + arXiv). |
| [`my-tester`](https://github.com/MyThingsLab/my-tester) | Finds one uncovered unit and opens a PR adding a test for it. The most-run tool in the fleet. |
| [`my-changelogger`](https://github.com/MyThingsLab/my-changelogger) | Folds ledger `ship`/`fix`/`build` entries into a `CHANGELOG.md` section. |
| [`my-projector`](https://github.com/MyThingsLab/my-projector) | Syncs the GitHub Project board and org tracking issue to live repo state. |
| [`my-pipeline`](https://github.com/MyThingsLab/my-pipeline) | Drives cross-tool handoffs — a workflow DAG expressed as labelled issues. |
| [`my-reporter`](https://github.com/MyThingsLab/my-reporter) | Digests the ledger into a report and posts it as a comment. |
| [`my-dashboard`](https://github.com/MyThingsLab/my-dashboard) | Renders org-wide fleet status and single-repo status cards. |

### Proving ground

| Repo | Why it exists |
|---|---|
| [`my-raytracer`](https://github.com/MyThingsLab/my-raytracer) | A real Monte Carlo path tracer, built by the fleet rather than about it. Non-trivial numerics with an exact regression oracle — which makes it the one target where "the agent shipped something" is objectively checkable. |

## Merging is gated, not forbidden

The fleet used to ban autonomous merges outright. That was the wrong rule: it
was really guarding against an *unverified* merge, and banning all of them
just moved the bottleneck onto a human.

A run may now merge a PR only when four things hold, each established by
mechanism rather than by the merging agent's own opinion:

- required checks report **`pass`** — not `skipped`, not `none`, not absent;
- the PR is not a draft;
- the repo's `main` is branch-protected, so "required" means something;
- the diff stays inside the scope of the issue it closes.

Short of all four, it stays open for a human. Four classes of change never
ride the gate however green they are: the gate's own code, the constraints on
agents, credential handling, and public API or schema migrations — because
merging a bad one of those destroys the ability to catch the next one.

## Stability

Most of the fleet is **v0**: build freely, float on `@main`, no version
discipline. A core has graduated to **v1** — real semver, a `CHANGELOG.md`
entry per bump, deprecate-before-remove, and v1-to-v1 dependencies pinned to
a tagged release. Currently v1: `my-things-core`, `my-fleet`, `my-guard`,
`my-director`, `my-dashboard`, `my-reporter`. The contract is
[`release.md`](https://github.com/MyThingsLab/my-things-core/blob/main/src/mythings/release.md).

## Design rules

- **Deterministic-first.** A model is called only where judgment is
  irreplaceable. Everything else is plain code — free to run, free to test.
- **GitHub-native.** Issues, Actions, PRs, and App identity *are* the
  substrate. No multi-forge abstraction, no scheduler beyond `schedule:`.
- **Evidence, not confidence.** Every gate asks what *reported* success, not
  who believes it. A skipped check is not a passing check; a green suite that
  tested the installed package instead of the diff proved nothing.
- **One shared ledger.** Append-only, written by every tool, read by every
  report — one trustworthy audit trail instead of N private ones.

## Status

Live status, open decisions, and safety gaps are on the fleet's
[project board](https://github.com/orgs/MyThingsLab/projects/1) (org members
only). The honest public record is the periodic full-ecosystem review —
including, at length, what is *not* working — versioned in
[`my-fleet/workspace/reviews`](https://github.com/MyThingsLab/my-fleet/tree/main/workspace/reviews).

All repos are public and MIT-licensed.
