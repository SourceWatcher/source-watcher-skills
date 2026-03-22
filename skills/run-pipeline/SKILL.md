---
name: run-pipeline
description: >
  Run a saved Source Watcher pipeline by name via the API.
  Handles authentication, executes the transformation, and reports results including duration.
argument-hint: <pipeline-name> [--api-url <url>] [--username <user>] [--password <pass>]
allowed-tools: Bash
---

Run a Source Watcher pipeline by name.

Input: `$@`

## Step 1 - Resolve arguments

Parse the input:
- `<pipeline-name>` (required) - the name of the `.swt` file without extension
- `--api-url` (optional, default: `http://localhost:8181`)
- `--username` / `--password` (optional, will prompt if not provided)

## Step 2 - Verify the pipeline file exists

```bash
ls source-watcher-dev-env/source-watcher-api/.source-watcher/transformations/<pipeline-name>.swt
```

If not found, list available pipelines:

```bash
ls source-watcher-dev-env/source-watcher-api/.source-watcher/transformations/*.swt
```

And tell the user which names are available.

## Step 3 - Authenticate

```bash
curl -s -X POST "$API_URL/api/v1/credentials" \
  -H "Content-Type: application/json" \
  -d '{"username": "<username>", "password": "<password>"}'
```

Extract `accessToken` from the response as `TOKEN`. If authentication fails, show the error and stop.

## Step 4 - Run the pipeline

Record the start time, then POST to the run endpoint:

```bash
START=$(date +%s%3N)

curl -s -X POST "$API_URL/api/v1/transformation-run" \
  -H "Content-Type: application/json" \
  -H "x-access-token: $TOKEN" \
  -d '{"name": "<pipeline-name>"}'

END=$(date +%s%3N)
```

## Step 5 - Report results

Compute elapsed time: `ELAPSED = (END - START) / 1000` seconds.

**On success** (HTTP 200), show:
```
Pipeline "<pipeline-name>" ran successfully.
Completed at: <UTC timestamp>
Execution took: <elapsed>s
```

Then check if the pipeline produced a SQLite output and offer to query it:
```bash
ls source-watcher-dev-env/source-watcher-api/.source-watcher/*.db 2>/dev/null
```

If `.db` files are found, ask the user if they want to inspect the output. If yes, delegate to `/source-watcher:query-output`.

**On failure** (non-200), show the full error response including:
- `message` - the general error
- `error` - the specific failure
- `stepIndex` + `stepName` - which step failed (if available)

Suggest checking:
1. The pipeline file for typos in step names or options
2. That the source data URL or file path is accessible from inside the container
3. Container logs: `docker compose -f source-watcher-dev-env/source-watcher-api/docker-compose.yml logs api`
