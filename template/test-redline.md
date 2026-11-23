# Test Redline

The line a task may not cross on its way to `done`. Patrol reads this file during
the evening sweep. A redline is not a preference — failing one means the task
stays where it is and the reason goes to the log.

## How to use it

1. Pick the tasks sitting in `review`.
2. Run every check below, in order, and write the result of each one to the task log.
3. If any check fails, stop. Do not move the task. Do not "partially" ship it.
4. A check you could not run counts as a failure, not as a pass.

## The checks

### R1 — Build is clean

The project builds from a fresh checkout with no manual steps in between.

    # from the project root
    bash -c 'set -e; ./build.sh'

Redline: a non-zero exit, or a build that only works because of a file you
left in a scratch directory.

### R2 — Tests run, and they are not empty

    bash -c 'set -e; ./test.sh'

Redline: zero tests collected is a failure. A suite that passes because
nothing was asserted is a failure.

### R3 — The tests can still fail

Break the thing under test on purpose, run the suite, put it back.

Redline: a suite that stays green while the code is broken is not a suite.

### R4 — No secrets, no local paths

    grep -rn --exclude-dir=.git -e 'api_key' -e 'token' -e 'password' .
    grep -rn --exclude-dir=.git -e 'ssh://' -e 'file:///' .

Redline: a credential committed to the repo, or a path that only resolves on
one machine.

### R5 — The task log is readable

Someone who has not seen the task should be able to read the log and say what
changed and why.

Redline: a `done` with no log is a `review` wearing a costume.

### R6 — Rollback is possible

Write down, in one sentence, how to undo the change.

Redline: no rollback sentence means no `done`.

## When a redline is wrong for your project

Replace it. Do not delete it and leave the gap. Write the replacement in the
same shape — a check, a command, and what failing it means — so the sweep can
still read the file without guessing.
