# GDB Review Skill

A Claude Code skill for reviewing GDB patches against the GNU/GDB coding
standards and conventions.

## Contents

The `gdb-review/` directory is a Claude Code skill. It contains:

- `SKILL.md` — Skill entry point and review workflow
- `review-rules.md` — What to check: commit messages, coding style, correctness, API consistency, test coverage, NEWS file
- `locate-source.md` — How to find the GDB source tree relative to the working directory
- `format-review.md` — How to structure and format review output

## Installation

Symlink the `gdb-review` directory (a subdirectory of this repository,
alongside this README) into your Claude skills directory:

```bash
ln -s /path/to/this/repo/gdb-review ~/.claude/skills/gdb-review
```

## Usage

In Claude Code, invoke the skill with:

```
/gdb-review              # review the top commit (HEAD~1..HEAD)
/gdb-review HEAD~3..HEAD  # review the last 3 commits
```

## Review Bot Integration

The `review-rules.md` file is also used by
[review-bot](https://github.com/user/review-bot) for automated reviews.
Point the review-bot config at this file:

```yaml
agent:
  review_rules_file: /path/to/this/repo/gdb-review/review-rules.md
```
