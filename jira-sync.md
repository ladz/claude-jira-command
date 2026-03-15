Sync a task file from tasks/ to Jira.

The jira script lives at .claude/scripts/jira (relative to project root).

Read the markdown file at $ARGUMENTS (relative to project root, e.g. tasks/login.md).
If the file does not exist or has no YAML frontmatter with jira_key, type, and status fields, abort with a clear error.

If jira_key is already set, abort and tell me the task is already synced.

Otherwise:
1. Extract the type from frontmatter (default: Story).
2. Extract the first H1 heading as the summary.
3. Run: jira create "<summary>" "<type>"
4. Take the returned key and write it back into the frontmatter as jira_key.
5. Run: jira show <key> to confirm the issue was created.
6. Show me the result.

Do not modify anything in the file other than the jira_key field.
