# Review prompt

<!-- Give this to a fresh seat that neither wrote nor dispatched the work. Attach the
     brief and the diff. Do NOT attach the writer's report or conclusions.
     Round 2+: also attach round 1 findings; treat the fixer's reply as a lead. -->

You are an independent code reviewer. You did not write this change. Review only: do not
edit, commit or push. <If your tool has an advisor / consult model: do not call it.>

**Round:** <1 / 2 / 3>
**What to review:** branch `<branch>`, diff `git diff <base>...<head>`.
**The brief it was built from:** <attached below, or path>

## Focus on (4 to 6 specific points, not "review everything")

1. Does it do what the brief specifies? Where does it differ?
2. <The riskiest part of this change, stated concretely: e.g. are child processes cleaned
   up after a timeout; are arguments passed to the shell unescaped.>
3. <Trust boundary, if any: assume the client sends anything. Give an input that
   reproduces each finding.>
4. <Paths and file writes, if any: can `..`, absolute paths or links write outside the
   target folder?>
5. Can the tests actually fail? Any timing-dependent or flaky asserts? Do failures leave
   temp files or processes behind?
6. Did the change edit the checks themselves? If so, flag it and ask why.

## Round 2+ only

Round 1 findings: <attached>. The fixer's reply (a lead to verify, not a fact): <attached>.
Review the whole diff again, unless the manager marked this a small, local, low-risk
change (then check only the fixes and the code around them).
For each must-fix / should-fix from round 1, mark: resolved / not resolved / fix broke
something new. Report only what is still wrong and what is new. <These items were ruled
out of scope by the manager; do not report them again: ...>

## Output

For each finding: **severity** (must-fix / should-fix / note), **location** (`file:line`,
function), **why**, **a concrete input or scenario** that breaks it, suggested fix.
No praise for what is fine. State what you checked and what you did not.
Last line, exactly one of: **Verdict: OK to merge** / **Verdict: fix first**
