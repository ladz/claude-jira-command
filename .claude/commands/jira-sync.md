Sync a task file from tasks/ to Jira.

The jira script lives at .claude/scripts/jira (relative to project root).

Read the markdown file at $ARGUMENTS (relative to project root, e.g. tasks/login.md).
If the file does not exist or has no YAML frontmatter with jira_key, type, and status fields, abort with a clear error.

If jira_key is already set, abort and tell me the task is already synced.

Otherwise:
1. Extract the type from frontmatter (default: Task).
2. Extract the first H1 heading as the summary.
3. Extract the body: everything after the first H1 heading, trimmed of leading/trailing whitespace. This becomes the description.
4. Run: jira create "<summary>" "<type>" "<description>"
5. Take the returned key and write it back into the frontmatter as jira_key.
6. Run: jira show <key> to confirm the issue was created.
7. Show me the result.

Do not modify anything in the file other than the jira_key field.
