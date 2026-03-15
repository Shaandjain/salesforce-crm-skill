---
name: tp-salesforce-find-contacts
description: |
  Search and list contacts in Salesforce CRM. Use when the user wants to find contacts, look up a person, list contacts for an account, or search by name/email/phone. Triggers on "find contact", "search contacts", "list contacts", "who is", "look up contact", "contacts for [account]", or similar requests.
---

# Find Contacts

Use when the user wants to search for contacts, look up a specific person, or list contacts associated with an account.

## Step 1 — Search for contacts

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

## Step 2 — Get full details for a specific contact

If the user needs complete details on a single contact, use the contact ID from search results:

**Tool**: `SALESFORCE_GET_CONTACT`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | string | **Yes** | Salesforce Contact ID (prefix `003`) |

## Alternative: Custom SOQL for advanced queries

**Tool**: `SALESFORCE_RUN_SOQL_QUERY`

```
query: "SELECT Id, Name, Email, Phone, Title, Account.Name, MailingCity, MailingState FROM Contact WHERE Account.Name LIKE '%Kumar%' ORDER BY LastName ASC"
```

**Example — contacts without email addresses:**
```
query: "SELECT Id, Name, Phone, Account.Name FROM Contact WHERE Email = null LIMIT 200"
```

## Present results clearly

Format results as a clean table or list showing: name, email, phone, title, and associated account. If the user is looking for a specific person, highlight the best match and confirm before proceeding with any follow-up action.
