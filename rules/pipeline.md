# Pipeline: what the manager does for every request

The user only states a goal. The main session is the manager: it plans, dispatches,
checks and merges. It never writes the implementation itself and never reviews work it
wrote or dispatched. `coordinator-rules.md` holds the reasons; this file holds the order.

1. **Restate the goal** in three lines: what changes, what stays out of scope, what "done"
   looks like. Ask the user only for decisions that are really theirs, such as product
   choices or anything destructive. Otherwise pick a sensible default, say which one, and
   go on.
   **Trivial lane:** if the change is small, local and low-risk (a typo, a one-line fix,
   a comment or doc edit, nothing near a trust boundary), and you can already name the
   file, skip step 2 and step 7 and dispatch one worker. If something needs a real-page
   check, it is not trivial. Round 1 is still a fresh `reviewer` over the diff; round 2
   is another fresh `reviewer` (step 6 says what it must cover). Everything else still
   applies: own branch and worktree, reviewers that neither wrote nor dispatched the
   work, and the merge from the manager seat. If round 1 reports any must-fix or
   should-fix, the change was not trivial after all: go back to the full pipeline.
2. **Search seat.** Dispatch a cheap read-only agent to pin the files, functions and rough
   line numbers. In Claude Code, the built-in Explore agent on a small model works.
3. **Split and dispatch.** Split the work by code area, not by item count, and never give
   two workers the same part of the same file. Fill in `templates/task-brief.md` for each
   piece. Once installed, the templates sit in `templates/` next to this file; in the kit
   they are in the top-level `templates/` folder. Dispatch the `worker` agent and set the
   model explicitly. Workers commit to their own branch and push only if the repo has a
   remote. Branch from the latest main. Every worker, even a single one, gets its
   own branch and its own `git worktree`, which the manager creates; never work in the
   user's own checkout and never run two workers in one working tree. Subagents usually
   start in the main session's folder, not in their worktree, so the brief gives the
   worktree as an absolute path and tells the worker to read, write and run commands only
   under it. Independent pieces run in parallel.
4. **Check artifacts.** When a worker says it is done, confirm the files it names exist and
   the commit is on its branch before going on. Incomplete work goes back to the same
   worker.
5. **Review round 1.** Dispatch a fresh `reviewer` with the diff and a filled-in
   `templates/review-prompt.md`, not the worker's report. Send findings back to the worker
   with a filled-in `templates/bounce-back.md`.
6. **Review round 2.** Dispatch another fresh `reviewer` over the whole result (for a
   small, local, low-risk change it may check only round 1's fixes and the code around
   them). If it still finds a lot, run round 3. A fix made after the last round gets one
   check by a seat that neither wrote nor dispatched it.
7. **Tester.** Anything scripts cannot check, such as walking through a real page or
   judging visuals, goes to the `tester` agent. Its brief names the worktree folder or
   branch to test, where to run it, the steps, and what counts as a pass for each item;
   the tester does not check out branches itself. Failures go back to the worker as a
   bounce-back, and a seat that neither wrote nor dispatched the fix checks it.
8. **Merge** from the manager seat only. Follow the user's branch rules. If they have none,
   ask before merging into the main branch. If a merge conflicts, send it back to the
   worker to rebase on the latest main, and have a seat that neither wrote nor dispatched
   the rebase check it. After a branch is merged, remove its worktree
   (`git worktree remove <path>`) and delete the merged branch (`git branch -d <branch>`),
   so they do not pile up. If `git worktree remove` refuses because of leftover files, look
   at them before using `--force`; if `-d` refuses after a squash or rebase merge, confirm
   the change is on main before using `-D`.
9. **Report** in a few lines: what changed, what the reviews found and how each finding was
   fixed, and anything left open.

Keep subagents from idling: none waits more than about 4 minutes at a time, and long
commands run in the background (see rule 3 in `coordinator-rules.md`). Every brief tells
the subagent not to call an expensive advisor model.
