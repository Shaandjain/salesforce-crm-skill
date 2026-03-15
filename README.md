# Salesforce CRM Manager — Claude Desktop Skill

A plug-and-play Claude Desktop skill that lets Twin Peaks Wealth advisors manage Salesforce CRM using natural language. Create and manage accounts, contacts, leads, and tasks — all through conversation.

## What it does

| Capability | Examples |
|------------|----------|
| **Accounts** | "Create a new account called Kumar Family Trust", "Update the billing address for Test 1" |
| **Contacts** | "Find all contacts for the Kumar account", "Update Sarah's email to sarah@newdomain.com" |
| **Leads** | "Add Sarah Chen from Anthropic as a hot lead", "Search for leads in San Francisco" |
| **Tasks** | "Create a follow-up task for the Kumar account due next Friday", "What's on my plate?" |
| **Geographic Queries** | "Find all clients within 20 miles of Menlo Park", "Who are my clients in the Bay Area?" |
| **SOQL** | "Run a SOQL query to find all accounts in New York" |

## How to install

1. [Download this repo as a ZIP](../../archive/refs/heads/main.zip)
2. Unzip it and find the file at `tp-salesforce-crm-manager/SKILL.md`
3. Open **Claude Desktop** → **Settings** → **Skills**
4. Click **Upload skill** and select the `SKILL.md` file
5. That's it — Claude will now use this skill automatically when you ask about Salesforce

> **Note**: You only need the single `SKILL.md` file. Everything else in the repo (this README, etc.) is just documentation.

## Prerequisites

This skill uses **Composio** to connect to Salesforce. You need:

1. A **Composio** account with a Salesforce integration set up
2. The Composio MCP server configured in your Claude Desktop settings (`claude_desktop_config.json`), with server name `mcp-config-5-gpex`

If you're using a different MCP server name, update the references in `SKILL.md` to match your setup.

## What's covered

The skill provides full CRUD (create, read, update, delete) for each Salesforce object:

- **Accounts** — create, search, get, update, delete
- **Contacts** — create, search, get, update, delete
- **Leads** — create, search, get, update, delete
- **Tasks** — create, search, update, complete, delete
- **Geographic queries** — find clients by city, state, or radius
- **Raw SOQL** — run any SOQL query directly

The skill handles duplicate checking, name-to-ID resolution, and SOQL construction automatically so you don't have to think about Salesforce internals.

## Usage

Once installed, just talk to Claude naturally:

- "Create a new account called Test 1"
- "Add a task to the Test 1 account saying test is running"
- "Create a new lead for John Smith at Acme Corp, mark as warm"
- "Show me all open tasks"
- "Find contacts for the Kumar Family Trust"
- "Update Sarah Chen's phone number to (650) 555-1234"
- "Mark the Kumar follow-up task as complete"
- "I'm hosting an event in San Francisco — who are my clients nearby?"
- "Delete the Test 1 account"

## Extending the skill

Adding new operations is straightforward — open `SKILL.md` and append to the relevant object section following the same pattern: tool slug, parameter table, example, pitfalls. See the "Common Patterns" section at the bottom of the skill for multi-step workflow templates.

## License

MIT
