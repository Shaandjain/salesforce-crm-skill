---
name: tp-salesforce-crm-manager
description: |
  Salesforce CRM management via Composio. Use this skill whenever the user asks to interact with Salesforce — creating leads, viewing tasks, creating tasks, updating task status, querying client data, finding clients by location or geography, running SOQL queries, or any CRM-related workflow. Trigger on mentions of "Salesforce", "CRM", "leads", "contacts", "tasks", "SOQL", "pipeline", "my tasks", "create a lead", "add a lead", "update task status", "mark task complete", "find contact", "update contact", "search contacts", "list contacts", "clients near", "clients in", "event invites", "who lives near", or any request to read/write Salesforce data. Also trigger when the user says things like "what's on my plate", "show me my open tasks", "create a new lead", "find contacts for [account]", "update their email", "I'm hosting an event in [city]", or "find clients within X miles". This skill should be used even for simple Salesforce lookups — it contains the exact tool slugs and parameter schemas needed to execute reliably without guessing.
---

# Salesforce CRM Manager

This skill enables reliable Salesforce CRM operations through Composio's MCP integration. It covers the four core workflows Twin Peaks advisors need most: **creating leads**, **creating tasks**, **querying task status**, and **geographic client queries**.

Every tool slug, parameter schema, and execution pattern is documented here so you never need to guess or search — just execute.

## Setup & Connection

**MCP Server Config**: `mcp-config-5-gpex`

All Salesforce tools in this skill are routed through the Composio MCP server registered as `mcp-config-5-gpex`. This is the preconfigured Composio instance that holds the Salesforce OAuth connection for this workspace.

### Before first use

1. Verify the Salesforce connection is active by calling `COMPOSIO_CHECK_ACTIVE_CONNECTION` with `toolkit: "salesforce"` through the `mcp-config-5-gpex` server
2. If no active connection exists, call `COMPOSIO_MANAGE_CONNECTIONS` with `toolkits: ["salesforce"]` and guide the user through the OAuth flow
3. Once connected, note the `connected_account_id` and `user_id` from the connection details — you'll need them for owner-scoped queries

### How to execute tools

All Salesforce tools are called through the `mcp-config-5-gpex` Composio MCP server using `COMPOSIO_MULTI_EXECUTE_TOOL` (parallel/batch) or `COMPOSIO_EXECUTE_TOOL` (single):

```
Tool: mcp__mcp-config-5-gpex__COMPOSIO_MULTI_EXECUTE_TOOL
Parameters:
  sync_response_to_workbench: false
  tools: [{ tool_slug: "<SLUG>", arguments: { ... } }]
  memory: { "salesforce": ["<connection details you've gathered>"] }
  current_step: "<STEP_NAME>"
  thought: "<one-line rationale>"
```

**Important**: Always prefix Composio tool calls with `mcp__mcp-config-5-gpex__` to route through the correct MCP server. For example:
- `mcp__mcp-config-5-gpex__COMPOSIO_MULTI_EXECUTE_TOOL`
- `mcp__mcp-config-5-gpex__COMPOSIO_EXECUTE_TOOL`
- `mcp__mcp-config-5-gpex__COMPOSIO_SEARCH_TOOLS`
- `mcp__mcp-config-5-gpex__COMPOSIO_MANAGE_CONNECTIONS`
- `mcp__mcp-config-5-gpex__COMPOSIO_CHECK_ACTIVE_CONNECTION`

---

## Workflows

This skill is organized into sub-skills for each core workflow:

1. **[Create a Lead](create-lead/SKILL.md)** — Add new prospective clients to the pipeline
2. **[Create a Task](create-task/SKILL.md)** — Create follow-ups, reminders, and action items
3. **[Query Task Status](query-tasks/SKILL.md)** — View open tasks, check due dates, update or complete tasks
4. **[Find Contacts](find-contacts/SKILL.md)** — Search and list contacts by name, email, account, or location
5. **[Update a Contact](update-contact/SKILL.md)** — Edit contact details like email, phone, address, or title
6. **[Geographic Client Queries](geo-queries/SKILL.md)** — Find clients near a location for events or outreach

---

## Quick Reference: Tool Slugs

| Action | Tool Slug | Key Required Params |
|--------|-----------|-------------------|
| Create lead | `SALESFORCE_CREATE_LEAD` | `LastName`, `Company` |
| Search leads | `SALESFORCE_SEARCH_LEADS` | (all optional filters) |
| Get single lead | `SALESFORCE_GET_LEAD` | `lead_id` |
| Update lead | `SALESFORCE_UPDATE_LEAD` | `lead_id` |
| Create task | `SALESFORCE_CREATE_TASK` | `subject` |
| Search tasks | `SALESFORCE_SEARCH_TASKS` | (all optional filters) |
| Update task | `SALESFORCE_UPDATE_TASK` | `task_id` |
| Complete task | `SALESFORCE_COMPLETE_TASK` | `task_id` |
| Search contacts | `SALESFORCE_SEARCH_CONTACTS` | (all optional filters) |
| Get single contact | `SALESFORCE_GET_CONTACT` | `contact_id` |
| Update contact | `SALESFORCE_UPDATE_CONTACT` | `contact_id` |
| Search accounts | `SALESFORCE_SEARCH_ACCOUNTS` | (all optional filters) |
| List accounts | `SALESFORCE_LIST_ACCOUNTS` | (optional SOQL query) |
| Run SOQL | `SALESFORCE_RUN_SOQL_QUERY` | `query` |

---

## SOQL Quick Reference

Key syntax rules for writing SOQL queries:

- No `AS` keyword for aliases — use implicit: `SUM(Amount) TotalSales` not `SUM(Amount) AS TotalSales`
- `FIELDS(ALL)` requires `LIMIT 200`
- Wrap ID values in single quotes: `AccountId = '001...'`
- Escape apostrophes with backslash: `Name = 'O\'Brien'`
- Aggregate queries without `GROUP BY` cannot use `ORDER BY` or `LIMIT`
- `IN` clauses use parentheses with single-quoted values: `BillingCity IN ('Menlo Park', 'Palo Alto')`

---

## Status & Priority Reference

**Task Status**: "Not Started" (default), "In Progress", "Completed", "Waiting on someone else", "Deferred"

**Task Priority**: "High", "Normal" (default), "Low"

**Lead Status**: "New", "Working - Contacted", "Qualified", "Closed - Converted", "Closed - Not Converted"

**Lead Rating**: "Hot", "Warm", "Cold"

---

## Error Handling

- **400 INVALID_FIELD**: A field name doesn't exist on the object. Remove it and retry with fewer fields.
- **REQUIRED_FIELD_MISSING**: You're missing a required field (e.g. `LastName` + `Company` for leads, `subject` for tasks).
- **MALFORMED_ID**: An ID is not a valid 15/18-char Salesforce ID. Double-check the value.
- **Duplicate rules**: If create fails due to org duplicate rules, search first and decide whether to update instead.
- **Pagination**: If `response.data.done` is `false`, there are more records. Use `nextRecordsUrl` to fetch subsequent pages.

---

## Common Patterns

### Resolve an account name to ID before creating a task
1. `SALESFORCE_SEARCH_ACCOUNTS` with `name: "<partial name>"`
2. Extract `Id` from first match
3. Pass as `what_id` in `SALESFORCE_CREATE_TASK`

### Resolve a task by subject to ID before updating
1. `SALESFORCE_SEARCH_TASKS` with `subject: "<partial subject>"`
2. Extract `Id` from matching record
3. Pass as `task_id` in `SALESFORCE_UPDATE_TASK` or `SALESFORCE_COMPLETE_TASK`

### Check for duplicate leads before creating
1. `SALESFORCE_SEARCH_LEADS` with `email: "<email>"` or `name: "<name>"`
2. If results exist, inform user and offer to update instead
3. If no results, proceed with `SALESFORCE_CREATE_LEAD`
