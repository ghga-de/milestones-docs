# Outbox Pattern Refactoring (Hero Shrew)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).


## Scope
### Outline:
The outbox pattern must be applied to key areas of our microservice system, but not
everywhere. The services that generate the first event in a request flow should be fitted
with outbox publishers, and the corresponding consumers of such events should be fitted
with outbox subscribers. By doing this, only the initial events need to be backed up. A
republishing and reprocessing of these events should then result in the re-creation of 
all transitive events, as far as idempotence allows.

### Included/Required:
- Implementation of the outbox pattern in the following:
  - Download Controller Service
  - Upload Controller Service
  - Purge Controller Service
- Any modifications to other services for the purpose of achieving idempotence
- Testing


## Additional Implementation Details:

The following services need the outbox *publisher* implemented for the listed events:
- Upload Controller: 
  - FileUploadReceived (consumed by the IRS)
- Download Controller:
  - NonStagedFileRequested (consumed by the IFRS)
- Purge Controller:
  - FileDeletionRequested (consumed by the IFRS, UCS, and DCS)

The following services need the outbox *subscriber* implemented for the listed events:
- Interrogation Room:
  - FileUploadReceived
- Internal File Registry
  - NonStagedFileRequested
  - FileDeletionRequested
- Download Controller
  - FileDeletionRequested
- Upload Controller
  - FileDeletionRequested


## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 2
