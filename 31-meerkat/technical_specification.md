# GHGA Validator (Meerkat)
**Epic Type:** Implementation Epic

## Scope:

The goal is to provide a CLI Tool for validation of the (meta)data related to submissions.

## Implementation Details:

GHGA Validator can be run as a separate tool by GHGA team or by the data submitters as a preliminary step before submitting data to GHGA. Alternatively it can be also included as a module in other microservices.

Validation includes the following types of checks:

- JSON schema validation of the input file - the JSON structure and the presence of required fields
- Non-inlined references - the objects which are referenced must exist in the same input file

As the base for validation the existent LinkML Validator (https://linkml.io/linkml-validator/) can be taken. It includes the implementation of the JSON schema validation and needs to be extended with the new pluggin for non-inlined references.

If the validation fails, the parseable reports in JSON format are generated.

### CLI

```
ghga-validator.py
    --schema <url_to_metadata_schema_in_YAML_format> \
    --input <url_to_JSON_file_to_be_validated> \
    --output <url_to_validation_report>
```

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
