# Coordinator rules

Rules for the manager seat (the session that plans, dispatches and merges). Put this file
in your tool's project instructions. Each rule carries a one-line *why*; numbers come from
one solo project's own session logs and are examples, not guarantees.

A **seat** is one agent context: the main session, or one subagent / sub-session.
A **fork** of a seat counts as that same seat.

---

## 1. Cross-review

- **The writer never reviews its own work.**
  *Why:* it checks the output against what it meant to write, not against what is there.
- **The dispatcher does not review the work it handed out either.** Review goes to a seat
  that neither wrote nor dispatched it.
  *Why:* the dispatcher already has a picture of the "right" answer and grades against it.
- **The reviewer gets the code and the diff, not the writer's report or conclusions.**
  Start it in a fresh context. Any summary you pass along is a lead to verify, not a fact.
  *Why:* a report tells the reviewer where not to look.
- **A fork counts as the same seat**, even if it runs on a different model.
  *Why:* it carries the writer's whole context, assumptions included.
- **At least two review rounds** over the whole result before calling work done. If round 2
  still finds a lot, run a third. For a small, local, low-risk change, round 2 may check
  only round 1's fixes and the code around them.
  *Why:* fixes introduce new bugs; one round only catches the first layer.
- **A fix made after the last round gets one check of just that fix**, by a seat that did
  not write the fix. That check is not a new round.
  *Why:* an unchecked last-minute fix is the easiest place for a regression to land.
- **Optional, recommended: cross-vendor whole-project review.** After each round of work
  merges to main, a model from a different vendor reviews the whole project. Review only,
  never writes; findings go back to the manager, who assigns fixes.
  *Why:* on one feature, two full review rounds on one vendor's models both missed the
  boundary checks on client input; a model from another vendor then found 7 issues.
- **Trust-boundary lens.** Any change touching a trust boundary (client commands, network
  messages, saved data, admin commands, file paths, shell arguments) gets one extra review
  lens: *assume the client will send anything.* Every finding must include a concrete,
  reproducible input.
  *Why:* this is the class of bug that same-vendor review missed most often.

## 2. Pick the model tier by the job

Start at the cheapest tier that can do the job reliably. If it gets stuck **and you can
say why**, escalate one tier, once. Never retry repeatedly on the same tier.
*Why:* one day with every seat on the strongest model at max effort ate 71% of that
week's spend.

| Tier | Typical jobs | Claude example |
|---|---|---|
| cheap | keyword / file searches, collecting logs, format conversion, simple data edits | Haiku |
| mid | implementing a well-specified brief, running tests, bounded verification, tester seat | Sonnet |
| strong | review rounds, hard implementation, cross-module changes, coordinating several workers on one feature, hard debugging | Opus |
| top | the hardest design calls and final integration review, used sparingly | the strongest model or effort level your plan offers |

Other vendors offer comparable small / mid / large tiers; match them to this table yourself. Review rounds
start at strong: a reviewer that misses things costs more than it saves.

- **Set the model explicitly on every dispatch.** Do not let a subagent inherit the main
  session's model. *Why:* an expensive manager silently makes every child expensive.

## 3. Subagent cache: never idle more than 4 minutes

Prompt caches have a short time-to-live, and it is provider-specific. In our Claude Code logs
(subscription plan, at the time of writing) subagent caches expired after **~5 minutes**
and the main session's after **~1 hour**. This can change between versions and plans;
check what your provider does before relying on these numbers.

- **A subagent never waits more than ~4 minutes at a time.** Long commands run in the
  background (your tool's background mode, `nohup`, `tmux`, or a small job-runner script),
  and the subagent polls every ~240 seconds.
  *Why:* after the cache expires, the next request re-pays the whole context at full
  price: roughly 12-20x the input cost of a cached request (about 15x in our logs).
- **Jobs expected to run over an hour are started and waited on by the main session**, not
  by a subagent. This includes time spent waiting in a queue.
  *Why:* one worker waiting in a job queue was woken 7 times, each a full rewrite: 1.56M
  tokens that produced no code.
- **Do not resume a long-idle subagent for a small question.** Look it up yourself or use
  a fresh short seat. Resume it only when its context is still worth having (for example,
  fixing its own work).
  *Why:* the first request after a long idle is a full rewrite.

## 4. Lean agent definitions

- **Give each seat only the tools it needs.** Workers: read, edit, write, shell, search.
  Reviewers and testers: read, search, shell, and write for notes only. Use a
  general-purpose agent only for jobs that need a browser, the web, or further delegation.
  *Why:* in our setup (many MCP tools and skills), a general-purpose subagent carried
  ~75k tokens of tool and skill descriptions on every request, versus ~25k for a lean one,
  and a typical subagent run took a median of 37 turns.
- **Do not let subagents call an expensive advisor / consult model.** If your tool has
  one, put "do not call the advisor" in every brief unless you allow it for that seat.
  *Why:* in Claude Code we saw subagents inherit the main session's advisor, and we found
  no per-subagent off switch; each consult re-reads the whole conversation at the
  advisor's price. (Seen at the time of writing; check your version.)

## 5. Check artifacts, not reports

- **"Done" is only the agent's own word.** Before releasing the next step, verify: every
  file named in the report exists and is non-empty; the commit hash is on the branch
  (`git merge-base --is-ancestor <hash> <branch>`); the check output is real.
  *Why:* a review once started on half-finished output because the previous step said done.
- **Incomplete work goes back to the same agent**, not to a new one.
  *Why:* a new agent re-reads everything from scratch, costs more and loses context.

## 6. Bounce-backs carry exactly four things

When sending work back: **which item**, **why**, **the evidence** (for example the last lines
of the failing check), and **the scope** (what to change and what not to touch).
If the same item bounces 3 times, stop and check whether the task was cut wrong.
*Why:* vague bounce-backs cause rewrites of things that were fine; repeat bounces usually
mean the brief, not the worker, is the problem.

## 7. Collision check before dispatch

- **Work touching the same part of the same file waits in a queue**, or the shared piece
  lands first on its own. For two branches that already have commits,
  `git merge-tree --write-tree <A> <B>` (Git 2.38+) shows conflicts without touching either branch.
  *Why:* several of the most expensive items in our cost log were two parallel work streams
  editing the same code; merging them hit 6, 10 and 7 conflicts, all resolved by hand.
- **Always branch from the latest main.** Parallel workers each get their own branch and
  their own `git worktree`, created by the manager; never two workers in one working tree.
  *Why:* one branch cut from a two-day-old main cost an extra 330k tokens to merge.
- **Split implementation by code area, not by item count.**
  *Why:* splitting by count puts two workers in the same function.
- **Run a search seat first.** A cheap seat pins files, functions and rough line numbers
  into the brief before the expensive seat starts.
  *Why:* the expensive model should not dig through the code from scratch. A model-free
  code index (ctags, an LSP, or a small script) that finds definitions and call sites in
  about a second helps further.

## 8. Briefs

- A brief must be self-contained: someone who reads only the brief, and nothing else in
  the repo, can finish the job and knows when it is done. Use `templates/task-brief.md`.
- Name the exact scope ("only these items"), the acceptance check, and an expected time
  to deliver.
- Tell seats to build first and collect open questions into one list at the end, instead
  of stopping mid-task to ask.
- Pin interfaces as tightly as you can. Anything left open, two workers will do differently.

## 9. Smaller rules

- **Timed jobs run alone.** Timed jobs (benchmarks, frame-time or latency measurements) get the
  machine to themselves; untimed jobs (tests, screenshot checks) run at most ~4 in parallel.
  *Why:* parallel load makes measured numbers meaningless.
- **A separate tester seat** handles what scripts cannot check: walking through a real
  page, judging whether visuals look right. It only tests, never edits; problems go back
  to the writer. Its result is verification, not a review round.
  *Why:* writers who cannot script a check tend to write "should be fine" instead.
- **Only the manager merges.** Other seats commit to their own branch (and push it, if the repo has a
  remote) and report.
  *Why:* one merge point means one place where integration is checked.
- **No routine status messages.** Seats speak up only to get a decision, when stuck, or
  when delivering. Progress goes into a notes file, not into messages.
  *Why:* every message wakes the manager, the most expensive context, to re-read everything.
- **Check on overdue work.** If a seat is past twice its expected time, or silent for
  ~15 minutes, look at its branch and job output; ask it to hand in what it has, or stop it
  and reassign.
  *Why:* a stuck seat that nobody looks at blocks a whole line of work.
- **Keep output short.** Pipe build, test and log output through `tail` / `grep`. Search
  for line numbers before reading large files; read only the needed range.
  *Why:* every line of tool output is re-read on every later request.
