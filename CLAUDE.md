# CLAUDE.md

## What this is

`jira` is a single-file bash CLI that wraps the Jira REST API v3. It handles issue creation, updates, transitions, display, and listing. Two Claude Code slash commands (`/jira-sync`, `/jira-transition`) use the script to keep local task files in sync with Jira.

## Architecture

Single script at `.claude/scripts/jira`, no build step. Config is loaded from two sources: `~/.jira.env` (global credentials) and the nearest `.jira.env` walking up from CWD (project-specific). Local values override global ones.

All JSON payloads are constructed via `jq -n`, never via string interpolation. This is a deliberate design choice for safe escaping -- do not regress to string concatenation.

All API calls go through `jira_curl`, which checks HTTP status codes and extracts error messages. Commands that don't need the response body redirect to `/dev/null`.

## Task file format

Task files live in `tasks/` at the project root. Each file is markdown with YAML frontmatter:

```yaml
---
jira_key:
type: Task
status:
---
# Task title

Description and acceptance criteria...
```

`jira_key` is empty until `/jira-sync` fills it. `type` maps to a Jira issue type — currently only `Task` is supported. `status` reflects the current Jira board column and is updated by `/jira-transition`.

## Slash commands

`/jira-sync <path>` creates a Jira issue from the task file's H1 heading and type field, writes the key back into frontmatter. Must not modify anything else in the file.

`/jira-transition <path>` queries available transitions via `jira transitions <key>`, asks the user to choose, then runs `jira transition <key> <id or name>` and updates the frontmatter status field. `transition` accepts both transition IDs and names. Must not modify anything else in the file.

Both commands take a path relative to the project root (e.g. `tasks/login.md`).

## Dependencies

bash 4+, curl, jq, pandoc. No other runtime dependencies.

## Known limitations

- `cmd_update` only handles string-valued fields. Structured fields (assignee, priority, labels) need JSON objects that the current interface doesn't support.
- `cmd_list` shows all open issues (statusCategory != Done). No sprint filtering -- Scrum boards would need the Agile API and `sprint in openSprints()` JQL re-added.
- `cmd_list` caps at 50 results with no pagination.
- `base64` encoding uses `-w0` with a fallback for macOS compatibility.
- `md_to_adf` handles common markdown (headings, lists, code blocks, links, bold, italic, blockquotes, horizontal rules). Tables, images, and strikethrough are not converted to ADF equivalents.

## When modifying

- Keep all JSON construction in `jq`. No heredocs, no printf-based JSON.
- Every new command needs a `require_args` call as its first line.
- API calls that don't consume the response body must redirect stdout to `/dev/null`.
- Error messages go to stderr via `die`. User-facing success messages go to stdout.
- The `jira_curl` function is the single point for auth, headers, and error handling. Do not call `curl` directly.
- Slash commands must only modify the specific frontmatter fields they own (jira_key for sync, status for transition). Never touch the rest of the file.
