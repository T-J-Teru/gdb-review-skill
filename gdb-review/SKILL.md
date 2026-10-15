---
name: gdb-review
description: >
  Review a GDB patch for correctness, style, documentation, and testing,
  operating on a commit or range of commits in a GDB or binutils-gdb
  repository.  Invoked manually via /gdb-review; the model does not
  trigger this skill automatically.
disable-model-invocation: true
allowed-tools: Bash(git *) Bash(test *) Read Grep Glob Agent Task
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

The reviewer mindset to adopt is set out at the top of
[review-rules.md](review-rules.md); follow it.

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

   b. **Split the review across three parallel sub-agents.**  Issue all
      three `Agent` calls in a single message so they run in parallel;
      keeping the axes in separate sub-agents stops each one's context
      from crowding out the others.  Give every sub-agent:

      - the commit SHA and the exact diff command to use
        (`git show <commit>`);
      - the path to the GDB source tree located in step 1 (the
        sub-agent reviews with that as its working tree);
      - the **absolute path** to `review-rules.md` (it sits next to this
        SKILL.md) and the list of sections that axis owns, with an
        instruction to read that file and apply only those sections;
      - permission to read the surrounding GDB source as needed to judge
        the change (it may use Read/Grep/Glob and `git`);
      - the brief: "Return a plain list of findings only.  For each: the
        section or smell it comes from, the file and line (or function),
        the problem in one sentence, why it is a problem, and a suggested
        fix when non-obvious.  Flag each finding as either a definite
        issue (a concrete bug or a breach of a documented rule) or a
        judgement call (a heuristic or matter of opinion); everything
        from the Structure and Design section is a judgement call.  Do
        NOT write a formatted report, headings, or an assessment line --
        the caller does that.  Do not report that something is correct."

      The three axes, and the sections each one owns:

      - **Correctness** — the whole *Review Methodology* (build the
        mental model, trace callers and callees, adversarial analysis,
        identify what is missing, verify commit-message claims) plus
        *4. Code Correctness* and *7. Test Coverage*.  This is the deep
        semantic pass; it must trace call chains and actively try to
        construct concrete failure scenarios.
      - **Conventions** — *1. Commit Message*, *2. Coding Style*,
        *3. Comments and Documentation*, *5. API Consistency*, and
        *8. NEWS File*.
      - **Structure and Design** — *6. Structure and Design* (patch
        structure, layering, code smells).  Report these as judgement
        calls.

   c. **Aggregate and format.**  Collect the findings from all three
      sub-agents and present them for this commit as a single document
      per [format-review.md](format-review.md): one heading for the
      commit, findings merged into the severity-ordered categories, and
      one assessment line.  You own the formatting and the verdict; the
      sub-agents only supply raw findings.  De-duplicate where two axes
      report the same issue.
