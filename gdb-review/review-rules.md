# GDB Patch Review Rules

Act as a strict reviewer.  Do not assume anything about the patch is
correct without checking.  Read surrounding code, trace call chains,
and verify claims made in the commit message against the actual diff.
A review that highlights every issue is better than one that gives the
change the benefit of the doubt.

---

## Review Methodology

Before writing any findings, work through these steps in order.
Do not skip steps — the analysis you do here is what makes the
review thorough.

### Step 1: Build a mental model

For each file changed by the patch, read the surrounding code — not
just the changed lines.  Understand what the function did before the
change and what it does after.  For each modified function, identify:

- What invariants does this function assume about its inputs?
- What side effects does it have?
- What does its caller expect it to return or guarantee?

### Step 2: Trace callers and callees

For each function that was added or modified, read at least one
caller.  Check what values the caller actually passes.  Can it pass
NULL?  Can it pass a negative number?  Can the collection be empty?
If the patch adds a new function, check how it is called — does the
call site handle all possible return values?

For functions that were changed, check whether the change altered the
function's contract in a way that could break existing callers that
were not updated by this patch.

### Step 3: Adversarial analysis

For each change, actively try to construct a scenario where the new
code fails.  Ask:

- What input would make this crash or produce wrong results?
- What state would need to exist for this to go wrong?
- What happens if this is called concurrently, or re-entrantly?
- What happens at boundary values (zero, one, maximum, empty)?

If you can construct a concrete failure scenario, report it with a
reproducer: describe the steps a user would take in GDB to trigger
the bug.  A good reproducer includes a small C (or other language)
test program if needed, the GDB commands to run, and what you expect
to go wrong (crash, wrong output, assertion failure).  Do not attempt
to compile or run the reproducer yourself — just describe it clearly
enough that the reviewer can try it.

If you tried to construct a failure scenario and could not, that
strengthens your confidence in the code — but do not report "I tried
to break it and couldn't" as a finding.

### Step 4: Identify what is missing

Look beyond what the patch does and consider what it should have done
but didn't:

- A new code path that lacks error handling present in sibling paths.
- A struct that gained a new member but was not updated in all the
  places where the struct is initialised, copied, or serialised.
- A header that gained a new declaration but no corresponding
  documentation comment.
- A function that changed behaviour but the callers were not audited.
- A user-visible change that lacks a NEWS entry or documentation
  update.

### Step 5: Verify commit message claims

Read the commit message as a set of factual claims about the code.
For each non-trivial claim ("X now calls Y", "this fixes the case
where Z"), verify it against the actual diff.  Report any claim that
is inaccurate, misleading, or no longer true.

---

## 1. Commit Message

- Summary line must match what the diff actually does ("add" means
  wholly new, "update" means enhancement, "fix" means bug fix).
- Prefix with the component: `gdb:`, `gdb/python:`, `gdb/doc:`,
  `gdb/testsuite:`, etc.
- Present tense, imperative mood for the summary line.
- Wrap body at 72 columns.
- Reference specific functions, variables, or files when describing
  what changed.
- Technical claims must be accurate: if the message says "X calls Y",
  verify that it does.
- No spelling or grammar errors.
- Consistent terminology (don't switch between "core file" and
  "corefile" in prose).

## 2. Coding Style (GNU / GDB Conventions)

### C/C++

- Space before `(` in function calls: `foo (arg)`, not `foo(arg)`.
- Braces on their own line for function definitions.
- If any branch of an `if`/`else` chain uses braces, all branches must.
- Single-line blocks should not be wrapped with `{...}`, except when a
  comment is needed inside the block.
- The opening `{` for nested blocks goes on a line by itself.  Empty
  functions may use `{}`.
- Two-space indentation for continuation lines inside function arguments.
- Tabs for indentation in YACC/Lex files.
- No trailing whitespace.
- Comments use `/* ... */`, not `//`.
- Comments go on their own line, not trailing at end of a code line.
- Comment sentences start with a capital letter and end with two spaces
  before the next sentence.
- Use `nullptr` instead of `NULL`.  When editing a line that uses
  `NULL`, update it to `nullptr`.
- Non-ASCII characters should be avoided unless required by the patch.

### Python / DAP

- Follow PEP 8 with GDB-specific decorators (`@in_gdb_thread`,
  `@request`).
- Watch for `=` vs `==` in Python -- `x == value` is a valid no-op
  expression statement.

### General patterns to flag

- `if CONDITION return true else return false` should be `return CONDITION`.
- When the return values are reversed, prefer `return !CONDITION`.
- If a class needs to delete copy/assignment, use the
  `DISABLE_COPY_AND_ASSIGN` macro from `include/ansidecl.h`.

## 3. Comments and Documentation

- Parameter names in comments must match the actual signature.  Watch
  for stale names after a rename or refactor.
- Comments must describe the parameter/variable correctly (e.g. don't
  say "PSPACE" when the type changed to `inferior *`).
- Docstrings in Python/Tcl must match the class or function they
  describe (watch for copy-paste from a similar class).
- Comments for extern functions belong in the header file by the
  declaration.  The source file definition should say
  `/* See FILENAME.h.  */`.
- Comments for top-level static functions go in the source file just
  before the definition, with a blank line between comment and function.
- All functions, globals, and struct/union members should have a comment.

### Texinfo

- Use `@code{}` for identifiers/literals, `@kbd{}` for user-typed commands.
- Use `@value{GDBN}` instead of writing "GDB".
- `@xref` at sentence start, `@pxref` for parenthetical references;
  both must be followed by a period or comma.
- Verify cross-reference targets exist as nodes or anchors.
- Use "core file" (two words) in text; "corefile" only in code identifiers.

## 4. Code Correctness

- Assignment vs comparison: `=` vs `==`.
- Functions that accept a parameter but use a global instead.
- Observer/event callbacks: verify the event object has the attributes
  the handler accesses.
- Lifetime: code after `delete this` must not access members.  Code
  after `unpush_target` must account for potential deletion.
- Strings passed to GDB commands: filenames with spaces or special
  characters must be escaped.
- Resource leaks, null-pointer dereferences, off-by-one errors.

## 5. API Consistency

- New `emit_*_event` functions: compare with existing ones in the same
  file.  Do they take a `registry` parameter or use the global?  Be
  consistent.
- New test helper procs: follow conventions of neighbouring procs.
- New `parse_some_args` entries: single-element entries are boolean
  flags (0/1), two-element entries take a value with a default.
- Copy-paste errors: duplicated blocks where only part was updated
  (e.g. a docstring that refers to the wrong event type).

## 6. Test Coverage

- Each branch of new conditional logic should have a test.
- Error/failure paths: if the code handles "X not found" or "write
  failed", there should be a test exercising that path.
- Event counts: tests should verify exact counts, not just "at least one".
- Replacement/transition paths: e.g. loading a core file over an
  existing one.
- Edge cases: filenames with spaces, empty arguments, boundary values.
- Tests must clean up properly: `dap_shutdown` called, event counters
  reset, no leaked state between sections.

### Test style (DejaGnu/Expect)

- `gdb_test` for checking output, `gdb_test_no_output` for silent
  commands.
- `with_test_prefix` to disambiguate repeated test names.
- `gdb_assert` for Tcl-level assertions.  Prefer `gdb_assert` over
  `if { COND } { pass } else { fail }`.
- Use `$gdb_test_name` inside `gdb_test_multiple` instead of a
  separate `set test` variable.
- Use the `-lbl` flag in `gdb_test_multiple` when discarding lines
  with a catch-all pattern.
- Avoid `return VALUE` at the top level of a `.exp` script; just use
  `return` (the value is never checked).
- Use `gdb_watchdog` (from `testsuite/lib/gdb_watchdog.h`) instead of
  `alarm` for watchdog timers -- it works on both POSIX and Windows.
- Avoid `$srcdir/$subdir/$srcfile` in regexp patterns for GDB output.
  For MI tests use `${fullname_syntax}$srcfile`; for CLI tests use
  `[^\r\n]+/$srcfile`.

## 7. NEWS File

- New commands added to GDB must be mentioned in the NEWS file and in
  the commit message.
- Significant behaviour changes require a NEWS entry and may need a
  documentation update.
- General bug fixes do not require NEWS entries.
