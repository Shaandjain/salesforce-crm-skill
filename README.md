# Salesforce CRM Manager — Claude Code Skill

A plug-and-play Claude Code skill that lets you manage your Salesforce CRM using natural language. Create leads, manage tasks, query clients by location, and more — all through conversation.

## What it does

| Workflow | Examples |
|----------|----------|
| **Create Leads** | "Add Sarah Chen from Anthropic as a hot lead" |
| **Create Tasks** | "Create a follow-up task for the Kumar account due next Friday" |
| **Query Tasks** | "What's on my plate?", "Show me high priority tasks due this week" |
| **Find Clients by Location** | "Find all clients within 20 miles of Menlo Park" |

## How to install

### Option A: Download and drop in (easiest)

1. Download the `.skill` file from the [Releases](../../releases) page
2. Open **Claude Code** (the CLI)
3. Drag and drop the `.skill` file into your Claude Code conversation, or run:
   ```
   claude install-skill /path/to/tp-salesforce-crm-manager.skill
   ```

### Option B: Install from this repo

1. Clone this repo or download it as a ZIP
2. Copy the `tp-salesforce-crm-manager/` folder into your Claude Code skills directory:
   ```
   cp -r tp-salesforce-crm-manager/ ~/.claude/skills/tp-salesforce-crm-manager/
   ```
3. Restart Claude Code — the skill will be available immediately

## Prerequisites

This skill uses **Composio** to connect to Salesforce. You need:

1. A **Composio** account with a Salesforce integration set up
2. The Composio MCP server configured in your Claude Code settings (server name: `mcp-config-5-gpex`)

If you're using a different MCP server name, update the references in the SKILL.md files to match your setup.

## Skill structure

```
tp-salesforce-crm-manager/
  SKILL.md            # Main skill — setup, tool reference, SOQL guide
  create-lead/
    SKILL.md          # Create and search leads
  create-task/
    SKILL.md          # Create tasks linked to accounts
  query-tasks/
    SKILL.md          # Search, update, and complete tasks
  geo-queries/
    SKILL.md          # Find clients by city, state, or radius
```

## Usage

Once installed, just talk to Claude naturally:

- "Create a new lead for John Smith at Acme Corp, mark as warm"
- "Show me all open tasks"
- "Mark the Kumar follow-up task as complete"
- "I'm hosting an event in San Francisco — who are my clients nearby?"
- "Run a SOQL query to find all accounts in New York"

The skill handles duplicate checking, account-to-ID resolution, and SOQL query construction automatically.

## License

MIT
