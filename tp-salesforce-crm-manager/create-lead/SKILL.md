---
name: tp-salesforce-create-lead
description: |
  Create a new lead in Salesforce CRM. Use when the user wants to add a new prospective client or contact into the pipeline. Triggers on "create a lead", "add a lead", "new lead", "add prospect", or similar requests.
---

# Create a Lead

Use when the user wants to add a new prospective client or contact into the pipeline.

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

## Before creating a lead

Always check for duplicates first using `SALESFORCE_SEARCH_LEADS`:

```json
{ "email": "prospect@example.com" }
```

or by name:

```json
{ "name": "Jane Doe", "company": "Acme Corp" }
```

If a match is found, inform the user and offer to update the existing lead instead of creating a duplicate.

## Example — create a new lead

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

**Response parsing**: On success, the new Lead ID is at `response.data.id`. The response may be minimal (just id/success/errors) — use `SALESFORCE_GET_LEAD` with the returned ID if you need to confirm field persistence.

## Pitfalls

- 400 validation errors mean org-required fields are missing or invalid. Check `data.errors` for specifics
- If `allow_duplicates` is `false` (default) and duplicates exist, creation will fail — search first
