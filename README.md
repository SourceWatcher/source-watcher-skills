# source-watcher-skills

Claude Code skills for [Source Watcher](https://github.com/TheCocoTeam/source-watcher) - a PHP ETL framework with a REST API and web UI.

## Skills

| Skill | Description |
|---|---|
| `list-steps` | List all available pipeline steps (extractors, transformers, loaders) from the running API |
| `author-pipeline` | Author a `.json` pipeline file from a natural language description |
| `run-pipeline` | Run a saved pipeline by name via the API, with timing and error reporting |
| `query-output` | Inspect and query SQLite output files produced by pipeline runs |

## Prerequisites

- [Source Watcher dev environment](https://github.com/TheCocoTeam/source-watcher-quickstart) running locally
- API available at `http://localhost:8181` (default)
- `sqlite3` CLI available on your host for `query-output`

## Usage

After installing these skills in Claude Code, invoke them with the `/source-watcher:` prefix:

```
/source-watcher:list-steps
/source-watcher:author-pipeline "fetch CVE data from the MITRE API and load into SQLite"
/source-watcher:run-pipeline cve-json-to-sqlite
/source-watcher:query-output cve-json-to-sqlite
```

## Example pipelines

Ready-to-run example pipeline files are available at [source-watcher-examples](https://github.com/TheCocoTeam/source-watcher-examples).

## License

MIT
