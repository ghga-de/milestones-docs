# State Management Service (Monkfish)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

## Scope
### Outline:
The [Archive Test Bed](https://github.com/ghga-de/archive-test-bed) needs a way to
consistently perform arbitrary CRUD operations
in MongoDB that are outside the scope of the component services, but also unwieldy
to manage in piecemeal code adaptations. Namely, the needs are:
- setting a predefined state before running the tests (empty or prepopulate databases)
- examining the state after or in between tests (whitebox testing)

Right now, there's not a clean way to achieve either. 
A dedicated service can offer a simple solution to these problems and allow test bed 
processes to programmatically seed, reset, modify and examine databases through a single API.

This service should **never** be deployed to production. It is *only* intended for use
in the Testing and Staging environments, where there is no access to real data.  
Despite this, there should be a way to restrict which databases and collections can be
accessed with this service through configuration, and a simple API key (set in config)
that can be used to authenticate requests.

### Included/Required:
The implementation of the SMS should include a token-secured RESTful API that interacts
with MongoDB. There should be configuration to control access to databases and collections,
as well as an API Key for authentication.

Access control for database/collection resources should occur on a whitelist basis, with
further specification for the individual permissions granted at the collection level
for create, read, update, delete. `None` should be used as a wildcard, equivalent to
'allow all'. For example, specifying a collection but not the permissions enables all
permissions. Specifying a database but no collections within it will enable all
operations on all collections within that database. However, not specifying any databases
at all will *disable* all operations.


### Not included:
Object schema validation would be complex to add because the models can come from a
variety of sources (`ghga-event-schemas`, service-specific models, etc.), and keeping
this service in sync with those definitions, either through config or code, would
incur a cost outweighing the benefits provided by the service.


## API Definitions:

The following REST endpoints will be created:

- `GET /{db-name}/{collection-name}`
  - *Returns all or some documents from the collection.*
  - Query string (optional):
    - Filter parameters to refine results, e.g. `user_id=123&role=supervisor` 
  - Authorization header: API Key (set in configuration)
  - Response body: list of resources in the collection matching the specified criteria
  - Response status: 
    - `200 OK`: Request successfully processed, results in response body
    - `401 Unauthorized`: auth error (not authenticated)
- `PUT /{db-name}/{collection-name}`
  - *Upserts the document(s) provided in the request body in the specified collection.*
  - Request body:
    - Single document to upsert, or array of documents for batch upsertion
  - Authorization header: API Key (set in configuration)
  - Response body: Empty
  - Response status:
    - `204 No Content`: Document(s) successfully upserted
    - `401 Unauthorized`: auth error (not authenticated)
- `DELETE /{db-name}/{collection-name}`
  - *Deletes all or some documents in the collection.*
  - Query string (optional):
    - Filter parameters to refine deletion, e.g. `user_id=123&name=keith`
  - Authorization header: API Key (set in configuration)
  - Response body: Empty
  - Response status: 
    - `204 No Content`: Document(s) did not exist or were successfully deleted
    - `401 Unauthorized`: auth error (not authenticated)


## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
