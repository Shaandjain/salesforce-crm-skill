---
name: tp-salesforce-crm-manager
description: |
  Salesforce CRM management via Composio for Twin Peaks Wealth advisors. Use for any Salesforce interaction: creating/searching leads, creating/querying/completing tasks, finding/updating contacts, geographic client queries, or running SOQL. Trigger on "Salesforce", "CRM", "leads", "contacts", "tasks", "SOQL", "pipeline", "create a lead", "my tasks", "find contact", "update contact", "clients near", "what's on my plate", or any CRM read/write request. Contains exact tool slugs and parameter schemas for reliable execution.
---

# Twin Peaks Salesforce CRM Manager

This skill enables reliable Salesforce CRM operations for Twin Peaks Wealth advisors through Composio's MCP integration. It covers six core workflows: **creating leads**, **creating tasks**, **querying task status**, **finding contacts**, **updating contacts**, and **geographic client queries**.

Every tool slug, parameter schema, and execution pattern is documented here so you never need to guess or search — just execute.

---

## Setup & Connection

**MCP Server Config**: `mcp-config-5-gpex`

All Salesforce tools in this skill are routed through the Composio MCP server registered as `mcp-config-5-gpex`. This is the preconfigured Composio instance that holds the Salesforce OAuth connection for Twin Peaks.

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

## Workflow 1: Create a Lead

Use when a Twin Peaks advisor wants to add a new prospective client into the pipeline.

**Tool**: `SALESFORCE_CREATE_LEAD`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `LastName` | string | **Yes** | Lead's last name |
| `Company` | string | **Yes** | Lead's company or affiliation |
| `FirstName` | string | No | Lead's first name |
| `Email` | string | No | Email address |
| `Phone` | string | No | Phone number |
| `Title` | string | No | Job title (max 128 chars) |
| `Status` | string | No | "New", "Working - Contacted", "Qualified", "Closed - Converted", "Closed - Not Converted" |
| `LeadSource` | string | No | "Web", "Phone Inquiry", "Partner Referral", "Purchased List", "Other" |
| `Rating` | string | No | "Hot", "Warm", "Cold" |
| `Industry` | string | No | e.g. "Technology", "Finance", "Healthcare" |
| `Street` | string | No | Street address |
| `City` | string | No | City |
| `State` | string | No | State/province (e.g. "CA" or "California") |
| `PostalCode` | string | No | Zip code |
| `Country` | string | No | Country |
| `Website` | string | No | Company website |
| `AnnualRevenue` | number | No | Estimated annual revenue |
| `NumberOfEmployees` | integer | No | Employee count |
| `CustomFields` | object | No | Custom field API names to values (keys typically end in `__c`) |
| `allow_duplicates` | boolean | No | Default `false`. Set `true` to bypass duplicate detection |

### Before creating a lead

Always check for duplicates first using `SALESFORCE_SEARCH_LEADS`:

```json
{ "email": "prospect@example.com" }
```

or by name:

```json
{ "name": "Jane Doe", "company": "Acme Corp" }
```

If a match is found, inform the advisor and offer to update the existing lead instead of creating a duplicate.

### Example — create a new lead

```json
{
  "LastName": "Chen",
  "FirstName": "Sarah",
  "Company": "Anthropic",
  "Email": "sarah.chen@anthropic.com",
  "Title": "Senior Engineer",
  "City": "San Francisco",
  "State": "CA",
  "Status": "New",
  "LeadSource": "Partner Referral",
  "Rating": "Hot"
}
```

**Response parsing**: On success, the new Lead ID is at `response.data.id`. Use `SALESFORCE_GET_LEAD` with the returned ID if you need to confirm field persistence.

### Pitfalls

- 400 validation errors mean org-required fields are missing or invalid. Check `data.errors` for specifics
- If `allow_duplicates` is `false` (default) and duplicates exist, creation will fail — search first

---

## Workflow 2: Create a Task

Use when a Twin Peaks advisor wants to create a follow-up, reminder, or action item tied to a client or lead.

**Tool**: `SALESFORCE_CREATE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `subject` | string | **Yes** | Task title (e.g. "Follow up with client about Q2 review") |
| `status` | string | No | Default: "Not Started". Options: "Not Started", "In Progress", "Completed", "Waiting on someone else", "Deferred" |
| `priority` | string | No | Default: "Normal". Options: "High", "Normal", "Low" |
| `activity_date` | string | No | Due date in YYYY-MM-DD format |
| `description` | string | No | Detailed notes |
| `what_id` | string | No | Related Account/Opportunity ID (prefix `001` for Account, `006` for Opportunity) |
| `who_id` | string | No | Related Contact (`003`) or Lead (`00Q`) ID |
| `owner_id` | string | No | User ID of task owner. Defaults to current user if omitted |
| `is_reminder_set` | boolean | No | Whether to set a reminder |
| `reminder_date_time` | string | No | ISO datetime for reminder (required if `is_reminder_set=true`) |
| `custom_fields` | object | No | Custom field API names to values |

### Before creating a task

If the advisor mentions a client or account name (not an ID), resolve the Account ID first:

1. Call `SALESFORCE_SEARCH_ACCOUNTS` with `name: "<account name>"`
2. Extract the `Id` from the first matching record
3. Pass that `Id` as `what_id`

This ensures the task appears on the account's activity timeline. Omitting `what_id` creates a standalone task.

### Example — create a follow-up task

```json
{
  "subject": "Schedule Q2 portfolio review with the Kumars",
  "priority": "High",
  "activity_date": "2026-03-25",
  "what_id": "001dL00001xrytEQAQ",
  "status": "Not Started",
  "description": "Discuss rebalancing strategy and tax-loss harvesting opportunities"
}
```

---

## Workflow 3: Query Task Status

Use when a Twin Peaks advisor wants to see their open tasks, check what's due, or look up the status of specific tasks.

### Option A: Search with filters (recommended)

**Tool**: `SALESFORCE_SEARCH_TASKS`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | "Not Started", "In Progress", "Completed", "Waiting on someone else", "Deferred" |
| `priority` | string | No | "High", "Normal", "Low" |
| `is_closed` | boolean | No | `false` for open tasks, `true` for closed |
| `assigned_to_name` | string | No | Partial match on owner name |
| `account_name` | string | No | Partial match on related account name |
| `contact_name` | string | No | Partial match on related contact name |
| `subject` | string | No | Partial match on task subject |
| `activity_date_from` | string | No | YYYY-MM-DD start date |
| `activity_date_to` | string | No | YYYY-MM-DD end date |
| `limit` | integer | No | Max results (default 50, max 2000) |
| `fields` | string | No | Default: `Id,Subject,Status,Priority,ActivityDate,IsClosed,Description,OwnerId,Owner.Name,WhatId,What.Name,WhoId,Who.Name` |

**Example — show my open tasks:**
```json
{
  "is_closed": false,
  "assigned_to_name": "<advisor's name>"
}
```

**Example — high priority tasks due this week:**
```json
{
  "priority": "High",
  "is_closed": false,
  "activity_date_from": "2026-03-09",
  "activity_date_to": "2026-03-15"
}
```

**Example — tasks for a specific client:**
```json
{
  "account_name": "Kumar",
  "is_closed": false
}
```

### Option B: Custom SOQL (for advanced queries)

**Tool**: `SALESFORCE_RUN_SOQL_QUERY`

```
query: "SELECT Id, Subject, Status, Priority, ActivityDate, What.Name, Who.Name FROM Task WHERE OwnerId = '<user_id>' AND IsClosed = false ORDER BY ActivityDate ASC"
```

### Updating task status

**Tool**: `SALESFORCE_UPDATE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | **Yes** | Salesforce Task ID |
| `status` | string | No | New status value |
| `priority` | string | No | New priority value |
| `activity_date` | string | No | New due date (YYYY-MM-DD) |
| `description` | string | No | Updated description |
| `what_id` | string | No | Updated related record ID |
| `who_id` | string | No | Updated Contact/Lead ID |
| `custom_fields` | object | No | Custom field updates |

If the advisor refers to a task by name, resolve its ID first with `SALESFORCE_SEARCH_TASKS` using `subject: "<partial subject>"`.

### Marking a task complete

**Tool**: `SALESFORCE_COMPLETE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | **Yes** | Salesforce Task ID |
| `completion_notes` | string | No | Notes appended to description |

This convenience wrapper sets Status to "Completed" and IsClosed to true.

---

## Workflow 4: Find Contacts

Use when a Twin Peaks advisor wants to search for contacts, look up a specific person, or list contacts associated with an account.

### Step 1 — Search for contacts

**Tool**: `SALESFORCE_SEARCH_CONTACTS`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | No | Partial match on contact name |
| `email` | string | No | Partial match on email address |
| `phone` | string | No | Partial match on phone number |
| `account_name` | string | No | Partial match on related account name |
| `title` | string | No | Partial match on job title |
| `department` | string | No | Partial match on department |
| `mailing_city` | string | No | Partial match on mailing city |
| `mailing_state` | string | No | Partial match on mailing state |
| `limit` | integer | No | Max results (default 50, max 2000) |
| `fields` | string | No | Default: `Id,Name,FirstName,LastName,Email,Phone,Title,Department,Account.Name,MailingCity,MailingState` |

**Example — find a contact by name:**
```json
{
  "name": "Sarah Chen"
}
```

**Example — list all contacts for an account:**
```json
{
  "account_name": "Kumar Family Trust"
}
```

**Example — find contacts by email domain:**
```json
{
  "email": "@anthropic.com"
}
```

### Step 2 — Get full details for a specific contact

**Tool**: `SALESFORCE_GET_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | string | **Yes** | Salesforce Contact ID (prefix `003`) |

### Alternative: Custom SOQL

**Tool**: `SALESFORCE_RUN_SOQL_QUERY`

```
query: "SELECT Id, Name, Email, Phone, Title, Account.Name, MailingCity, MailingState FROM Contact WHERE Account.Name LIKE '%Kumar%' ORDER BY LastName ASC"
```

### Present results clearly

Format results as a clean table or list showing: name, email, phone, title, and associated account. If the advisor is looking for a specific person, highlight the best match and confirm before proceeding with any follow-up action.

---

## Workflow 5: Update a Contact

Use when a Twin Peaks advisor wants to modify an existing contact's information — email, phone, address, title, or any other field.

### Step 1 — Find the contact to update

If the advisor refers to a contact by name (not by ID), resolve the Contact ID first:

1. Call `SALESFORCE_SEARCH_CONTACTS` with `name: "<contact name>"`
2. If multiple results, present the matches and ask the advisor to confirm which one
3. Extract the `Id` from the confirmed record

**Example — resolve by name:**
```json
{
  "name": "Sarah Chen"
}
```

**Example — resolve by name + account:**
```json
{
  "name": "Sarah",
  "account_name": "Anthropic"
}
```

### Step 2 — Update the contact

**Tool**: `SALESFORCE_UPDATE_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | string | **Yes** | Salesforce Contact ID (prefix `003`) |
| `FirstName` | string | No | Updated first name |
| `LastName` | string | No | Updated last name |
| `Email` | string | No | Updated email address |
| `Phone` | string | No | Updated phone number |
| `MobilePhone` | string | No | Updated mobile number |
| `Title` | string | No | Updated job title |
| `Department` | string | No | Updated department |
| `MailingStreet` | string | No | Updated street address |
| `MailingCity` | string | No | Updated city |
| `MailingState` | string | No | Updated state/province |
| `MailingPostalCode` | string | No | Updated zip code |
| `MailingCountry` | string | No | Updated country |
| `Description` | string | No | Updated description/notes |
| `custom_fields` | object | No | Custom field API names to values (keys typically end in `__c`) |

Only include the fields you want to change — omitted fields are left untouched.

**Example — update email and phone:**
```json
{
  "contact_id": "003dL00001ABC123",
  "Email": "sarah.chen@newdomain.com",
  "Phone": "(650) 555-1234"
}
```

**Example — update mailing address:**
```json
{
  "contact_id": "003dL00001ABC123",
  "MailingStreet": "123 Main St",
  "MailingCity": "San Francisco",
  "MailingState": "CA",
  "MailingPostalCode": "94105"
}
```

### Pitfalls

- **MALFORMED_ID**: Ensure the contact ID is a valid 15/18-character Salesforce ID starting with `003`
- **INVALID_FIELD**: A field name doesn't exist on the Contact object. Remove it and retry
- If multiple contacts match the name, always confirm with the advisor before updating — never guess

---

## Workflow 6: Geographic Client Queries

Use when a Twin Peaks advisor wants to find clients near a location — for event planning, regional outreach, or proximity-based filtering. This is particularly useful for scenarios like "I'm hosting an event in Menlo Park, give me all clients within 20 miles."

### Step 1 — Query accounts by city/state/region

Use `SALESFORCE_RUN_SOQL_QUERY` to pull accounts in the target area:

```
query: "SELECT Id, Name, Phone, BillingStreet, BillingCity, BillingState, BillingPostalCode, BillingCountry FROM Account WHERE BillingCity = 'Menlo Park' AND BillingState = 'CA'"
```

For broader regional queries, use nearby cities:

```
query: "SELECT Id, Name, Phone, BillingStreet, BillingCity, BillingState, BillingPostalCode FROM Account WHERE BillingState = 'CA' AND BillingCity IN ('Menlo Park', 'Palo Alto', 'Redwood City', 'Mountain View', 'San Mateo', 'Sunnyvale', 'Los Altos', 'Atherton', 'Woodside', 'Portola Valley', 'East Palo Alto', 'San Carlos', 'Belmont', 'Foster City', 'Fremont', 'Santa Clara', 'San Jose', 'Cupertino', 'Milpitas', 'Newark')"
```

### Step 2 — For radius-based queries, calculate distances

When the advisor asks for "within X miles," after pulling accounts with addresses, use the Composio Remote Workbench to calculate distances:

```python
from math import radians, cos, sin, asin, sqrt

def haversine(lat1, lon1, lat2, lon2):
    """Calculate distance in miles between two lat/lon points."""
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    return 2 * 3956 * asin(sqrt(a))  # 3956 = Earth radius in miles
```

Use a geocoding lookup (via web search or a known city-to-lat/lon mapping) to convert city names to coordinates, then filter accounts that fall within the requested radius.

### Step 3 — Present results clearly

Format results as a clean list showing: client name, city, estimated distance, and phone number. Group by proximity (closest first). If the advisor mentions an event, suggest composing an outreach list.

### Common geographic queries

| Advisor says | SOQL approach |
|-----------|--------------|
| "Clients in San Francisco" | `WHERE BillingCity = 'San Francisco'` |
| "Clients in the Bay Area" | `WHERE BillingCity IN ('San Francisco', 'Oakland', 'San Jose', 'Palo Alto', ...)` |
| "Clients in California" | `WHERE BillingState = 'CA'` |
| "Clients within 20 miles of Menlo Park" | Query CA accounts, geocode, haversine filter |
| "Clients on the East Coast" | `WHERE BillingState IN ('NY', 'NJ', 'CT', 'MA', 'PA', ...)` |

### Contacts variant

If the advisor asks about contacts (people) rather than accounts (companies), query the Contact object instead:

```
query: "SELECT Id, Name, Email, Phone, MailingCity, MailingState, Account.Name FROM Contact WHERE MailingCity = 'Menlo Park' AND MailingState = 'CA'"
```

### Leads variant

For leads not yet converted to accounts:

```
query: "SELECT Id, Name, Email, Phone, City, State, Company FROM Lead WHERE City = 'Menlo Park' AND State = 'CA' AND IsConverted = false"
```

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

### Resolve a contact by name to ID before updating
1. `SALESFORCE_SEARCH_CONTACTS` with `name: "<partial name>"`
2. Confirm with advisor if multiple matches
3. Pass `Id` as `contact_id` in `SALESFORCE_UPDATE_CONTACT`

### Resolve a task by subject to ID before updating
1. `SALESFORCE_SEARCH_TASKS` with `subject: "<partial subject>"`
2. Extract `Id` from matching record
3. Pass as `task_id` in `SALESFORCE_UPDATE_TASK` or `SALESFORCE_COMPLETE_TASK`

### Check for duplicate leads before creating
1. `SALESFORCE_SEARCH_LEADS` with `email: "<email>"` or `name: "<name>"`
2. If results exist, inform advisor and offer to update instead
3. If no results, proceed with `SALESFORCE_CREATE_LEAD`

---

## Adding More Workflows

Adding a new workflow to this skill is straightforward — just append a new `## Workflow N` section following the same pattern:

1. State when to use it
2. List the tool slug and parameter table
3. Show any prerequisite steps (e.g. resolving a name to an ID)
4. Include an example with realistic Twin Peaks data
5. Note any pitfalls
