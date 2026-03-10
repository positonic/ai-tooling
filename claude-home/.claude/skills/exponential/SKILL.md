---
description: "Manage tasks, projects, contacts, and deals via the Exponential CLI"
---

# Exponential CLI Skill

Interact with the Exponential productivity app via the `exponential` CLI. Use this skill when the user wants to manage tasks/actions, projects, contacts, deals, or workspaces in Exponential.

## Usage

Interpret the user's request and run the appropriate `exponential` command(s) via Bash. Always use `--json` flag when you need to parse output programmatically, and use the default pretty output when showing results to the user.

## Arguments

The user may pass arguments after `/exponential`. Interpret them as natural language instructions. Examples:
- `/exponential list my tasks` → `exponential actions list`
- `/exponential create task: Fix login bug, priority: 1st Priority` → `exponential actions create -n "Fix login bug" --priority "1st Priority"`
- `/exponential show today's tasks` → `exponential actions today`
- `/exponential list contacts` → `exponential contacts list --workspace <default>`

If no arguments provided, run `exponential actions today` to show today's tasks.

## Command Reference

### Actions (Tasks)
```bash
# List & query
exponential actions list [--project <id>] [--status <status>] [--assignee <id>]
exponential actions today [--workspace <id>]
exponential actions range --start <YYYY-MM-DD> --end <YYYY-MM-DD> [--workspace <id>]
exponential actions kanban [--project <id>] [--status <status>] [--assignee <id>]

# Create & update
exponential actions create -n <name> [-d <description>] [-p <project-id>] [--priority <priority>] [--due <YYYY-MM-DD>] [--effort <minutes>]
exponential actions update --id <id> [-n <name>] [-d <description>] [-p <project-id>] [--priority <priority>] [--status <status>] [--kanban <kanban>] [--due <date>]
```

**Kanban statuses:** BACKLOG, TODO, IN_PROGRESS, IN_REVIEW, DONE, CANCELLED
**Priorities:** Quick, Scheduled, 1st Priority, 2nd Priority, 3rd Priority, 4th Priority, 5th Priority, Errand, Remember, Watch, Someday Maybe
**Action statuses:** ACTIVE, COMPLETED, CANCELLED

### Projects
```bash
exponential projects list [--workspace <id>] [--include-actions]
```

### Workspaces
```bash
exponential workspaces list
exponential workspaces set-default <slug>
```

### Contacts (CRM)
```bash
exponential contacts list --workspace <id> [--search <query>] [--tags <tags>] [--organization <id>] [--limit <n>]
exponential contacts get <id> [--interactions]
exponential contacts create --workspace <id> [--first-name <name>] [--last-name <name>] [--email <email>] [--phone <phone>] [--linkedin <url>] [--telegram <handle>] [--twitter <handle>] [--github <handle>] [--about <text>] [--tags <tags>]
exponential contacts update --id <id> [--first-name <name>] [--last-name <name>] [--email <email>] ...
exponential contacts delete <id>
exponential contacts add-interaction --contact <id> --type <type> --direction <dir> [--subject <text>] [--notes <text>]
```

**Interaction types:** EMAIL, TELEGRAM, PHONE_CALL, MEETING, NOTE, LINKEDIN, OTHER
**Interaction directions:** INBOUND, OUTBOUND

### Deals (Pipeline)
```bash
exponential deals pipeline --workspace <id>
exponential deals stages --workspace <id>
exponential deals list --workspace <id>
exponential deals get <id>
exponential deals create --workspace <id> --stage <id> --title <title> [--description <text>] [--value <number>] [--currency <code>] [--probability <0-100>] [--close-date <YYYY-MM-DD>] [--contact <id>] [--organization <id>]
exponential deals update --id <id> [--title <title>] [--value <number>] [--stage <id>] ...
exponential deals move --id <id> --stage <id> [--order <n>]
exponential deals delete <id>
```

### Auth
```bash
exponential auth status
exponential auth whoami
exponential auth login --token <jwt> --api-url <url>
exponential auth logout
```

## Guidelines

1. **Always check auth first** if a command fails with unauthorized — run `exponential auth status`
2. **Use `--json` for parsing** — when you need to extract IDs or chain commands, add `--json` and parse the JSON output
3. **Workspace-scoped commands** — contacts, deals, and some action queries require `--workspace`. If not provided, the CLI uses the default workspace
4. **Setting fields to null** — use `--due null` or `--description null` to clear optional fields
5. **Date format** — always use YYYY-MM-DD for dates
6. **Multiple operations** — run independent commands in parallel for efficiency

## Workflow Examples

### Create a task from a conversation
```bash
exponential actions create -n "Implement user auth" -d "Add OAuth2 login flow" --priority "1st Priority" --due 2026-03-15
```

### Move a task through kanban
```bash
exponential actions update --id <id> --kanban IN_PROGRESS
# ... after completing work ...
exponential actions update --id <id> --kanban DONE --status COMPLETED
```

### Log a meeting with a contact
```bash
exponential contacts add-interaction --contact <id> --type MEETING --direction OUTBOUND --subject "Product demo" --notes "Discussed pricing, follow up next week"
```

### Create a deal from a contact interaction
```bash
# Get stages first
exponential deals stages --workspace <ws-id> --json
# Create deal
exponential deals create --workspace <ws-id> --stage <stage-id> --title "Enterprise deal" --value 50000 --contact <contact-id>
```

Now execute the user's request: $ARGUMENTS