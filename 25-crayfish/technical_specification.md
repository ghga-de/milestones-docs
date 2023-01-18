# Metadata refactoring proof of concept (\<Epic Code Name\>)
**Epic Type:** Implementation Epic

**Attention: Please do not put any confidential content here.**

## Scope:
The aim of this study is to create a proof of concept implementation of the strategy
outlined in
https://docs.ghga-dev.de/main/architecture_concepts/ac002_metadata_lifecycle.html.


## Additional Implementation Details:

- A implementation that can be run by data stewards locally,
  no production-ready and independently deployed services.
- includes:
    - basic in implementation of the submission store
    - Validation of metadata upon submission against the linkML schema
    - Essential transformations and generation of associated artifacts, including:
        - Fully embedded datasets
        - Custom embeddings
        - Non-restricted (public) metadata
        - Summary statistics
        - Mongodb-based search index
    - REST APIs for querying the above artifacts
- not includes:
    - difference between published and non published submissions
    - reviews of submissions
    - status changes for submissions (all submissions will be immediately be treated
      as completed and published)
    - Deprecation and emptying of submissions

## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: 1
