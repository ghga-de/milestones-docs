# DLQ Service (Matamata Turtle)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

## Scope
### Outline:
This epic is concerned with the implementation of a basic dead letter queue service,
which will provide the means to assess, process, and retry Kafka events that resulted
in errors during their initial processing. This service will make use of the
[Kafka DLQ](https://github.com/ghga-de/hexkit/pull/123) additions to `hexkit`.

### Included/Required:
- Implementation of the DLQ service (including its test suite)
- `hexkit`: Add exception information to DLQ event headers
- `hexkit`: Add preview functionality to `KafkaDLQSubscriber`
- `hexkit`: Add preview dry-run functionality to `KafkaDLQSubscriber`

### Optional:
- Add event ID header in `hexkit`, consisting of the original topic, partition, and
offset

### Not included:
- A web-based user interface

## API Definitions:

### RESTful/Synchronous:

DLQ topics consist of the original topic name, the service abbreviation, and the fixed
suffix `-dlq`. The latter could technically be omitted, but using it visually
distinguishes dlq topics from others, making debugging easier.
When interacting with DLQ topics via the API, the above structure does not need
to be remembered: only the service abbreviation and the plain topic name.

- `GET /services`
  - *Returns all configured topics for all services*
  - Auth Header: internal token
  - Response Body: dict with service names as keys and array of topic names as values
  - Response Status: `200 OK`
- `GET /{service}/{topic}`
  - *Returns the next events in the topic.*
    *This is a preview of the events and does not impact event ordering within the DLQ.*
    *Like the test endpoint below, this endpoint is idempotent.*
  - Auth Header: internal token
  - Query parameters:
    - `limit` *(int)*: the maximum number of events to preview per 'page'
    - `skip` *(int)*: pagination param to skip the first `skip` results
  - Response Body: array of event objects
  - Response Status:
    - `200 OK`: The preview was successful
    - `401 Unauthorized`: auth error (not authenticated)
- `GET /test/{service}/{topic}`
  - *Returns the result of processing the next event in the specified topic. Nothing*
    *is published and offsets are not affected. Calling the same test path multiple*
    *times will return the same result.*
  - Auth Header: internal token
  - Response Body: the event that would be published, empty if event would be discarded.
  - Response Status:
    - `200 OK`: The event test was successful
    - `422 Unprocessable Entity`: The non-empty request body is not a valid event. 
    - `401 Unauthorized`: auth error (not authenticated)
- `POST /{service}/{topic}`
  - *Processes the next event in the topic, optionally publishing the supplied event*
  - Auth Header: internal token
  - Request Body: empty OR JSON representation of corrected event
    - Need to consider whether checks should be added for fields like correlation ID.
  - Response Status:
    - `204 No Content`: The event has been processed
    - `422 Unprocessable Entity`: The non-empty request body is not a valid event.
    - `401 Unauthorized`: auth error (not authenticated)
- `DELETE /{service}/{topic}`
  - *Directly discards the next event in the topic*
  - Auth Header: internal token
  - Response Body: empty
  - Response Status:
    - `204 No Content`: The event has been discarded
    - `401 Unauthorized`: auth error (not authenticated)

Each endpoint will return a `404` status code if the `service` parameter does not match
an existing service, or the `topic` parameter is not configured for the given service.

All endpoints should require an internal auth token.

### Payload Schemas for Events:

**Outbound from DLQ Service**:  
The outbound events published to retry topics will look exactly like other Kafka events
published or consumed by the GHGA microservices, except they will feature the
`original_topic` header.

**Inbound to DLQ Service**:  
The inbound events in the DLQ topics will feature additional
headers to communicate exception information like the message and class name. That will
require a small modification to `hexkit`. In addition to the exception information,
DLQ events will feature a header containing the original topic, partition, and offset
of the failed event in a single string. This enables devs to quickly confirm that the
event they see in the DLQ service is in fact the same event from a given error log.

**Previewed Events**:  
Events returned by the preview endpoint will match the format defined by the
[`ExtractedEventInfo`](https://github.com/ghga-de/hexkit/blob/e3cad2d57212d1ba897e620501d5d2b120c70338/src/hexkit/providers/akafka/provider/eventsub.py#L63) dataclass.

**Other**:  
The event ID header (consisting of the original topic, partition, and offset) will be
featured in both inbound and outbound events.


## Additional Implementation Details:

### Definitions:
- *Requeue/Republish an event*: In the context of the DLQ service, this means to publish
  the next Kafka event from a given DLQ topic to the corresponding retry topic.
- *Process an event*: Apply validation and processing logic (if applicable) to the next
  event in a given DLQ topic. The event will either be republished or discarded as a
  result.
- *Discard an event*: To decline to requeue/republish an event to the retry topic. This
  can occur upfront via a direct command by the user, or it can occur as a result of
  the event-processing logic.
- *Resolve an event*: Process or discard an event.
- *Preview events*: Peek at the next N events without affecting the order of events in
  the DLQ topic. This is crucial functionality that enables the user to decide how to
  resolve the next event. The number of events previewed is capped by configuration.

### DLQ Topic Arrangement

For a given Kafka topic, there will be a separate DLQ topic *per service*.
For example, the UCS, DCS, and IFRS subscribe to the topic containing File Deletion
Requested events. They would publish failed File Deletion Requested events to their own
DLQ topics, e.g. `file-deletions.ucs-dlq`, `file-deletions.dcs-dlq`,
`file-deletions.ifrs-dlq`. However, each service will have its own dedicated retry
topic regardless of the number of DLQ topics. In the above example, there would be three
distinct retry topics: one for the UCS, one for the DCS, and one for the IFRS.

![DLQ Topic Arrangement](./images/topic%20distribution.png)

### Event Ordering:
Dead letter queues inherently present a potential threat to system-wide event ordering.
However, ordering events by keys, the idempotent design of our services, and having
separate DLQ topics for each original topic *per service* prevents sequence problems
as long as events are designed to use the correct keys and topics in the first place.
Additionally, it is imperative that only one partition is configured per DLQ topic as
there will be only one consumer for each DLQ topic. If there were multiple consumers
reading from a DLQ topic, it's likely that ordering problems would soon arise without
careful programming to ensure the preservation of event order.

### Event identification:
Right now, events can be identified through a combination of correlation ID + event
type and topic. It would be easier if there were a single field to associate a given
event in a DLQ topic with its source event in the original topic. The combination of
topic name, partition, and offset uniquely identify an event within a topic, so we can
concatenate the three values in a single string and pass it along as an event header.

E.g.: `file-registrations - 0 - 1` would indicate an event from the "file-registrations"
topic, partition 0, offset 1. The information is only seen by the consumer, not by the
publisher, which is the only downside of doing it this way instead of generating a UUID
right before publishing the event.

The event ID would only be propagated if it entered the DLQ cycle.
This differs from the correlation ID, which is propagated across services and persisted
in the database in the case of outbox events. Also unlike the correlation ID,
the event ID is never saved to the database at any point.

To illustrate:

![Event ID usage](./images/event%20ID.png)
1. An event is published to the "users" topic, where it is stored at partition 0, offset 17.
2. The `hexkit` Kafka provider used by the `NOS` gets the message from `aiokafka`.
3. The Kafka provider hands the transformed event over to the service-defined
   translator, which performs some business logic driven by the event.
4. The translator encounters an error which it cannot mitigate, and all retries fail.
5. The Kafka provider logs the error, including the event ID (`"users - 0 - 17"`).
6. The Kafka provider publishes the event with the event ID header to the DLQ topic.
7. We use the DLQ service to take a look at the failed event and see nothing wrong.
8. We learn the problem was actually a database issue that we've since fixed.
9. We use the DLQ service to publish the event to the retry topic, sans event ID header.
   Instead, the DLQ service includes a special header with the original topic ("users")
   but not the partition or offset. 
10. The service encounters the republished event, this time from its dedicated retry
  topic. The Kafka provider understands that the retry topic is special, so it obtains
  the topic field from the original topic header. If the retry event fails again, the
  old event ID would be misleading had we modified the event in the DLQ service.

If the translator encounters no errors, the event ID is not needed and never created.


### Event processing:
When we resolve the next event in a given DLQ topic, it can go one of two ways:
- We can discard the event, meaning it is not republished to a retry queue and is
  effectively ignored.
- We can process the event, where we run the event through custom logic that can be as
  extensive or specific as needed. Ultimately, the event would be requeued to the retry
  topic or discarded. The processing can also be bypassed by directly supplying a JSON
  event to the API, which publishes the event outright.

The event processing logic can be part of the DLQ service. The idea is that the event
can be routed to some bit of code based on its service of origin, topic, type, or
anything else. The logic can also exist elsewhere, from which an event is provided in
an API request.

Another useful piece of information for the DLQ service is the exception information.
There is currently no mechanism to provide that to the DLQ service, but the info could
be passed through event headers. The exception header would then be removed upon
republishing the event to a retry topic.

This [Java DLQ implementation](https://medium.com/nerd-for-tech/-to-re-queue-apache-kafka-dlq-messages-95941525ca77)
uses such headers (toward the bottom).

### Test Endpoint:
It would be nice to be able to perform a dry-run of event processing as a sanity check.
The test endpoint would perform any applicable processing on the next event and
return the result instead of publishing it. The DLQ service would maintain the initial
offset position, just like with the preview action. If processing would result in the
event being discarded instead of published, then it will be indicated by the return
value.


### Configuration:
The DLQ service needs to know which topics are subscribed to by each service.
We might benefit by listing the topic names along with each consuming service.
Something like:

```yaml
subscriptions:
- ifrs:
  - file-deletions
  - file-interrogations
  - file-downloads
- ucs:
  - file-deletions
  - ...
# etc.
```
or inversely:

```yaml
subscriptions:
- file-deletions: [ucs, dcs, ifrs]
- file-downloads: [ifrs]
# etc
```

```python
subscriptions: dict[str, list[str]]
```

### Previewing Events:
The DLQ service has to be able to preview Kafka events so we can see what's in a given
DLQ topic. This can't be accomplished by the consumer loop we use in our standard
Kafka providers because the events must be seen once for the preview, then a second time
to resolve the event. The `KafkaDLQSubscriber` only has the ability right now to resolve
events, so it must be updated.

When a preview for N events is fetched, the DLQ service's consumer should revert its
offset to the original value. Sequential calls for previewing N events should always
return the same N events if no resolutions are executed in the meantime.

Previewed events will be formatted with the `ExtractedEventInfo` class and returned as 
JSON:
```json
{
  "topic": ...,
  "type_": ...,
  "payload": {
    "field1": ...,
    "field2": ...,
    ...
  },
  "key": ...,
  "headers": {
    "correlation_id": ...,
    "exc_class": ...,
    "exc_msg": ...,
    "id": <topic - partition - offset>
  }
}
```

### Tests:
At the very least, the DLQ service tests should cover the following:
- Test that previewing is idempotent and returns events in consistent order
- Test that endpoints are secured
- Test that API calls trigger the correct action
- Test that events are consumed from the correct DLQ topics
- Test that events are published to the correct retry topics
- Test that only one event is processed or discarded per API call
- Test that the preview endpoint returns the next N events (e.g. not off by 1)
- Test that the preview endpoint does not disturb the offset of the next-resolved event
- Test that the service doesn't break if Kafka data is lost (i.e. offsets adjust)


### Usage:
Startup: In the current form there is only one entrypoint, so the container only needs
to be started (no specific CLI command required).
Preview & resolve events: Done via HTTP API calls.

### Scaling:
The DLQ Service should not be scaled, because the manual intervention required will
be the limiting factor rather than infrastructure.

## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: 1
