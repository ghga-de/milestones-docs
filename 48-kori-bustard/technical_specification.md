# Notification Orchestration Service (Kori Bustard)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**


## Scope
### Outline:
The goal of this epic is to create a base implementation for a new service which
publishes Notification events upon consuming events corresponding to specific points
in user journeys.
A description of the concept can be found in
[this ADR](https://github.com/ghga-de/adrs/blob/main/docs/adrs/adr007_sourcing_notifications.md).
The name of the new service is the **Notification Orchestration Service (NOS)**.

### Included/Required:
- Initial implementation of NOS
- Notification Service Idempotence

### Not Included/Required:

## API Definitions:

### List of Notification Sources

The following list contains notifications and the intended recipients.

_**Authentication**_
> It needs to be determined whether the Data Steward here is Central or Local

| Recipient    | Purpose                                  | Source Exists | Data Req'd |
|--------------|------------------------------------------|---------------|------------|
| Data Steward | LS Login doesn’t match what's registered | No            | User ID    |
| User         | IVA invalidated                          | No            | User ID    |
| User         | IVA verification code requested          | No            | User ID    |
| Data Steward | IVA verification code requested          | No            | User ID    |
| User         | IVA verification code transmitted        | No            | User ID    |
| Data Steward | IVA verification code submitted by user  | No            | User ID    |


_**Data Submission**_
| Recipient     | Purpose                           | Source Exists | Data Req'd |
|---------------|-----------------------------------|---------------|------------|
| Central DS    | *Metadata is ready for review     | No            | None?      |
| RD Controller | Research data upload completion   | Yes           | File ID    |
| RD Submitter  | *Approval/rejection of submission | No            | User ID    |


_**Data Request and Download**_
> If there is a stored entity linking the request to both the dataset and user IDs, then
> the request would be the only piece of information needed from the event.

| Recipient    | Purpose                          | Source Exists | Data Req'd |
|--------------|----------------------------------|---------------|------------|
| DRR          | Request Created (Confirmation)   | No            | Dataset ID, User ID|
| DACR         | Request Created                  | No            | Dataset ID, User ID|
| DRR          | Request Allowed                  | No            | Dataset ID, User ID|
| DACR         | Request Allowed                  | No            | Dataset ID, User ID|
| DRR          | Request Denied                   | No            | Dataset ID, User ID|
| DACR         | Request Denied                   | No            | Dataset ID, User ID|
| DRR          | *Dataset ready for download      | Yes           | Dataset ID, User ID|
| DRR          | *Data access expiration reminder | No            | Dataset ID, User ID|
| DRR          | *Data access expired             | No            | Dataset ID, User ID|

_**Data Deletion**_
| Recipient    | Purpose                     | Source Exists | Data Req'd |
|--------------|-----------------------------|---------------|------------|
| Data Steward | Deletion request received   | Yes           | File ID    |
| Data Steward | *Deletion request fulfilled | Yes           | File ID    |
>(*): TBD if desired or not


## Additional Implementation Details:

The Notification Orchestration Service will use an event subscriber to consume events
from other services. Some of these events already exist, while others still
need to be defined and implemented. This approach ensures that microservices remain
agnostic to the notification framework. Instead, when a point in a user journey is
reached which merits a notification, the given microservice publishes an event. That
event is picked up by the NOS and used to construct a Notification event.

### Initial Implementation

The initial implementation assumes that it has access to a database containing
documents storing required relationships:
- User ID to user email
- Dataset to Local Data Steward email
- Dataset to Research Data Controller email

**Structure**

The NOS will comprise four primary components:
1. An inbound adapter, an event subscriber, to consume notification source events
2. A core containing notification content and relevant logic
3. An outbound adapter for obtaining required information stored in the database
4. Another outbound adapter, an event publisher, to issue Notification events

**Static Content**

The body text used for notifications will be stored in code rather than configuration,
allowing for tighter control over public-facing content. While configuration is more
readily changed, it is crucial that changes to user notifications are reviewed and the
changes documented via version control.

The Central Data Steward email address could be stored in configuration, code, or
provided externally by the Well Known Values Service (WKVS). A decision on this specific
implementation detail is not urgent. For the initial implementation the Central Data
Steward email will be stored in configuration.


### Notification Service Idempotence:

The Notification Service needs to maintain idempotence with regard to event processing,
yet ensure that notifications are issued once and only once. This requirement is not
met in its current state, where reprocessing a notification event would result in
multiple identical emails. One way to achieve this would be to generate a deterministic
key for each event and store in a database whether or not it was sent.

## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: 1
