# Contracting Works Announcements

> Updates on model and API changes.

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


## 2024-11-25: Further Model Update to Address

### Release Dates
Support for `Address.PostalNumber` is planned removed:

| Environment | Date       |
|-------------|------------|
| Extest      | 2025-01-05 |
| Prod        | 2025-01-07 |

Note: Because we see the PostalNumber field is still in use from time to time, we have postponed this update
until january 2025.

### Upcoming Model Change:
`Address.PostalNumber` (string) has been replaced by `Address.PostalCode` (string).
The old field `Address.PostalNumber` is retained for backwards compatibility,
and is currently kept in sync with the new field.

This is temporary, and the old `Address.PostalNumber` field should no longer be used.
Please switch your references from PostalNumber to PostalCode as soon as possible.

This change affects both the Contracting Works GraphQL and REST APIs.

### Impacts of the Change:
1. **GraphQL API**: Queries for `postalNumber` will result in errors.
2. **REST API**: Updates to `PostalNumber` will be ignored, as the field will no longer exist in the model.

### Recommended Action:
Update your code to use `PostalCode` instead of `PostalNumber` immediately.
We would appreciate it if you let us know through Contracting.Works Teams when the change is done.



## 2024-11-20: Model Update to Address
### Release Dates
These are our planned release dates for this model update:

| Environment | Date       |
|-------------|------------|
| Extest      | 2024-11-21 |
| Prod        | 2024-11-27 |

### Upcoming Model Change:
1. `Address.MunicipalityNumber` (int) will be replaced by `Address.MunicipalityCode` (string).

This change affects both the Contracting Works GraphQL and REST APIs.

### Impacts of the Change:
1. **GraphQL API**: Queries for `municipalityNumber` will result in errors.
2. **REST API**: Updates to `municipalityNumber` will be ignored, as the field will no longer exist in the model.

### Recommended Action:
Update your code to use `municipalityCode` instead of `municipalityNumber`.
> **Note:** The field type is changing from `int` to `string`.

We recommend testing your integration thoroughly to ensure compatibility once the change is implemented.
