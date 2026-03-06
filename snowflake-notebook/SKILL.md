---
name: snowflake-notebook
description: Convert a markdown investigation or analysis document containing SQL queries into a Snowflake-native notebook (.ipynb) and snowflake.yml project definition, ready for `snow notebook deploy`. Use when you have an analysis doc with SQL code blocks that should become a shareable, re-runnable Snowflake Notebook.
allowed-tools: Read, Write, Glob, Bash(python3 *)
argument-hint: [markdown-file-path]
---

# Convert Markdown to Snowflake Notebook

Convert a markdown document into a Snowflake-native notebook project:
1. **Notebook (.ipynb)** — Snowflake notebook format with native SQL cells
2. **snowflake.yml** — project definition for `snow notebook deploy`

## Input

Read the markdown file at: `$ARGUMENTS`

If no path is provided, ask the user which file to convert.

## Conversion Rules

### Cell Mapping

- **Markdown sections** (headings, prose, tables, lists) become `"cell_type": "markdown"` cells
- **SQL code blocks** (` ```sql `) become `"cell_type": "code"` cells with `"language": "sql"`
- **Non-SQL code blocks** (python, bash, etc.) become `"cell_type": "code"` cells with `"language": "python"`
- Split at natural boundaries — each SQL block gets its own cell, with the preceding narrative as a separate markdown cell

### Snowflake Notebook Cell Format

**Markdown cell:**
```json
{
  "cell_type": "markdown",
  "id": "<unique-id>",
  "metadata": {
    "name": "<descriptive_snake_case_name>_md",
    "collapsed": false
  },
  "source": ["line 1\n", "line 2\n"]
}
```

**SQL cell:**
```json
{
  "cell_type": "code",
  "id": "<unique-id>",
  "metadata": {
    "language": "sql",
    "name": "<descriptive_snake_case_name>",
    "codeCollapsed": false
  },
  "source": ["SELECT ...\n", "FROM ..."],
  "execution_count": null,
  "outputs": []
}
```

**Python cell:**
```json
{
  "cell_type": "code",
  "id": "<unique-id>",
  "metadata": {
    "language": "python",
    "name": "<descriptive_snake_case_name>",
    "codeCollapsed": false
  },
  "source": ["import pandas as pd\n"],
  "execution_count": null,
  "outputs": []
}
```

### Cell Naming

Every cell needs a unique `name` in metadata. Use descriptive snake_case names:
- Markdown cells: `title_md`, `step1_header_md`, `interpretation_md`
- SQL cells: `baseline_trend`, `untagged_breakdown`, `receiver_sor_crossref`
- Names enable cross-cell references via `{{cell_name}}` in Snowflake Notebooks

### Source Array Format

The `source` field is an array of strings. Each string is one line INCLUDING the
trailing newline (`\n`), EXCEPT the last line which has no trailing newline.

### SQL Cell Content

- Strip the markdown code fence markers (` ```sql ` and ` ``` `)
- Preserve SQL comments (they provide context in the notebook)
- Preserve table names exactly as they appear in the source document
- Do NOT include semicolons at the end of queries (Snowflake Notebooks don't need them)

### Notebook Metadata

```json
{
  "metadata": {
    "kernelspec": {
      "display_name": "Streamlit Notebook",
      "name": "streamlit"
    }
  },
  "nbformat": 4,
  "nbformat_minor": 5
}
```

## Output Structure

Create a project directory alongside the source file:

```
<source-dir>/snowflake-notebooks/<notebook-name>/
  snowflake.yml
  <notebook-name>.ipynb
```

If the directory already exists, ask before overwriting.

### snowflake.yml Format

```yaml
definition_version: 2
entities:
  <notebook_name>:
    type: notebook
    query_warehouse: <warehouse>
    notebook_file: <notebook_name>.ipynb
    artifacts:
      - <notebook_name>.ipynb
```

For `query_warehouse`: ask the user which warehouse to use. If the Snowflake CLI
is available, you can check the default with:
```bash
snow sql -q "SELECT CURRENT_WAREHOUSE()"
```

## Deployment Instructions

After generating, print:

```
Files created:
  <path>/snowflake.yml
  <path>/<name>.ipynb

To deploy:
  cd <path> && snow notebook deploy --database <DB> --schema <SCHEMA>

To deploy with replace:
  cd <path> && snow notebook deploy --database <DB> --schema <SCHEMA> --replace

Or import manually: Open Snowsight > Notebooks > Import .ipynb file
```

## Validation

After writing the notebook, validate it:

```python
import json, sys
with open("<path>") as f:
    nb = json.load(f)
cells = nb.get("cells", [])
sql_cells = [c for c in cells if c.get("metadata", {}).get("language") == "sql"]
md_cells = [c for c in cells if c.get("cell_type") == "markdown"]
print(f"Valid notebook: {len(cells)} cells ({len(sql_cells)} SQL, {len(md_cells)} markdown)")
```

Report the cell count to the user.
