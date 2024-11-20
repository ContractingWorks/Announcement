# Contracting Works Announcements

> Updates on model and API changes.

## 2024-11-20: Model Update
> No implementation date set yet.

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
