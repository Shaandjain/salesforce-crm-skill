---
name: tp-salesforce-crm-manager
description: |
  Salesforce CRM management via Composio for Twin Peaks Wealth advisors. Use for any Salesforce interaction: creating/searching/updating/deleting accounts, contacts, leads, and tasks, geographic client queries, or running SOQL. Trigger on "Salesforce", "CRM", "leads", "contacts", "accounts", "tasks", "SOQL", "pipeline", "create a lead", "my tasks", "find contact", "update contact", "clients near", "what's on my plate", or any CRM read/write request. Contains exact tool slugs and parameter schemas for reliable execution.
---

# Twin Peaks Salesforce CRM Manager

This skill enables reliable Salesforce CRM operations for Twin Peaks Wealth advisors through Composio's MCP integration. It provides full CRUD operations across four Salesforce objects — **Accounts**, **Contacts**, **Leads**, and **Tasks** — plus geographic client queries and raw SOQL access.

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

## Quick Reference: All Tool Slugs

### Accounts
| Action | Tool Slug | Key Required Params |
|--------|-----------|-------------------|
| Create account | `SALESFORCE_CREATE_ACCOUNT` | `Name` |
| Search accounts | `SALESFORCE_SEARCH_ACCOUNTS` | (all optional filters) |
| List accounts | `SALESFORCE_LIST_ACCOUNTS` | (optional SOQL query) |
| Get single account | `SALESFORCE_GET_ACCOUNT` | `account_id` |
| Update account | `SALESFORCE_UPDATE_ACCOUNT` | `account_id` |
| Delete account | `SALESFORCE_DELETE_ACCOUNT` | `account_id` |

### Contacts
| Action | Tool Slug | Key Required Params |
|--------|-----------|-------------------|
| Create contact | `SALESFORCE_CREATE_CONTACT` | `LastName` |
| Search contacts | `SALESFORCE_SEARCH_CONTACTS` | (all optional filters) |
| Get single contact | `SALESFORCE_GET_CONTACT` | `contact_id` |
| Update contact | `SALESFORCE_UPDATE_CONTACT` | `contact_id` |
| Delete contact | `SALESFORCE_DELETE_CONTACT` | `contact_id` |

### Leads
| Action | Tool Slug | Key Required Params |
|--------|-----------|-------------------|
| Create lead | `SALESFORCE_CREATE_LEAD` | `LastName`, `Company` |
| Search leads | `SALESFORCE_SEARCH_LEADS` | (all optional filters) |
| Get single lead | `SALESFORCE_GET_LEAD` | `lead_id` |
| Update lead | `SALESFORCE_UPDATE_LEAD` | `lead_id` |
| Delete lead | `SALESFORCE_DELETE_LEAD` | `lead_id` |

### Tasks
| Action | Tool Slug | Key Required Params |
|--------|-----------|-------------------|
| Create task | `SALESFORCE_CREATE_TASK` | `subject` |
| Search tasks | `SALESFORCE_SEARCH_TASKS` | (all optional filters) |
| Update task | `SALESFORCE_UPDATE_TASK` | `task_id` |
| Complete task | `SALESFORCE_COMPLETE_TASK` | `task_id` |
| Delete task | `SALESFORCE_DELETE_TASK` | `task_id` |

### Other
| Action | Tool Slug | Key Required Params |
|--------|-----------|-------------------|
| Run SOQL | `SALESFORCE_RUN_SOQL_QUERY` | `query` |

---

## Accounts

### Create an account

**Tool**: `SALESFORCE_CREATE_ACCOUNT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `Name` | string | **Yes** | Account name (e.g. "Kumar Family Trust") |
| `Phone` | string | No | Phone number |
| `Website` | string | No | Company website |
| `Industry` | string | No | e.g. "Financial Services", "Technology" |
| `BillingStreet` | string | No | Street address |
| `BillingCity` | string | No | City |
| `BillingState` | string | No | State/province |
| `BillingPostalCode` | string | No | Zip code |
| `BillingCountry` | string | No | Country |
| `Description` | string | No | Account description/notes |
| `Type` | string | No | "Customer", "Partner", "Prospect", etc. |
| `custom_fields` | object | No | Custom field API names to values (keys end in `__c`) |

**Example:**
```json
{
  "Name": "Test 1",
  "Type": "Prospect",
  "BillingCity": "San Francisco",
  "BillingState": "CA"
}
```

**Response**: New Account ID at `response.data.id`.

### Search accounts

**Tool**: `SALESFORCE_SEARCH_ACCOUNTS`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | No | Partial match on account name |
| `industry` | string | No | Partial match on industry |
| `billing_city` | string | No | Partial match on city |
| `billing_state` | string | No | Partial match on state |
| `limit` | integer | No | Max results (default 50) |

### Get a single account

**Tool**: `SALESFORCE_GET_ACCOUNT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | **Yes** | Salesforce Account ID (prefix `001`) |

### Update an account

**Tool**: `SALESFORCE_UPDATE_ACCOUNT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | **Yes** | Salesforce Account ID (prefix `001`) |
| `Name` | string | No | Updated account name |
| `Phone` | string | No | Updated phone |
| `Website` | string | No | Updated website |
| `Industry` | string | No | Updated industry |
| `BillingStreet` | string | No | Updated street |
| `BillingCity` | string | No | Updated city |
| `BillingState` | string | No | Updated state |
| `BillingPostalCode` | string | No | Updated zip |
| `BillingCountry` | string | No | Updated country |
| `Description` | string | No | Updated description |
| `custom_fields` | object | No | Custom field updates |

Only include fields you want to change — omitted fields are left untouched.

### Delete an account

**Tool**: `SALESFORCE_DELETE_ACCOUNT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | **Yes** | Salesforce Account ID (prefix `001`) |

**Warning**: Deleting an account may cascade-delete or orphan related contacts, tasks, and opportunities. Always confirm with the advisor before deleting.

---

## Contacts

### Create a contact

**Tool**: `SALESFORCE_CREATE_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `LastName` | string | **Yes** | Contact's last name |
| `FirstName` | string | No | Contact's first name |
| `AccountId` | string | No | Parent Account ID (prefix `001`) — links the contact to an account |
| `Email` | string | No | Email address |
| `Phone` | string | No | Phone number |
| `MobilePhone` | string | No | Mobile number |
| `Title` | string | No | Job title |
| `Department` | string | No | Department |
| `MailingStreet` | string | No | Street address |
| `MailingCity` | string | No | City |
| `MailingState` | string | No | State/province |
| `MailingPostalCode` | string | No | Zip code |
| `MailingCountry` | string | No | Country |
| `Description` | string | No | Notes |
| `custom_fields` | object | No | Custom field API names to values |

If the advisor mentions an account name, resolve the Account ID first with `SALESFORCE_SEARCH_ACCOUNTS`, then pass it as `AccountId`.

**Example:**
```json
{
  "LastName": "Kumar",
  "FirstName": "Priya",
  "AccountId": "001dL00001xrytEQAQ",
  "Email": "priya.kumar@email.com",
  "Phone": "(650) 555-9876",
  "Title": "Managing Partner"
}
```

### Search contacts

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

**Example — list all contacts for an account:**
```json
{
  "account_name": "Kumar Family Trust"
}
```

### Get a single contact

**Tool**: `SALESFORCE_GET_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | string | **Yes** | Salesforce Contact ID (prefix `003`) |

### Update a contact

**Tool**: `SALESFORCE_UPDATE_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | string | **Yes** | Salesforce Contact ID (prefix `003`) |
| `FirstName` | string | No | Updated first name |
| `LastName` | string | No | Updated last name |
| `Email` | string | No | Updated email |
| `Phone` | string | No | Updated phone |
| `MobilePhone` | string | No | Updated mobile |
| `Title` | string | No | Updated job title |
| `Department` | string | No | Updated department |
| `MailingStreet` | string | No | Updated street |
| `MailingCity` | string | No | Updated city |
| `MailingState` | string | No | Updated state |
| `MailingPostalCode` | string | No | Updated zip |
| `MailingCountry` | string | No | Updated country |
| `Description` | string | No | Updated notes |
| `custom_fields` | object | No | Custom field updates |

Only include fields you want to change. If the advisor refers to a contact by name, resolve the ID first with `SALESFORCE_SEARCH_CONTACTS`.

### Delete a contact

**Tool**: `SALESFORCE_DELETE_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | string | **Yes** | Salesforce Contact ID (prefix `003`) |

Always confirm with the advisor before deleting a contact.

---

## Leads

### Create a lead

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
| `State` | string | No | State/province |
| `PostalCode` | string | No | Zip code |
| `Country` | string | No | Country |
| `Website` | string | No | Company website |
| `AnnualRevenue` | number | No | Estimated annual revenue |
| `NumberOfEmployees` | integer | No | Employee count |
| `CustomFields` | object | No | Custom field API names to values |
| `allow_duplicates` | boolean | No | Default `false`. Set `true` to bypass duplicate detection |

Always check for duplicates first with `SALESFORCE_SEARCH_LEADS` using `email` or `name`. If a match exists, inform the advisor and offer to update instead.

**Example:**
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

**Response**: New Lead ID at `response.data.id`.

### Search leads

**Tool**: `SALESFORCE_SEARCH_LEADS`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | No | Partial match on name |
| `email` | string | No | Partial match on email |
| `company` | string | No | Partial match on company |
| `status` | string | No | Lead status filter |
| `limit` | integer | No | Max results |

### Get a single lead

**Tool**: `SALESFORCE_GET_LEAD`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lead_id` | string | **Yes** | Salesforce Lead ID (prefix `00Q`) |

### Update a lead

**Tool**: `SALESFORCE_UPDATE_LEAD`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lead_id` | string | **Yes** | Salesforce Lead ID (prefix `00Q`) |

Accepts the same fields as create (except `allow_duplicates`). Only include fields you want to change.

### Delete a lead

**Tool**: `SALESFORCE_DELETE_LEAD`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lead_id` | string | **Yes** | Salesforce Lead ID (prefix `00Q`) |

Always confirm with the advisor before deleting.

---

## Tasks

### Create a task

**Tool**: `SALESFORCE_CREATE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `subject` | string | **Yes** | Task title (e.g. "Follow up with client about Q2 review") |
| `status` | string | No | Default: "Not Started". Options: "Not Started", "In Progress", "Completed", "Waiting on someone else", "Deferred" |
| `priority` | string | No | Default: "Normal". Options: "High", "Normal", "Low" |
| `activity_date` | string | No | Due date in YYYY-MM-DD format |
| `description` | string | No | Detailed notes |
| `what_id` | string | No | Related Account (`001`) or Opportunity (`006`) ID |
| `who_id` | string | No | Related Contact (`003`) or Lead (`00Q`) ID |
| `owner_id` | string | No | User ID of task owner. Defaults to current user if omitted |
| `is_reminder_set` | boolean | No | Whether to set a reminder |
| `reminder_date_time` | string | No | ISO datetime for reminder |
| `custom_fields` | object | No | Custom field API names to values |

If the advisor mentions an account name, resolve the Account ID first with `SALESFORCE_SEARCH_ACCOUNTS`, then pass it as `what_id`. This ensures the task appears on the account's activity timeline.

**Example:**
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

### Search tasks

**Tool**: `SALESFORCE_SEARCH_TASKS`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Task status filter |
| `priority` | string | No | Priority filter |
| `is_closed` | boolean | No | `false` for open, `true` for closed |
| `assigned_to_name` | string | No | Partial match on owner name |
| `account_name` | string | No | Partial match on related account |
| `contact_name` | string | No | Partial match on related contact |
| `subject` | string | No | Partial match on task subject |
| `activity_date_from` | string | No | YYYY-MM-DD start date |
| `activity_date_to` | string | No | YYYY-MM-DD end date |
| `limit` | integer | No | Max results (default 50, max 2000) |

**Example — show open tasks:**
```json
{ "is_closed": false, "assigned_to_name": "<advisor's name>" }
```

### Update a task

**Tool**: `SALESFORCE_UPDATE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | **Yes** | Salesforce Task ID |
| `subject` | string | No | Updated subject |
| `status` | string | No | New status value |
| `priority` | string | No | New priority value |
| `activity_date` | string | No | New due date (YYYY-MM-DD) |
| `description` | string | No | Updated description |
| `what_id` | string | No | Updated related record ID |
| `who_id` | string | No | Updated Contact/Lead ID |
| `custom_fields` | object | No | Custom field updates |

If the advisor refers to a task by name, resolve its ID first with `SALESFORCE_SEARCH_TASKS` using `subject`.

### Complete a task

**Tool**: `SALESFORCE_COMPLETE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | **Yes** | Salesforce Task ID |
| `completion_notes` | string | No | Notes appended to description |

Sets Status to "Completed" and IsClosed to true.

### Delete a task

**Tool**: `SALESFORCE_DELETE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | **Yes** | Salesforce Task ID |

Always confirm with the advisor before deleting. If the advisor refers to a task by name, resolve its ID first with `SALESFORCE_SEARCH_TASKS`.

---

## Geographic Client Queries

Use when a Twin Peaks advisor wants to find clients near a location — for event planning, regional outreach, or proximity-based filtering.

### Step 1 — Query accounts by city/state/region

Use `SALESFORCE_RUN_SOQL_QUERY`:

```
query: "SELECT Id, Name, Phone, BillingStreet, BillingCity, BillingState, BillingPostalCode FROM Account WHERE BillingCity = 'Menlo Park' AND BillingState = 'CA'"
```

For broader queries, use `IN` with nearby cities:

```
query: "SELECT Id, Name, Phone, BillingStreet, BillingCity, BillingState, BillingPostalCode FROM Account WHERE BillingState = 'CA' AND BillingCity IN ('Menlo Park', 'Palo Alto', 'Redwood City', 'Mountain View', 'San Mateo', 'Sunnyvale', 'Los Altos', 'Atherton')"
```

### Step 2 — For radius-based queries, calculate distances

When the advisor asks for "within X miles," use the Composio Remote Workbench with the haversine formula to filter by distance after pulling accounts with addresses. Use a geocoding lookup to convert city names to coordinates.

### Common geographic queries

| Advisor says | SOQL approach |
|-----------|--------------|
| "Clients in San Francisco" | `WHERE BillingCity = 'San Francisco'` |
| "Clients in California" | `WHERE BillingState = 'CA'` |
| "Clients within 20 miles of Menlo Park" | Query CA accounts, geocode, haversine filter |
| "Clients on the East Coast" | `WHERE BillingState IN ('NY', 'NJ', 'CT', 'MA', 'PA', ...)` |

### Contacts variant

Query the Contact object for people instead of accounts:

```
query: "SELECT Id, Name, Email, Phone, MailingCity, MailingState, Account.Name FROM Contact WHERE MailingCity = 'Menlo Park' AND MailingState = 'CA'"
```

### Leads variant

```
query: "SELECT Id, Name, Email, Phone, City, State, Company FROM Lead WHERE City = 'Menlo Park' AND State = 'CA' AND IsConverted = false"
```

---

## SOQL Quick Reference

- No `AS` keyword for aliases — use implicit: `SUM(Amount) TotalSales`
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
- **REQUIRED_FIELD_MISSING**: You're missing a required field (e.g. `LastName` + `Company` for leads, `subject` for tasks, `Name` for accounts).
- **MALFORMED_ID**: An ID is not a valid 15/18-char Salesforce ID. Double-check the value.
- **Duplicate rules**: If create fails due to org duplicate rules, search first and decide whether to update instead.
- **Pagination**: If `response.data.done` is `false`, there are more records. Use `nextRecordsUrl` to fetch subsequent pages.

---

## Common Patterns

### Resolve an account name to ID
1. `SALESFORCE_SEARCH_ACCOUNTS` with `name: "<partial name>"`
2. Extract `Id` from first match
3. Pass as `what_id` (tasks), `AccountId` (contacts), etc.

### Resolve a contact by name to ID
1. `SALESFORCE_SEARCH_CONTACTS` with `name: "<partial name>"`
2. Confirm with advisor if multiple matches
3. Pass `Id` as `contact_id`

### Resolve a task by subject to ID
1. `SALESFORCE_SEARCH_TASKS` with `subject: "<partial subject>"`
2. Extract `Id` from matching record
3. Pass as `task_id`

### Check for duplicate leads before creating
1. `SALESFORCE_SEARCH_LEADS` with `email` or `name`
2. If results exist, inform advisor and offer to update instead
3. If no results, proceed with `SALESFORCE_CREATE_LEAD`

### Create an account and immediately add a task to it
1. `SALESFORCE_CREATE_ACCOUNT` with `Name`
2. Extract `Id` from `response.data.id`
3. `SALESFORCE_CREATE_TASK` with `what_id` set to the new account ID
