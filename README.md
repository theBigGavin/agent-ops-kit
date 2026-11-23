# agent-ops-kit
给你的 agent 一套接活干活的骨架：看板编排+巡检+红线+验收。

**agent-ops-kit is a task intake skeleton.** It is not a framework, not a
runtime, not a model wrapper. It is the folder layout, the four config files and
the two written rules an agent needs in order to pick up work, keep it moving
without being prodded, and refuse to ship something it cannot verify.

Most agent setups fail in the same boring place: the agent can do a task, but
there is no answer to "what should it be doing right now", "who noticed it got
stuck", and "on whose authority is this finished". This kit answers those three
questions with files you can read, edit and commit.

## Status

Open-source private beta, free during beta; a hosted version is in the works.
Everything here is plain files in your repository — no service to sign up for,
no account, no telemetry. Clone it, run the initializer, edit what it drops in,
and it works the same with or without anything hosted.

## Quickstart

```bash
git clone <this-repo-url> agent-ops-kit
cd agent-ops-kit
bash bin/agent-ops init
```

That copies the templates into the current directory and prints what it did.
Nothing is overwritten: run it a second time and every file reports `skipped`,
so it is safe to re-run after pulling, or to point at a project that already has
some of these files.

Target another directory instead:

```bash
bash bin/agent-ops init ../my-project
```

The initializer is idempotent and prints one line per entry — `created` or
`skipped` — then the next steps to take. `bash bin/agent-ops help` shows the
usage.

## What you get

```
agent-ops-kit/
├── bin/
│   └── agent-ops              # the initializer: init, help
├── template/
│   ├── board.example.json     # the board: columns, limits, task cards
│   ├── patrol.example.json    # the sweep schedule: when, what, what it may write
│   ├── heartbeat.example.json # how often the agent reports in, and what a stall means
│   ├── test-redline.md        # the checks a task may never ship past
│   ├── acceptance-gate.md     # what has to be true before a task is done
│   └── agent.example.yaml     # scope, limits and authority of the agent
├── LICENSE
└── README.md
```

After `init`, the project side looks like this:

```
ops/
├── board.json           # from board.example.json
├── patrol.json          # from patrol.example.json
├── heartbeat.json       # from heartbeat.example.json
├── agent.yaml           # from agent.example.yaml
├── test-redline.md      # copied verbatim — edit it, do not delete it
├── acceptance-gate.md   # copied verbatim — same
├── tasks/               # one file per task, moved between columns
└── logs/                # board, patrol, heartbeat and per-task logs
```

Commit `ops/*.json`, `ops/*.yaml`, `ops/*.md` and `ops/tasks/`. Leave
`ops/logs/` out of git — `.gitignore` already does that for you.

## The four pieces

**Board** (`ops/board.json`) — columns, work-in-progress limits and the task
cards. Every card names its acceptance clause, its log file and the owner. The
`doing` column holds one task at a time; that single constraint removes most of
the chaos you would otherwise have to babysit.

**Patrol** (`ops/patrol.json`) — the schedule. A sweep says what it reads, what
it is allowed to write, and what to do when it is blocked. Point any scheduler
at it: cron, systemd timers, a GitHub Action, whatever your environment already
runs. Nothing here invents a runner of its own.

**Redline** (`ops/test-redline.md`) — the floor. Checks a task cannot pass to
`done` without: build clean, tests actually run, tests can still fail, no
secrets or machine-local paths in the diff, log readable, rollback written down.
A check you could not run counts as a failure, not as a pass.

**Gate** (`ops/acceptance-gate.md`) — the doorway. The redline is necessary and
not sufficient; the gate is the per-task checklist plus the named closings
(`sprint-close`, `sweep-close`, `triage-close`) that say who is allowed to call
a piece of work finished.

The **heartbeat** (`ops/heartbeat.json`) is the one that tells you the difference
between an agent working slowly and an agent that stopped: it defines the report
cadence, and it defines silence as a finding rather than as progress.

## How a task moves

```
backlog → intake → doing → review → done
                       ↑________|   (gate or redline rejects it)
```

1. Patrol promotes the highest-priority item from `intake` into `doing`.
2. The agent works it, writing progress to the task log, reporting on the
   heartbeat cadence.
3. The evening sweep runs the redline. Anything red stays put.
4. The gate is walked. By default a human closes it; `ops/agent.yaml` is where
   you change that, in writing, so the next reader knows who signed off.

Rejected twice, a task is either two tasks or one task with a wrong acceptance
clause. Fix the task, not the record.

## Design rules

- **Files over services.** Every piece of state is a file you can `cat`.
- **The agent never edits the rules it is graded against.** `test-redline.md`,
  `acceptance-gate.md` and `agent.yaml` are in `never_write`.
- **Silence is a finding.** A missing heartbeat is a bug report, not a quiet run.
- **Templates are the shared artefact.** `template/*.example.*` ships here; your
  live `ops/*` is yours and `.gitignore` keeps the local variants local.
- **Small diffs.** A task whose diff a human cannot read in one sitting is not
  finished — it is two tasks.

## License

MIT — see [LICENSE](LICENSE).
