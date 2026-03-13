---
name: tp-salesforce-create-task
description: |
  Create a new task in Salesforce CRM. Use when the user wants to create a follow-up, reminder, or action item tied to a client or lead. Triggers on "create a task", "add a task", "remind me to", "follow up with", or similar requests.
---

# Create a Task

Use when the user wants to create a follow-up, reminder, or action item tied to a client or lead.

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

## Before creating a task

If the user mentions a client or account name (not an ID), resolve the Account ID first:

1. Call `SALESFORCE_SEARCH_ACCOUNTS` with `name: "<account name>"`
2. Extract the `Id` from the first matching record
3. Pass that `Id` as `what_id`

This ensures the task appears on the account's activity timeline. Omitting `what_id` creates a standalone task that won't be associated with any record.

## Example — create a follow-up task

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
