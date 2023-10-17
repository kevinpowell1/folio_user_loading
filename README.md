# folio_user_loading

## Purpose

An implementation of the [folio_migration_tools](https://github.com/FOLIO-FSE/folio_migration_tools) that focuses specifically on loading users. Example source data and transformed data are in the iterations folder. Examples of the necessary files are in [the examples folder](examples).

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

## 1. Map Data

The folio migration tools use metadata maps for transforming user records into FOLIO-ready data.

### User Groups

`mapping_files/user_groups.tsv`

The FOLIO patron groups **MUST ALREADY BE CONFIGURED IN FOLIO** before transforming user records

This file maps patron groups as represented in source data to patron groups configured in FOLIO. The source data column should have the same header as the source TSV file.

**Example:** <br>

<img width="387" alt="image" src="https://github.com/kevinpowell1/folio_user_loading/assets/66270317/326f4969-cc7d-4151-87b9-f1619ffa3644">

The map requires a wildcard row for any source values that are not mapped. The wildcard row in this example is

| Patron Group | folio_group |
| ------------ | ----------- |
| \*           | Other       |

### Field-to-Field Mapping

`mapping_files/user_mapping.json`

This file takes care of field-to-field mapping

In this example, the source field `Patron ID` is mapped to the FOLIO field `barcode`.

```json
{
  "folio_field": "barcode",
  "legacy_field": "Patron ID",
  "value": "",
  "description": ""
}
```

This file can also hard-code values in user records. In this example, the FOLIO field `personal.addresses[0].primaryAddress` will be hardcoded as `true`.

```json
{
  "folio_field": "personal.addresses[0].primaryAddress",
  "legacy_field": "",
  "value": true,
  "description": ""
}
```

TIP: Columns in your original CSV data can be mapped to one or more FOLIO fields

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

## 2. Transform Data

Open `configuration/configuration.json`

### Fill out Library Configuration

```json
  {
    "libraryInformation": {
      "tenantId": "YOUR TENANT ID",
      "multiFieldDelimiter": "<^>",
      "okapiUrl": "https://okapi-YOURDOMAIN.folio.ebsco.com",
      "okapiUsername": "YOUR_API_USER",
      "logLevelDebug": false,
      "libraryName": "YOUR LIBRARY NAME",
      "folioRelease": "orchid",
      "addTimeStampToFileNames": false,
      "iterationIdentifier": "user_loading"
    }
}
```

### Configure Task

- Place source_data in `iterations/user_loading/source_data/users`.
- Edit `file_name`

```json
{
  "name": "transform_users",
  "migrationTaskType": "UserTransformer",
  "userMappingFileName": "user_mapping.json",
  "groupMapPath": "user_groups.tsv",
  "useGroupMap": true,
  "userFile": {
    "file_name": "YOUR_SOURCE_DATA.TSV"
  }
}
```

### Run User Transform

From the `folio_user_loading` directory...

`python -m folio_migration_tools --base_folder_path . configuration/configuration.json user_transform`

When prompted for the okapi password, enter the password for your `okapiUsername`. The username / password can be any account in FOLIO that has administrative privileges.

- Reports on the tranform are in `iterations/user_loading/reports`
- The transformed data is in `iterations/user_loading/results`

## 3. Load Data

### Configure Task(s)

- Edit `file_name` if needed. Be sure the file is in `iterations/user_loading/results`. 

```json
{
  "name": "post_users",
  "migrationTaskType": "BatchPoster",
  "objectType": "Users",
  "batchSize": 250,
  "files": [
    {
      "file_name": "folio_users_transform_users.json"
    }
  ]
}
```

**Sometimes** the user transform generates "extradata". This often contains mapped note fields.

Extradata is in `iterations/user_loading/results`

```json
{
    "name": "post_extradata_users",
    "migrationTaskType": "BatchPoster",
    "objectType": "Extradata",
    "batchSize": 250,
    "files": [
        {
            "file_name": "extradata_transform_users.extradata"
        }
    ]
}
```

### Load Users

From the `folio_user_loading` directory...

`python -m folio_migration_tools --base_folder_path . configuration/configuration.json post_users`

When prompted for the okapi password, enter the password for your `okapiUsername`. The username / password can be any account in FOLIO that has administrative privileges.

- Reports on the user load are in `iterations/user_loading/reports`

### Load Extradata

From the `folio_user_loading` directory...

`python -m folio_migration_tools --base_folder_path . configuration/configuration.json post_extradata_users`


