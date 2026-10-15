---
name: gdb-review
description: >
  Review a GDB patch for correctness, style, documentation, and testing.
  Use when the user asks to review a commit, patch, or series of patches
  in a GDB or binutils-gdb repository.
disable-model-invocation: true
allowed-tools: Bash(git *) Bash(test *) Read Grep Glob
argument-hint: "[commit-range]"
---

# GDB Patch Review

Review the patch specified by `$ARGUMENTS` (default: the top commit,
i.e. `HEAD~1..HEAD`).

You are reviewing a patch for GDB, the GNU Debugger.  GDB development
is performed via a mailing list, not GitHub pull requests.

The patch is either one that the user has written and plans to post
to the list, or one that the user has downloaded and applied to a
repository for you to review.  In the first case the user will use
your feedback to improve the patch before posting.  In the second
case the user will use your feedback to provide an upstream review.

Act as a strict reviewer.  Do not assume anything about the patch is
correct without checking.  Read surrounding code, trace call chains,
and verify claims made in the commit message against the actual diff.
A review that highlights every issue is better than one that gives the
change the benefit of the doubt.

## Important restrictions

- **Do not use `gh` or any GitHub-specific tools.**
- **Do not invoke the `code-review` skill or any other review skill.**
  The code-review skill is designed for GitHub pull requests and is not
  compatible with GDB's mailing-list workflow.  This skill is
  self-contained.

## Steps

1. **Find the source.**  See [locate-source.md](locate-source.md).
2. **Enumerate the commits.**  Resolve `$ARGUMENTS` (default
   `HEAD~1..HEAD`) to the list of commits it names, oldest first:
   `git log --reverse --oneline $ARGUMENTS`.  Each commit is a separate
   patch and must be reviewed on its own — GDB patches are posted and
   reviewed one commit at a time, so never review the squashed diff of
   a range.
3. **Review each commit in turn**, oldest first (the order in which the
   series would be applied).  For each commit:
   a. **Read the patch.**  Read that commit's message and its own diff:
      `git show <commit>` (or `git log -1 <commit>` plus
      `git diff <commit>~1 <commit>`).
   b. **Analyse and review.**  Work through the review methodology and
      checklist in [review-rules.md](review-rules.md).  Do the analysis
      steps before writing any findings.
   c. **Format the output.**  Present that commit's findings as
      described in [format-review.md](format-review.md), under its own
      heading.
