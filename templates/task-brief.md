# Task brief

<!-- Replace every <...>. Self-check before sending: could someone who reads only this
     page, and nothing else in the repo, finish the job and know when it is done? -->

**Role:** worker. <If your tool has an advisor / consult model: do not call it.>
**Where:** branch `<branch>`, cut from `<latest main commit>`, working folder `<path>`.
**Expected time to deliver:** `<minutes>`. Past twice that, hand in what you have.

## Goal

<One or two sentences: what to build or fix, and the real problem it solves.>

## Scope

- Only these items: <list>.
- Only touch these files / folders: <list>.
- Do not touch: <files, interfaces, behavior that must stay as is>.

## Pinned locations (from the search seat)

- `<file>:<line>` `<function>`: <why it matters>
- `<file>:<line>` `<function>`: <...>

## Interface (pin it down; anything left open will be done differently by each worker)

- <function / command / message shape, inputs, outputs, error cases, exit codes>

## Rules

- Reuse existing code before writing new code. Add nothing not asked for.
- Every command gets a timeout. Never wait more than ~4 minutes at a time; run long
  commands in the background and poll every ~240 s. Over an hour: hand the command back.
- Same error twice: stop and report. Same check failing after three attempts: stop.
- Build first; collect any open questions into one list at the end.
- <Commit and push to your own branch> / <Do not commit; the manager will>.

## Done when

- [ ] <acceptance check command> passes; paste its last lines.
- [ ] <test case 1: input -> expected result>
- [ ] <test case 2 / error case / edge value>

## Report format

- What I did (one line per file):
- Commit / branch, or changed files:
- Check output (last lines):
- Not done / not verified:
- For the tester (what scripts cannot check, steps, what counts as a pass):
- Out of scope (noticed, not fixed):
- Open questions:
