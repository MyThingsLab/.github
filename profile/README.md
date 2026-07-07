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

[`mythings-core`](https://github.com/MyThingsLab/mythings-core) is the SDK
this is built on: five contracts (`ledger`, `policy`, `engine`, `github`,
`isolation`) that every tool imports and no tool reimplements. It calls no
LLM itself — `engine` is the single seam where a model is ever invoked, and
it's a `Protocol`, so a tool runs and tests its whole skeleton for free
before any real backend is wired in.

## The tools

| Tool | Does |
|---|---|
| [`my-guard`](https://github.com/MyThingsLab/my-guard) | Rule engine — evaluates an `Action` to allow / ask / deny. |
| [`my-orchestrator`](https://github.com/MyThingsLab/my-orchestrator) | Picks the single next unit of work across the whole fleet. |
| [`my-tester`](https://github.com/MyThingsLab/my-tester) | Finds one uncovered unit and opens a PR adding a test for it. |
| [`my-changelogger`](https://github.com/MyThingsLab/my-changelogger) | Turns ledger ship/fix/build entries into a `CHANGELOG.md` section and opens a PR. |
| [`my-reporter`](https://github.com/MyThingsLab/my-reporter) | Digests the ledger into a report and posts it as a comment. |
| [`my-telegram-bot`](https://github.com/MyThingsLab/my-telegram-bot) | Pushes ledger notifications over Telegram; relays a Policy `ASK` to a human. |
| [`my-template`](https://github.com/MyThingsLab/my-template) | Scaffold for a new `My[X]` tool — copy it, replace `template`. |
| [`fleet-dispatch`](https://github.com/MyThingsLab/fleet-dispatch) | Workspace-level dispatcher that drives the fleet across repos. |

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

All repos are public and MIT-licensed.
