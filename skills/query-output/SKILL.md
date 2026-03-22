---
name: query-output
description: >
  Inspect and query SQLite output files produced by Source Watcher pipeline runs.
  Lists available databases, shows schemas, and runs SQL queries.
argument-hint: [<pipeline-name-or-db-path>] [--query <sql>]
allowed-tools: Bash
---

Inspect SQLite output from a Source Watcher pipeline run.

Input: `$@`

## Step 1 - Find available output databases

```bash
ls source-watcher-dev-env/source-watcher-api/.source-watcher/*.db 2>/dev/null
```

If a specific `<pipeline-name>` or `<db-path>` was provided, resolve it:
- If it ends in `.db`, use it directly as the path
- Otherwise, look for `<pipeline-name>.db` under `.source-watcher/`

If no `.db` files exist, tell the user no pipeline output has been produced yet and suggest running a pipeline first via `/source-watcher:run-pipeline`.

## Step 2 - Show available tables

```bash
sqlite3 <db-path> ".tables"
```

List the tables found.

## Step 3 - Show schema

For each table, show its schema:

```bash
sqlite3 <db-path> ".schema <table-name>"
```

## Step 4 - Run a query

If `--query <sql>` was provided, run it directly:

```bash
sqlite3 -column -header <db-path> "<sql>"
```

If no query was provided, show a sample of the first table:

```bash
sqlite3 -column -header <db-path> "SELECT * FROM <first-table> LIMIT 10;"
```

## Step 5 - Present results

Show the query output in a readable format.

If the result has more than 50 rows, note the truncation and suggest adding a `LIMIT` clause.

If the user wants to explore further, accept follow-up SQL queries and repeat Step 4.
