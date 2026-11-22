---
name: multi-agent-workflow
description: Sets up and runs a multi-agent development workflow. A manager plans and dispatches, a search seat pins files, mid-tier workers implement, two fresh reviewers cross-review, a tester checks what scripts cannot, and the manager merges. Use it when the user asks to set up this workflow or kit. In a repo that has .claude/workflow/pipeline.md, also use it for every build, fix or change request.
---

# Multi-agent workflow

This skill has two parts:

- **Setup** runs once per repo. It installs the rules, the pipeline and the agents.
- **Run** applies to every request after that. The user only says what they want, and you
  act as the manager for the whole pipeline.

Setup reads its source files (`rules/`, `agents/`, `templates/`) from this skill's own
folder, next to this file (usually `~/.claude/skills/multi-agent-workflow/`). Run uses the copies that Setup put in the user's repo under
`.claude/workflow/`.

## Setup (once per repo)

1. **Look first.** Check that the folder is a git repo. If it is not, ask before going on.
   Find the project instructions file: `CLAUDE.md` at the repo root, or `.claude/CLAUDE.md`
   if that is the one in use. If neither exists, plan a new root `CLAUDE.md` that holds only
   the lines from step 4. Look at what is in `.claude/agents/`, `~/.claude/agents/` and
   `.claude/workflow/`.
2. **Show the plan and wait.** List every file you will create, the exact lines you will
   add to the instructions file, and the `model:` line each agent gets. Never overwrite a
   file the user already has. If an agent file, or an agent whose `name:` is `worker`,
   `reviewer` or `tester`, already exists, show both versions and ask which to keep. Go on only after the user says yes.
3. **Rules and pipeline.** Copy `rules/coordinator-rules.md` and `rules/pipeline.md` to
   `.claude/workflow/`.
4. **Hook them into the instructions file.** Add the lines below at the end, unless they are
   already there. The two `@` lines are relative to the instructions file: from a root
   `CLAUDE.md` they look like this; from `.claude/CLAUDE.md` drop the leading `.claude/`.
   The third line is plain text and always names the path from the repo root.
   ```
   @.claude/workflow/coordinator-rules.md
   @.claude/workflow/pipeline.md
   For every build, fix or change request, act as the manager and follow .claude/workflow/pipeline.md.
   ```
   Leave everything else in the file as it is.
5. **Agents.** Copy `agents/worker.md`, `agents/reviewer.md` and `agents/tester.md` to
   `.claude/agents/`. In each copy, add a `model:` line to the frontmatter: `sonnet` for
   worker and tester, `opus` for reviewer. If the user names other models, use theirs.
6. **Templates.** Copy the three files in `templates/` to `.claude/workflow/templates/`.
7. **Report.** Show the lines you added to the instructions file and list the new files.
   Do not commit unless the user asks. Mention that worktrees only contain committed files,
   so the `.claude/` changes and the instructions file should be committed before workers
   run in worktrees.
8. **Tell the user two things.** First, restart Claude Code once (or open `/agents`) so the
   new agents load. Second, from then on they only need to say what they want built.

## Run (every request after setup)

Follow `.claude/workflow/pipeline.md` step by step, and `.claude/workflow/coordinator-rules.md`
for the reasons behind each step. Fill in the templates in `.claude/workflow/templates/`
before sending them; never send a template with placeholders left in it.

## Other tools

Codex CLI, Cursor, Gemini CLI and others can follow this file as plain instructions. Put
`rules/coordinator-rules.md` and `rules/pipeline.md` where that tool reads project
instructions (`AGENTS.md`, `.cursor/rules/`, `GEMINI.md`). Use the bodies of `agents/*.md`
as role prompts, and run each seat as its own session. See `README.md`.
