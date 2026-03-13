---
name: tp-salesforce-query-tasks
description: |
  Query, update, and complete tasks in Salesforce CRM. Use when the user wants to see their open tasks, check what's due, update task status, or mark tasks complete. Triggers on "my tasks", "what's on my plate", "show me my open tasks", "update task status", "mark task complete", "what's due", or similar requests.
---

# Query Task Status

Use when the user wants to see their open tasks, check what's due, or look up the status of specific tasks.

## Option A: Search with filters (recommended for most queries)

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
  "assigned_to_name": "<user's name>"
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

## Option B: Custom SOQL (for advanced queries)

**Tool**: `SALESFORCE_RUN_SOQL_QUERY`

```
query: "SELECT Id, Subject, Status, Priority, ActivityDate, What.Name, Who.Name FROM Task WHERE OwnerId = '<user_id>' AND IsClosed = false ORDER BY ActivityDate ASC"
```

## Updating task status

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

If the user refers to a task by name, resolve its ID first with `SALESFORCE_SEARCH_TASKS` using `subject: "<partial subject>"`.

## Marking a task complete

**Tool**: `SALESFORCE_COMPLETE_TASK`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `task_id` | string | **Yes** | Salesforce Task ID |
| `completion_notes` | string | No | Notes appended to description |

This convenience wrapper sets Status to "Completed" and IsClosed to true.
