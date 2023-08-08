# folio_user_loading

## Purpose

An implementation of the [folio_migration_tools](https://github.com/FOLIO-FSE/folio_migration_tools) that focuses specifically on loading users. Example source data and transformed data can be found in the iterations folder. Examples of the necessary files can be found in [the examples folder](examples).

## Requirements

- [Python >= 3.9](https://www.python.org/downloads/)
- [Poetry](https://python-poetry.org/docs/)

## Setup

- `poetry install`
- `poetry shell`

## Workflow Outline

1. Map Data
   - Patron groups, user records
2. Transform Data
   - Turning delimited data into JSON that the FOLIO API can understand
3. Load Data
   - Loading the user records into FOLIO

## Map Data

The folio migration tools use maps for transforming user records into FOLIO-ready data. 

### User Groups

The FOLIO patron groups **must already be configured** before transforming user records

`mapping_files/user_groups.tsv` maps patron groups as represented in source data to patron groups configured in FOLIO. The source data column should have the same header as the source TSV file. 

The map requires a wildcard row for any source values that are not mapped. The wildcard row in this example is

| Patron Group | folio_group |
|--------------|-------------|
| *            | Other       |

*Example:*<br>

<img width="387" alt="image" src="https://github.com/kevinpowell1/folio_user_loading/assets/66270317/326f4969-cc7d-4151-87b9-f1619ffa3644">

### Field-to-Field Mapping

`mapping_files/user_mapping.json` takes care of field-to-field mapping.

```json
{
  "folio_field": "barcode",
  "legacy_field": "Patron ID",
  "value": "",
  "description": ""
}
```

You can also hardcode values for a field

```json
{
  "folio_field": "personal.addresses[0].primaryAddress",
  "legacy_field": "",
  "value": true,
  "description": ""
}
```

Columns in your original CSV data can be mapped to one or more FOLIO fields

```json
[
  {
    "folio_field": "externalSystemId",
    "legacy_field": "Email",
    "value": "",
    "description": ""
  },
  {
    "folio_field": "personal.email",
    "legacy_field": "Email",
    "value": "",
    "description": ""
  }
]
```
