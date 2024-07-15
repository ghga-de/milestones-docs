# State Management Service (Monkfish)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

## Scope
### Outline:
The [Archive Test Bed](https://github.com/ghga-de/archive-test-bed) needs a way to
consistently perform arbitrary CRUD operations
in MongoDB that are outside the scope of the component services, but also unwieldy
to manage in piecemeal code adaptations. A dedicated service can offer a simple
solution to this problem and allow test bed services and processes to programmatically
seed, reset, modify and examine databases through a single API.

### Included/Required:
The implementation of the SMS should include a token-secured RESTful API that interacts
with MongoDB.


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
  - Auth header: internal token
  - Response body: list of resources in the collection matching the specified criteria
  - Response status: 
    - `200 OK`: Request successfully processed
    - `401 Unauthorized`: auth error (not authenticated)
- `PUT /{db-name}/{collection-name}/{id}`
  - *Upserts the resource with the given ID using the payload provided in the request body.*
  - Request body:
    - Resource to upsert
  - Auth header: internal token
  - Response body:
    - `id`: string (the ID of the upserted resource)
  - Response status:
    - `201 Created`: resource successfully upserted
    - `401 Unauthorized`: auth error (not authenticated)
- `DELETE /{db-name}/{collection-name}`
  - *Deletes the specified collection.*
  - Query string (optional):
    - Filter parameters to refine deletion, e.g. `user_id=123`
  - Auth header: internal token
  - Response body: list of deleted IDs
  - Response status: 
    - `200 OK`: Request successfully processed
    - `401 Unauthorized`: auth error (not authenticated)


## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
