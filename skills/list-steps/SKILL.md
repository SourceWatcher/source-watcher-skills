---
name: list-steps
description: >
  List all available Source Watcher pipeline steps (extractors, transformers, loaders)
  from the running API. Use this before authoring a pipeline to know what steps are available.
argument-hint: [--api-url <url>]
allowed-tools: Bash
---

List all available Source Watcher pipeline steps from the API.

Input: `$@`

## Step 1 - Resolve API URL

Use the provided `--api-url` argument if given. Otherwise default to `http://localhost:8181`.

```bash
API_URL="http://localhost:8181"
# override if --api-url was passed
```

## Step 2 - Get a JWT token

Source Watcher requires authentication. Prompt the user for credentials if not already available in the session.

```bash
curl -s -X POST "$API_URL/api/v1/credentials" \
  -H "Content-Type: application/json" \
  -d '{"username": "<username>", "password": "<password>"}'
```

Extract `accessToken` from the response and store it as `TOKEN`.

## Step 3 - Fetch the steps list

```bash
curl -s "$API_URL/api/v1/steps" \
  -H "x-access-token: $TOKEN"
```

## Step 4 - Present the results

Parse the JSON response and display the steps grouped by type:

- **Extractors** - steps with `type: extractor` or `type: execution-extractor`
- **Transformers** - steps with `type: transformer`
- **Loaders** - steps with `type: loader`

For each step show: `name`, `object` (class name), and `description`.

If the API is unreachable, tell the user to ensure the Source Watcher API container is running:

```bash
cd source-watcher-quickstart/source-watcher-api && docker compose up -d api
```
