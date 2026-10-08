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
7. **Structural/Design Issues** — patch splitting, layering concerns.
8. **Testing** — missing test coverage, test style issues.
9. **Minor Issues** — typos, naming nits, minor suggestions.

Only include categories that have findings.  Do not include empty
sections.

## How to present a finding

Number findings sequentially across all categories.  For each finding:

- State the problem clearly in one sentence.
- Give the file and line number (or function name) where it occurs.
- Quote the relevant code when it helps make the issue concrete.
- Explain *why* it is a problem — don't just state the rule.
- When the fix is non-obvious, suggest a specific correction.

Do not pad findings with praise or hedging.  Be direct: "This is
wrong because..." not "This might potentially be an issue...".

## Assessment

End the review with an assessment line in exactly this format:

```
**Assessment:** READY_TO_APPROVE | NEEDS_CHANGES | UNCLEAR
```

- **READY_TO_APPROVE** — zero issues found.  No bugs, no style
  problems, no missing tests, no commit message issues.
- **NEEDS_CHANGES** — any issue was found, regardless of severity.
  This includes style nits, minor wording problems, and suggestions.
- **UNCLEAR** — insufficient context to judge patch quality.

If you reported any finding at all, the assessment must be
NEEDS_CHANGES.
