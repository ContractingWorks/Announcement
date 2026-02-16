# Contracting Works Announcements

> Updates on model and API changes.

## 2026-02-13: Breaking change on date fields
We have been using the DateTime data type for handling dates in the API so far. This has several disadvantages, in particular handling time zone adjustments causes a lot of workarounds in integration code.
When working with the change on Service time registration, we decided to switch the data type from DateTime to DateOnly in our APIs for date fields.

This means: For upsert operations, date fields no longer accept a time or time zone component, and must follow the format YYYY-MM-DD
Similarly, when reading date fields through GraphQL, they will be returned on the format YYYY-MM-DD

Two date fields on WagePeriod incorrectly have the suffix UTC, and will be renamed as part of this change: StartDateUTC to StartDate and EndDateUTC to EndDate.

**Update 2026-02-16**: StartDate and EndDate on Project also incorrectly has the UTC suffix, and will be renamed as part of the release.

This change will be deployed to our ExTest environment early next week, probably Tuesday morning. Please test your integrations to ensure they remain functional after the change.

## 2026-02-12: Breaking change on Service time registration
The fields StartDateTimeUTC and EndDateTimeUTC have been used for storing local time zone data, often without the time component in practice. The fields will be replaced with three new fields representing this more clearly:
ServiceDate, ServiceTimeStart and ServiceTimeEnd.

**Update 2026-02-13**: The field Calc_StartDateLocal is now redundant on Service, and will be removed. Please use the ServiceDate field instead.

EndDateTimeUTC has never been used in our frontend or integrations, and currently contain NULL values only in practice.
The old fields will be removed in the beginning of March 2026.


## 2025-12-29: Switching SystemMessage and IntegrationStatus to use Enum_EntityID instead of TableName
Row state messages are generally handled through SystemMessage and IntegrationStatus. Today, this should only be in use by the CW backend and CW internal integrations, so the update will be deployed once the changes pass internal testing. 
The purpose of the change is to improve performance and ensure data quality on system messages.

In addition, we are preparing for 3rd party intgrations to be able to use our integration status message system, and we have exposed a cleanup method for use with all integrations (internal and external) to reset old integration status messages.
For now, please contact the CW dev team if you wish to use the integration status messages for your integration.

The change will be released to production this week or the first week of 2026.

## 2025-09-26: Removing Sys_RowState from the write API
The use of Sys_Rowstate has evolved since our first versions of the API, and this field is currently not intended to be set by integrations, or indeed reacted to in any way by integrations.
It is used internally in CW to manage showing errors for end-users in the user interface. It can still be read by the GraphQL endpoint, but will no longer be returned after an upsert. 
Similarly, the field will not be part of webhook events.

The change will be released to production in week 41 (next week).

## 2025-08-22: Removed unused endpoints
We are removing the *GetNotSynced* endpoints from most entities in the ClientAPI, as they were not in use by our integrations.
We are not aware of any external parties using the endpoints, they will therefore be removed as part of a regular release this month.
Please let us know if you rely on the endpoints for anything!

## 2025-08-15: Stricter write authorization

### Upcoming API Change
We are tightening write-authorization checks. After the next release, write operations require explicit permission at the table and column level. Requests that previously succeeded due to permissive checks may now fail.

### Impacts of the Change
1. Increased authorization errors for write requests when the authenticated user lacks permission for the target table or any included column.
2. Read operations are unaffected.

Example error:
```
Write authorization failed for user 'integration-sc@contracting.works' on table 'Storage'
```
You may also see errors that include a specific column name.

### Recommended Action
- Review the permissions/roles of users that perform writes.
- Ensure write permission on every table and column included in your payloads.
- If errors occur, remove unauthorized fields from the payload or request appropriate permissions.
- Contact support and include the failing user, table, and (if applicable) column from the error.

### Notes
- Retries will not resolve authorization failures; a permission update or payload change is required.
