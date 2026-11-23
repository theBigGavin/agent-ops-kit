# Acceptance Gate

The redline says what may not ship. This file says what has to be true before a
task is called finished. Redline is the floor; the gate is the doorway.

## Per-task gate

Before any task moves from `review` to `done`:

- [ ] Every acceptance clause written on the task card is satisfied, one by one.
- [ ] All redline checks (see `ops/test-redline.md`) pass, or a written
      exception exists naming who accepted the risk.
- [ ] The task log records what changed, what was checked, and what was skipped.
- [ ] The rollback sentence exists.
- [ ] The diff is small enough that a human could read it in one sitting. If it
      is not, split the task and move the halves separately.

## Named gates

### `sprint-close`

The end of a work cycle.

- [ ] No task has been sitting in `doing` longer than two heartbeat windows.
- [ ] Every task claimed `done` this cycle passed its per-task gate.
- [ ] The patrol log for the cycle is read in full, not skipped.
- [ ] Anything dropped from the board has a reason in the log.

### `sweep-close`

One patrol sweep finishing without incident.

- [ ] The sweep ran inside its own window and touched only the paths it is
      allowed to write.
- [ ] Each task it looked at got a log line.
- [ ] Nothing moved twice in one sweep.
- [ ] Failures are counted, and the count is below the ceiling in
      `ops/patrol.json`.

### `triage-close`

A backlog pass.

- [ ] Every item was promoted, merged, or dropped — no item left untouched.
- [ ] Each drop names its reason.
- [ ] The intake column is under its limit.

## Who closes the gate

The board says so in `ops/agent.yaml`. Default: a sweep may open the gate and
walk through the checklist, but only a human closes it. If you let the agent
close it yourself, write that down in `ops/agent.yaml` so the next reader knows
who signed off.

## When the gate will not open

That is the gate working. Move the task back to `intake`, write the reason to
the log, and let the next sweep pick it up. A task that has been rejected by the
gate twice is not a task any more — it is two tasks, or it is one task with a
wrong acceptance clause. Fix whichever it is.
