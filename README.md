# Salesforce CRM Manager — Claude Desktop Skill

A plug-and-play Claude Desktop skill that lets you manage your Salesforce CRM using natural language. Create leads, manage tasks, query clients by location, and more — all through conversation.

## What it does

| Workflow | Examples |
|----------|----------|
| **Create Leads** | "Add Sarah Chen from Anthropic as a hot lead" |
| **Create Tasks** | "Create a follow-up task for the Kumar account due next Friday" |
| **Query Tasks** | "What's on my plate?", "Show me high priority tasks due this week" |
| **Find Clients by Location** | "Find all clients within 20 miles of Menlo Park" |

## How to install

### Option A: Add as a Project

1. Download or clone this repo
2. Open **Claude Desktop** and create a new Project
3. Add the contents of the `tp-salesforce-crm-manager/` folder as project knowledge — upload each `SKILL.md` file as a project file

### Option B: Copy into your filesystem

1. Clone this repo or download it as a ZIP
2. Copy the `tp-salesforce-crm-manager/` folder somewhere accessible on your machine
3. In Claude Desktop, add the `SKILL.md` files as project knowledge so Claude can reference them during conversations

## Prerequisites

This skill uses **Composio** to connect to Salesforce. You need:

1. A **Composio** account with a Salesforce integration set up
2. The Composio MCP server configured in your Claude Desktop settings (`claude_desktop_config.json`), with server name `mcp-config-5-gpex`

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
