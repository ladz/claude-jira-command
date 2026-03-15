Transition a synced task to a new Jira status.

The jira script lives at .claude/scripts/jira (relative to project root).

Read the markdown file at $ARGUMENTS (relative to project root, e.g. tasks/login.md).
If the file does not exist or has no YAML frontmatter with jira_key, type, and status fields, abort with a clear error.

If jira_key is empty, abort and tell me to run /jira-sync first.

Otherwise:
1. Run: jira transitions <jira_key> to get the available transitions.
2. Show me the available statuses and ask which one I want.
3. After I choose, run: jira transition <jira_key> "<chosen status>"
4. Update the status field in the frontmatter to match the new status.
5. Run: jira show <jira_key> to confirm.
6. Show me the result.

Do not modify anything in the file other than the status field.
