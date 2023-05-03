# Download Request Management (Green Wrasse)

**Epic Type:** Implementation Epic

Epic planning and implementation follows the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

## Scope

### Outline

This epic covers the implementation of functionality for managing access to datasets.

### Included/Required

The epic includes the development of a backend service "Access Request Service" (ARS)
that stores access requests by users and allows changing their state by data stewards,
and the frontend functionality in the Data Portal UI for users and data stewards.

The UI currently provides a button "Request access" that opens a form that allows
sending a request to a DAC team. This form should be changed so that instead the
request is stored in the backend database. The backend should then inform the data
steward and the user via email that a new request is pending.

The UI should provide data stewards with functionality to list all requests grouped
by state (pending, denied, accepted) and modifying the state of the existing requests.
When access has been granted, it should be also possible to enter an expiration date.
When the status of a request was changed, the backend should inform the user via email
about the status change.

### Optional

Optionally, the UI should also provide users with functionality to list their requests
with current state.

### Not included

Currently, the access management only covers download access. However, the same service
could be later extended to cover upload access management as well.

This epic does not include functionality for managing access directly by the DAC
(data access committee) or to support the decision making and communication between
the involved parties (DAC, RDC, DR). It assumes that a GHGA data steward is acting
as a broker and drives the process manually.

## User Journeys

Actors: Data Requester (DR), Data Steward (DS).

When data is requested:

- DR finds and views a dataset in the data portal
- DR clicks on "Request access" button for a dataset
- If user is not yet logged in via LS Login, user is required to do so
- Form is displayed with info on required information from requester
- DR fills required fields
- TODO: specify fields (probably only one free text field for now)
- DR clicks on "Submit request"
- Request is stored in the backend database
- Notification email is sent out to DS
- Confirmation email is sent out to DR
- Confirmation email is sent out to DR

When access was granted or denied:

- DS logs in to data portal
- DS views list of access requests
- DS finds access request of DR
- DS changes the status from "pending" to either
  - "allowed": access permitted, setting an expiration date
  - "denied": access denied
- An email is sent to the DR informing about the status change
- DR logs in to data portal
- DR visits the download or profile page
- DR should see the newly accessible dataset
- DR can now create a work package for download.
Creating work packages is covered by
[epic 26](https://github.com/ghga-de/epic-docs/blob/main/26-dracula-ant/technical_specification.md).

## API Definitions

The Access Request Service REST API should have the following endpoints:

Used by the web frontend to create and view access requests:

- `POST /access-requests`
  - auth header: internal access token
  - request body:
    - `user_id`: string (the registered user ID of the requester)
    - `dataset_id`: string (the ID of the requested dataset)
    - `word_type`: enum (download or upload, must be download for now)
    - `request_info`: string (the text submitted with the request)
    - `request_start_date`: date (optional, when access should start)
    - `request_end_date`: date (optional, when access should end)

TODO: Discuss exact fields in the request body.

The response body should include the ID of the newly created access request.

Used by the web frontend to fetch all existing requests:

- `GET /access-requests`
  - auth header: internal access token
  - parameters (optional):
    - `user_id`: string (a registered user ID)
    - `state`: an access request state
    - `work_type`: enum (download or upload)
    - `limit`: integer (maximum number of datasets returned)

This endpoint gets the existing requests, filtered and limited using the
specified parameters, and ordered by descending creation date.
If the user specified in the access token does not have a data steward role,
the `user_id` parameter is assumed to be the ID of that user,
and an authorization error is returned if a different `user_id` is passed.

The response body should be an array of access request detail objects
(see below).

Used by the web frontend to get the details for one existing request:

- `GET /access-requests/{access_request_id}`
  - auth header: internal access token

Gets the details of the access request with the given ID.
If the user specified in the access token does not have a data steward role
and the access request has not been made by that user, an authorization error
is returned.

Used by the web frontend to modify the status of an existing request:

- `PATCH /access-requests/{access_request_id}`
  - auth header: internal access token
  - request body:
    - `status`: enum (allowed/denied/pending)
    - `access_starts`: data (when the access permission should start)
    - `access_ends`: date (when the access permission should end)

This endpoint should return and authorization error if the user specified
in the access token does not have the data steward role.

It sets the status of the specified access request to the given state and
also notes in the database the user ID of the data steward and the date
when the change was made. If the status is set to "allowed", then the two
date fields must be also provided, otherwise they must not be provided.

## Additional Implementation Details

### Access Request Object

The access requests are stored in the database with the following details:

- `id`: string (the auto generated ID of this object)
- `user_id`: string (the registered user ID of the requester)
- `dataset_id`: string (the ID of the requested dataset)
- `word_type`: enum (download or upload, must be download for now)
- `request_info`: string (the text submitted with the request)
- `request_start_date`: date (optional, when access should start)
- `request_end_date`: date (optional, when access should end)
- `request_created`: date (when the request has been created)
- `status`: enum (allowed/denied/pending)
- `access_starts`: data (when the access permission should start)
- `access_expires`: date (when the access permission should end)
- `status_changed`: date (when the status was last changed)
- `changed_by`: string (user ID of the data steward)

The `status` field should be automatically set to "pending" upon creation,
and the `request_created` field should be filled with the current date.

### Communication with the Claims Repository

When changes to the status of an access request have been made, these need to
be communicated to the Claims Repository which is the source of truth for all
controlled access grants in GHGA.

To allow the Access Request Service to send changed permissions to the
Claims Repository, the latter provides the following *internal* endpoint:

- `POST /download-access/users/{user_id}/datasets/{dataset_id}`
  - authorization: only internal from Access Request Service (via Istio)
  - request body:
    - `download_access`: bool
    - `valid_from`: date
    - `valid_until`: date
  - returns nothing

If `download_access` is set to true, then a controlled access grant for the
specified user and dataset with the given dates will be added to the Claims
Repository, replacing any other controlled access grants for the same user
and the same dataset (this means that multiple access grants with different
time frames are not supported). If `download_access` is set to false, then all
controlled access grants for the same user and the same dataset will be removed
from the Claims Repository.

In order to facilitate authorization, the path of these endpoint starts with
`download-access` and not with `users` which is already used by other endpoints
of the user registry and claims repository. Also note that a corresponding
GET endpoint already exists for use by the Download Controller Service.
An Istio policy should be implemented that allows the Access Request Service
to only use the POST method, and the Download Controller Service to only use
the GET method.

## Human Resource/Time Estimation

Number of sprints required: 2

Number of developers required: 1
