# Integration of all File Services - Download Stream (Common Raven)
**Epic Type:** Implementation Epic

The goal for this epic is to integrate all Backend File Services into the Testbed developed in [*19 - Pied Raven*](../19-pied-raven/technical_specification.md). This includes updating dependency versions in all services as well as ironing out possible differences in API usage and event processing to include the complete file download path.

## Scope:
A scope definition can be found here:

## User Journeys

This epic covers the following user journeys:

### Integration of File Services (Upload Journey)

Produce a script to test the download path in the testbed and adapt the DCS, EKSS and IFRS to allow for a fully functionioning download path.

## RESTful API changes

The new OpenAPI doc can be found here:
[OpenAPI YAML](./api_definitions/dcs.yaml) - [Swagger UI](https://editor.swagger.io/?url=https://raw.githubusercontent.com/ghga-de/epic-docs/main/24-common-raven/api_definitions/dcs.yaml)

## Additional Implementation Details:


### DCS
- Adapt to request envelope from EKSS and serve the assembled file using download ranges

### IFRS
- The IFRS needs to consume the *NonStagedFileRequested* event, stage file to outbox and emit the *FileStagedForDownload* event afterwards
### Testbed:

- Add a test script for the download path
    1. Run upload path script to populate permanent storage and IFRS state
    2. Request a file download through the ghga-connector
    3. Check for download staging events happening
    4. Verify the downloaded file corresponds to the uploaded one


## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 2
