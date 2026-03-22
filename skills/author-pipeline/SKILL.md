---
name: author-pipeline
description: >
  Author a Source Watcher pipeline file (.json) from a natural language description.
  Writes the file to the transformations directory so it can be run immediately.
argument-hint: <description of what the pipeline should do> [--name <pipeline-name>] [--api-url <url>]
allowed-tools: Bash
---

Create a Source Watcher pipeline file from a description.

Input: `$@`

## Step 1 - Understand the pipeline goal

Parse the user's description to identify:
- **Source**: what data to extract (CSV URL, JSON URL, local file, database)
- **Transformations**: any case conversion, column renaming, filtering needed
- **Destination**: where to load the output (SQLite file, table name)

If the description is ambiguous, ask one clarifying question before proceeding.

## Step 2 - Know the available steps

Use the step reference below to choose the right step types and option shapes.

### Extractors (`type: extractor`)

**Csv** - Extract rows from a CSV file or URL
```json
{
  "type": "extractor",
  "name": "Csv",
  "options": {
    "filePath": "https://example.com/data.csv",
    "columns": ["col1", "col2"],
    "delimiter": ",",
    "enclosure": "\""
  }
}
```

**Json** - Extract fields from a JSON file or URL using JSONPath
```json
{
  "type": "extractor",
  "name": "Json",
  "options": {
    "filePath": "https://api.example.com/data.json",
    "columns": {
      "output_column": "$.path.to.field",
      "another_column": "$.other.path"
    }
  }
}
```

**Txt** - Extract lines from a plain text file (each line = one row)
```json
{
  "type": "extractor",
  "name": "Txt",
  "options": {
    "filePath": "/var/www/html/.source-watcher/data/file.txt",
    "column": "line"
  }
}
```

**Database** - Extract rows from a database using SQL
```json
{
  "type": "extractor",
  "name": "Database",
  "options": {
    "driver": "pdo_sqlite",
    "path": "/path/to/database.sqlite",
    "query": "SELECT id, name FROM my_table"
  }
}
```
For MySQL/PostgreSQL use `driver: pdo_mysql` or `driver: pdo_pgsql` with `host`, `port`, `database`, `user`, `password` options instead of `path`.

**Find Missing** (execution-extractor) - Find gaps in a numeric sequence from the previous extractor
```json
{
  "type": "execution-extractor",
  "name": "FindMissingFromSequence",
  "options": {
    "filterField": "id"
  }
}
```

### Transformers (`type: transformer`)

**ConvertCase** - Change column name casing (does not affect values)
```json
{
  "type": "transformer",
  "name": "ConvertCase",
  "options": {
    "columns": ["col1", "col2"],
    "mode": "lower",
    "encoding": "UTF-8"
  }
}
```
Modes: `lower`, `upper`, `title`

**RenameColumns** - Rename columns
```json
{
  "type": "transformer",
  "name": "RenameColumns",
  "options": {
    "columns": {
      "old_name": "new_name"
    }
  }
}
```

**GuessGender** - Detect gender from a first name column
```json
{
  "type": "transformer",
  "name": "GuessGender",
  "options": {
    "firstNameField": "first_name",
    "genderField": "gender",
    "country": "usa"
  }
}
```

### Loaders (`type: loader`)

**Database** - Load rows into a database table
```json
{
  "type": "loader",
  "name": "Database",
  "options": {
    "driver": "pdo_sqlite",
    "tableName": "my_table",
    "path": "/var/www/html/.source-watcher/output.db",
    "memory": false
  }
}
```

### Pipeline rules
- Every pipeline needs at least one extractor and one loader.
- Steps execute in array order.
- Transformers operate between extractors and loaders.
- `execution-extractor` steps must follow a regular extractor - they read from its result.
- The `x` and `y` fields are optional canvas positions for the board UI (use `80 + i*220` and `100` as defaults).
- Output file paths inside the container use `/var/www/html/.source-watcher/` as the base.

### Pipeline file format

Pipeline files are JSON objects with a `$schema` reference and a `steps` array:

```json
{
  "$schema": "https://raw.githubusercontent.com/TheCocoTeam/source-watcher-api/master/pipeline.schema.json",
  "steps": [
    { "type": "extractor", "name": "...", "options": {} },
    { "type": "loader",    "name": "...", "options": {} }
  ]
}
```

## Step 3 - Write the pipeline file

Determine the pipeline name from `--name` if provided, otherwise derive it from the description (lowercase, hyphens, no spaces).

Write the file to:
```
source-watcher-quickstart/source-watcher-api/.source-watcher/transformations/<name>.json
```

Show the generated JSON to the user before writing and ask for confirmation if the pipeline has more than 3 steps or if the source is a database with credentials.

## Step 4 - Confirm and offer to run

After writing, show:
- The file path written
- A summary of the pipeline steps
- The command to run it:

```bash
TOKEN="<jwt>"
curl -s -X POST http://localhost:8181/api/v1/transformation-run \
  -H "Content-Type: application/json" \
  -H "x-access-token: $TOKEN" \
  -d '{"name": "<pipeline-name>"}'
```

Ask the user if they want to run it now. If yes, delegate to `/source-watcher:run-pipeline <pipeline-name>`.
