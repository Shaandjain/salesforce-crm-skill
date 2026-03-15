---
name: tp-salesforce-update-contact
description: |
  Update an existing contact in Salesforce CRM. Use when the user wants to edit contact details like email, phone, address, title, or any other field on a contact record. Triggers on "update contact", "change contact", "edit contact", "update their email", "change phone number for", or similar requests.
---

# Update a Contact

Use when the user wants to modify an existing contact's information — email, phone, address, title, or any other field.

## Step 1 — Find the contact to update

If the user refers to a contact by name (not by ID), resolve the Contact ID first:

1. Call `SALESFORCE_SEARCH_CONTACTS` with `name: "<contact name>"`
2. If multiple results, present the matches and ask the user to confirm which one
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

## Step 2 — Update the contact

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

## Pitfalls

- **MALFORMED_ID**: Ensure the contact ID is a valid 15/18-character Salesforce ID starting with `003`
- **INVALID_FIELD**: A field name doesn't exist on the Contact object. Remove it and retry
- If multiple contacts match the name, always confirm with the user before updating — never guess
