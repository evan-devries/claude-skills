# Claude Code Skills

Personal [Claude Code](https://claude.ai/code) skills for cross-project workflows.

## Skills

### `snowflake-notebook`

Converts a markdown investigation/analysis document containing SQL queries into a
[Snowflake Notebook](https://docs.snowflake.com/en/user-guide/ui-snowsight/notebooks)
(`.ipynb`) with native SQL cells, plus a `snowflake.yml` project definition ready
for [`snow notebook deploy`](https://docs.snowflake.com/en/developer-guide/snowflake-cli/command-reference/notebook-commands/deploy).

**Usage:**
```
/snowflake-notebook path/to/investigation.md
```

**What it does:**
- Reads a markdown doc with ` ```sql ` code blocks
- Splits into markdown cells (narrative) and SQL cells (`"language": "sql"`)
- Generates the `.ipynb` + `snowflake.yml` project definition
- Validates the output and prints deploy instructions

### `review-pr`

Adversarial code review of a pull request. Verifies claims against actual code,
finds bugs, edge cases, missing tests, and security issues. Produces a structured
findings document and a ready-to-post PR comment.

**Usage:**
```
/review-pr https://github.com/org/repo/pull/123
/review-pr 123
```

**What it does:**
- Fetches PR metadata and diff via `gh`
- Reads every changed file in full (not just diff hunks)
- Checks correctness, root cause analysis, safety, blast radius, test coverage, architecture fit
- Searches for similar patterns elsewhere in the codebase
- Writes structured findings doc (`review-pr-<number>.md`)
- Includes a ready-to-post PR comment and pre/post deploy validation steps

## Installation

Copy a skill directory into `~/.claude/skills/` (personal) or `.claude/skills/` (project-scoped):

```bash
cp -r snowflake-notebook ~/.claude/skills/
```
