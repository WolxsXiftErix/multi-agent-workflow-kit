# Bounce-back

<!-- Send back to the SAME agent that did the work, not a new one. Exactly four things
     per item. If this item has bounced 3 times, stop and re-check how the task was cut. -->

**To:** <the seat that wrote it>
**Bounce count for this item:** <1 / 2 / 3>

## Item <n>

- **Which:** <the finding or acceptance item, with `file:line`>
- **Why:** <what is wrong, one or two sentences>
- **Evidence:** <the last lines of the failing check, or the reproducing input and what
  happened>
- **Scope:** <what to change; what not to touch; which decisions stay as they are>

## Item <n+1>

- **Which:**
- **Why:**
- **Evidence:**
- **Scope:**

## When you are done

Rerun <check command> and report per item: what you changed (or why you did not, if you
think the finding is wrong), and the check's last lines. A seat that did not write the
fix will check it.
