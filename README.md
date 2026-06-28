**Argus Schemas**: Configuration files for [Argus](https://github.com/impact-initiatives/argus) datasets.

# About
[Argus](https://github.com/impact-initiatives/argus) performs validations on different datasets that have different schemas and validation rules. To allow these schemas to be created and managed independetly of Argus they are stored here as `yaml` files. This allows schemas to be created, updated and deployed without having to re-deploy Argus.

# Structure
In order for schemas to properly integrate with Argus files and directories are required to follow a set structure in this repository.
```bash
dataset_type/locale/schema.yaml
dataset_type/locale/validators.yaml
```
All folders and file names should be kept lowercase. 

For example, an English version of the jmmi dataset would contain:
```
jmmi/en/schema.yaml
jmmi/en/validators.yaml
```
`schema.yaml` contains the dataset schema. This includes the sheets, columns and other relevant properties.
`validators.yaml` contains the validation rules relevant for the dataset and their relevant properties.



# Schema
Schema files are mapped back to classes in Argus and so need to contain the same properties. Specifically, [BaseDatasetSchema](https://github.com/impact-initiatives/argus/blob/main/src/argus/models/base_dataset_schemas.py) and its components in [SchemaSheetMap, SchemaColumnMap and ProcessValueMap](https://github.com/impact-initiatives/argus/blob/main/src/argus/models/base.py).

These can be specified in `schema.yaml` directly or `schema.yaml` can import shared common components. 

```yaml
enumerator_performance_sheet:
    standard_name: "enumerator_performance_log"
    alternate_names: []
    required: false
    mandatory_columns: []
```
## Common Components
Schemas often have the same requirements for sheets and columns. To reduce duplicated configuration shared components can be stored in `common/schema_defaults.yaml`. These can then be imported in schema specific files or even used within `schema_defaults.yaml` itself.

For example, within `schema_defaults` `deletion_log` makes use of `uuid_column` via `$use`.
```yaml
deletion_log:
    standard_name: "deletion_log"
    allow_fuzzy_matching: false
    mandatory_columns:
      - $use: uuid_column
      - standard_name: "reason_deletion"
      - standard_name: "enum_id"

uuid_column:
    standard_name: "uuid"
    alternate_names: ["_uuid"]
    is_unique: true
    allow_fuzzy_matching: false
```
`deletion_log` can then be used as part of another schema
```yaml
_imports:
  - "../../common/schema_defaults.yaml"

dataset_type: "JMMI"

schema_loaded_sheets:
    - $use: deletion_log
    ...
```
it can even be modified if required. To change one of the properties:
```yaml
_imports:
  - "../../common/schema_defaults.yaml"

dataset_type: "JMMI"

schema_loaded_sheets:
    - $use: deletion_log
        override:
            alternate_names: ["deleted_log"]
    ...
```
or to add to one of the properties:
```yaml
_imports:
  - "../../common/schema_defaults.yaml"

dataset_type: "JMMI"

schema_loaded_sheets:
    - $use: deletion_log
        $append_mandatory_columns:
            - $use: some_existing_column #default column
            - standard_name: "Some_new_column" #new column
                alternate_names: ["new_column"]
    ...
```
or to add a new sheet not from common
```yaml
_imports:
  - "../../common/schema_defaults.yaml"

dataset_type: "JMMI"

schema_loaded_sheets:
    - some_new_sheet:
        standard_name: "a_new_sheet"
        mandatory_columns: []
# rest of schema...
```
If a rule or property is specified in a file but does not exist then Argus will produce a validation error.

# Validators
Validator files are mapped back to their respective classes in Argus and so need to contain the same class names and parmaters. 

To specifiy a rule as required using default paramaters
```yaml
validators:
    - type: "MissingSheetsCheck"
    - type: "CrossSheetIdCheck"
```
or to change the paramaters based on knowledge of the schema
```yaml
validators:
    - type: "MissingSheetsCheck"
    - type: "CrossSheetIdCheck"
    - type: "CrossSheetIdCheck"
        kwargs:
            master_sheet: "clean_data"
            child_sheets: 
            - "cleaning_log"
```

If a rule or property is specified in a file but does not exist then Argus will produce a validation error.

