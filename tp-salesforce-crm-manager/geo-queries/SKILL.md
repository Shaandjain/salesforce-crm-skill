---
name: tp-salesforce-geo-queries
description: |
  Find Salesforce clients near a location for event planning, regional outreach, or proximity-based filtering. Use when the user asks about "clients near", "clients in", "who lives near", "event invites", "find clients within X miles", "I'm hosting an event in [city]", or any geographic/location-based client query.
---

# Geographic Client Queries

Use when the user wants to find clients near a location — for event planning, regional outreach, or proximity-based filtering. This is particularly useful for scenarios like "I'm hosting an event in Menlo Park, give me all clients within 20 miles."

## Approach: SOQL city/state filtering + distance calculation

Salesforce standard objects store addresses as text fields (BillingCity, BillingState, etc.), not as geolocation coordinates. The strategy is to query accounts by region, then refine by distance if needed.

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

When the user asks for "within X miles," after pulling accounts with addresses, use the Composio Remote Workbench to calculate distances:

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

Format results as a clean list showing: client name, city, estimated distance, and phone number. Group by proximity (closest first). If the user mentions an event, suggest composing an outreach list.

## Common geographic queries

| User says | SOQL approach |
|-----------|--------------|
| "Clients in San Francisco" | `WHERE BillingCity = 'San Francisco'` |
| "Clients in the Bay Area" | `WHERE BillingCity IN ('San Francisco', 'Oakland', 'San Jose', 'Palo Alto', ...)` |
| "Clients in California" | `WHERE BillingState = 'CA'` |
| "Clients within 20 miles of Menlo Park" | Query CA accounts, geocode, haversine filter |
| "Clients on the East Coast" | `WHERE BillingState IN ('NY', 'NJ', 'CT', 'MA', 'PA', ...)` |

## Contacts variant

If the user asks about contacts (people) rather than accounts (companies), query the Contact object instead:

```
query: "SELECT Id, Name, Email, Phone, MailingCity, MailingState, Account.Name FROM Contact WHERE MailingCity = 'Menlo Park' AND MailingState = 'CA'"
```

## Leads variant

For leads not yet converted to accounts:

```
query: "SELECT Id, Name, Email, Phone, City, State, Company FROM Lead WHERE City = 'Menlo Park' AND State = 'CA' AND IsConverted = false"
```
