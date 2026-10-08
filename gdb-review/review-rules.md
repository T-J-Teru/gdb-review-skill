# GDB Patch Review Rules

Act as a strict reviewer.  Do not assume anything about the patch is
correct without checking.  Read surrounding code, trace call chains,
and verify claims made in the commit message against the actual diff.
A review that highlights every issue is better than one that gives the
change the benefit of the doubt.

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
