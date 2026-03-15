# jira

Minimal Jira CLI for Kanban-style workflows. Manages issues via the Jira REST API v3.

## Dependencies

- `bash` 4+
- `curl`
- `jq`

## Project structure

```
your-project/
  .jira.env
  .claude/
    commands/
      jira-sync.md
      jira-transition.md
    scripts/
      jira
  tasks/
    login.md
    checkout.md
```

## Setup

Place the `jira` script at `.claude/scripts/jira` and make it executable:

```bash
chmod +x .claude/scripts/jira
```

Create `~/.jira.env` with your global credentials:

```
JIRA_BASE_URL=https://yourorg.atlassian.net
JIRA_USER=you@example.com
JIRA_TOKEN=your-api-token
```

Create a `.jira.env` in your project directory (or any parent directory) with project-specific config:

```
JIRA_PROJECT=PROJ
```

The script walks up from the current working directory to find the nearest `.jira.env`. Local values override global ones.

Generate an API token at https://id.atlassian.net/manage-profile/security/api-tokens.

## Usage

```
.claude/scripts/jira create "Implement login page"
.claude/scripts/jira create "Fix header alignment" Bug
.claude/scripts/jira show PROJ-123
.claude/scripts/jira update PROJ-123 summary "New title"
.claude/scripts/jira transitions PROJ-123
.claude/scripts/jira transition PROJ-123 21
.claude/scripts/jira transition PROJ-123 "In Progress"
.claude/scripts/jira list
```

`create` returns the new issue key. `transitions` lists available status transitions for an issue (id and name, tab-separated). `transition` accepts both a transition ID and a transition name. `list` shows all open issues as tab-separated output (key, status, summary).

## Claude Code integration

Task files live in `tasks/` as markdown with YAML frontmatter:

```yaml
---
jira_key:
type: Story
status:
---
# Implement login page

Acceptance criteria and details here...
```

Two slash commands manage the Jira sync:

`/jira-sync tasks/login.md` reads the task file, creates the Jira issue from the H1 heading and type, and writes the returned key into the frontmatter. Aborts if the task is already synced.

`/jira-transition tasks/login.md` queries available transitions from Jira, presents them for selection, then updates both Jira and the frontmatter status field.

## Limitations

`list` shows all open issues (statusCategory != Done), not sprint-scoped. It does not support Scrum sprint filtering.

`update` sets field values as plain strings. This works for text fields like `summary` but not for structured fields like `assignee`, `priority`, or `labels`, which require JSON objects. For those, use the Jira web UI or extend the script.

`list` is capped at 50 results.
