# Review Output Format

Present the review as a single markdown document.  Use the structure
below.

## Title

Start with a level-2 heading that includes the patch subject:

```
## Review: [PATCH v2 3/5] gdb: fix something in infrun.c
```

For a multi-patch series, review each patch separately with its own
heading.

## Findings

Group findings under level-3 headings by category.  Order the
categories by severity — most serious first:

1. **Correctness Issues** — bugs, logic errors, null dereferences,
   use-after-free, resource leaks, race conditions.
2. **Missing NEWS Entry** — if applicable per the review rules.
3. **Commit Message Issues** — inaccurate claims, missing component
   prefix, style problems.
4. **Coding Style Violations** — GNU/GDB style deviations.
5. **Comments and Documentation** — stale parameter names, missing
   comments, texinfo issues.
6. **API Consistency** — deviation from neighbouring code patterns.
7. **Structural/Design Issues** — patch splitting, layering concerns,
   and code smells per the Structure and Design rules (duplicated logic,
   feature envy, speculative generality, etc.).  These are judgement
   calls — label them as such.
8. **Testing** — missing test coverage, test style issues.
9. **Minor Issues** — typos, naming nits, minor suggestions.

Only include categories that have findings.  Do not include empty
sections.

These presentation categories deliberately do **not** mirror the
numbered sections of [review-rules.md](review-rules.md).  That file
groups the review *topically* (Commit Message, Coding Style, Comments
and Documentation, Code Correctness, API Consistency, Structure and
Design, Test Coverage, NEWS File); the list above groups the *output* by
severity, splits some rule sections across several categories, and adds a
presentation-only bucket ("Minor Issues") with no single backing rule
section.  The mapping is therefore intentionally not 1:1,
and the two lists are maintained independently — editing the rules does
not require renumbering these categories, and vice versa.  A finding
produced by any rule section should still land in whichever category
above best fits it.

## How to present a finding

Number findings sequentially across all categories.  For each finding:

- State the problem clearly in one sentence.
- Give the file and line number (or function name) where it occurs.
- Quote the relevant code when it helps make the issue concrete.
- Explain *why* it is a problem — don't just state the rule.
- When the fix is non-obvious, suggest a specific correction.
- Mark judgement calls.  If the finding is a heuristic or a matter of
  opinion rather than a concrete bug or a breach of a documented rule,
  begin it with `Judgement call:`.  Everything in the Structure and
  Design category (code smells) is a judgement call, as are optional
  naming and wording suggestions.  This marking drives the assessment
  below, so apply it honestly: do not dress a real bug as a judgement
  call to soften the verdict, and do not inflate a judgement call into a
  definite issue.

Do not pad findings with praise or hedging.  Be direct: "This is
wrong because..." not "This might potentially be an issue...".

Every finding must identify a concrete problem.  Do not create
numbered entries to note that something is correct or "all good" —
if there is no issue, there is no finding.

## Assessment

End the review with an assessment line in exactly this format:

```
**Assessment:** READY_TO_APPROVE | NEEDS_CHANGES | UNCLEAR
```

The verdict keys off *definite issues* only — a definite issue is a
concrete bug or a breach of a documented rule (correctness defect,
commit-message inaccuracy, GNU/GDB style violation, wrong or stale
comment, missing required NEWS entry, an untested new code path).  A
finding marked `Judgement call:` is not a definite issue.

- **NEEDS_CHANGES** — at least one definite issue was found, regardless
  of how minor (this includes style nits and typos — a documented-rule
  breach is still a breach).
- **READY_TO_APPROVE** — no definite issues.  This is still the verdict
  when the only findings are judgement calls; list them so the author
  can weigh them, but they do not block.
- **UNCLEAR** — insufficient context to judge patch quality.
