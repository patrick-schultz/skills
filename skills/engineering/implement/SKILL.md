---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Call the Skill tool with "tdd" where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, call the Skill tool with "code-review" to review the work.

Commit your work to the current branch.

End your final report with every review finding you did not fix. The user never sees the reviewers' output, so make each entry self-contained: the finding, the rule or spec line it cites, file:line, why you left it, and whether it needs the user's call.
