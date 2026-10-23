---
name: implement-issue
description: "Implement GitHub issue <N> unattended, then push and open a draft PR for review."
disable-model-invocation: true
---

Implement GitHub issue `#<N>` (the argument) end to end, **AFK**. The run's only output is a **draft PR**. Nobody reads this session, so everything meant for the user goes into the PR body.

Wherever a skill you call would stop to ask the user (interfaces to test at, where the spec lives, a fixed point), make the call yourself and note it for the PR's **Decisions** section.

## Reading the issue

`gh issue` and `gh pr` use GraphQL, which cloud sessions block. Read through the REST API with
`gh api`, and spell out the repo: `gh` may not recognise the session's proxied git remote, so
`{owner}/{repo}` placeholders can fail.

    repo=hail-is/datafusion-sandbox
    gh api repos/$repo/issues/<N> \
      --jq '{title, state, labels: [.labels[].name], is_pr: (.pull_request != null), body}'
    gh api --paginate repos/$repo/issues/<N>/comments --jq '.[] | "--- \(.user.login):\n\(.body)"'

Read the comments as well as the body: refinements and decisions often land there, and a later
comment overrides the body.

Stop and report, without implementing, if:
- `is_pr` is true: the number is a pull request, not an issue (both share one number space);
- `state` is `closed`;
- the issue is blocked. List the open blockers with
  `gh api repos/$repo/issues/<N>/dependencies/blocked_by --jq '.[] | select(.state == "open") | .number'`.

Context the issue points at:
- **Parent**, often the spec this ticket was cut from:
  `gh api repos/$repo/issues/<N>/parent --jq '{number, title, body}'`. This returns 404 when
  there is none.
- **Other issues it references** (`#123`): read each one with the same call as above when the body
  depends on it.

## Steps

1. **Read the issue over REST.** 

2. **Get on a branch.** If you're on the default branch, create `issue-<N>` from it. Otherwise stay on the branch you were given.

3. **Build it.** Implement the spec, calling the Skill tool with "tdd" where possible, at pre-agreed interfaces. Run typechecking regularly, single test files regularly, and the full test suite once at the end; keep that final run's output for the PR's Evidence. Commit to the branch.

4. **Review it.** Call the Skill tool with "code-review", passing the default branch as the fixed point and the spec file's path as the spec. Fix what you can and commit. Every finding you leave unfixed goes in the PR.

5. **Push** the branch to origin.

6. **Open the draft PR.** Call the Skill tool with "pr" and write the body to its template, then append these sections:
   - `## Unfixed review findings`: every finding from step 4 you didn't fix. The user never sees the reviewers' output, so make each entry self-contained: the finding, the rule or spec line it cites, file:line, why you left it, and whether it needs the user's call. Write "None" if there are none.
   - `## Decisions`: each call you made in the user's place, with the option you passed over.
   - `Closes #<N>` as the final line.

   `POST repos/<owner>/<repo>/pulls` with the issue's title, `head` the branch, `base` the default branch, the body, and `"draft": true`. Build the JSON with `jq -n --rawfile body <file>` so the body escapes cleanly. If GitHub rejects `draft` (some plans lack draft PRs), open it ready for review and say so in the body's first line.

## When a step fails

A run that stalls (tests that won't go green, a spec too vague to build) still ends in the PR. Commit and push what you have, open the draft PR, and lead its body with `## Blocked`: what stopped you, what you tried, and what you need from the user. If no branch reaches GitHub (nothing to commit, or the push is rejected), post that same `## Blocked` text as a comment on the issue instead (`POST repos/<owner>/<repo>/issues/<N>/comments`).

The run is done when the draft PR (or that issue comment) exists on GitHub and holds everything above. End the session by printing its URL.
