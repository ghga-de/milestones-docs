# Correlation ID Exploration (Marabou Stork)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**

## Scope
### Outline:
The aim of this epic is to implement the functionality required to use interservice correlation IDs for tracking associated requests.
Please see the prototype [here](https://github.com/ghga-de/prototype-correlation-id).

### Included/Required:
- Generate Correlation ID in API Gateway
- Correlation ID Utilities in Hexkit
- Correlation ID in Kafka Event Header
- Middleware for FastAPI Apps

### Not Included:
- Updating services to use correlation IDs


## Additional Implementation Details:

### Generate Correlation ID in API Gateway
The API Gateway will need to be updated with functionality to ensure every request has a correlation ID in the header.

### Correlation ID Utilities in Hexkit
ContextVars can only be referenced via a single instance, so it is not possible to instantiate a correlation ID ContextVar from one library and set it via another library without importing it. Therefore, `hexkit` should be made a production dependency of `ghga-service-commons`, and correlation ID utilities should be defined within `hexkit`.

The following are needed in `hexkit`:
- A ContextVar to store the correlation ID.
- An asynccontextmanager to set the correlation ID ContextVar, yield, and then reset the token.
- A function to retrieve the value stored in the ContextVar and raises an error if none is found.
- A function that validates a correlation ID value and raises an error if the value is empty or invalid.

### Correlation ID in Kafka Event Header
There needs to be a way to supply the correlation ID as a Kafka event header. If the correlation ID is not supplied, one should be generated, but only if a flag has been explicitly set. Otherwise, an error should be raised.
Kafka events have a header that can be used to store ancillary information, but the current Kafka providers and protocols don't allow for access to the event headers.
Since it will be passed as an event header detail, correlation ID should be added as a parameter to the following:
- `_publish_validated()` and `publish()` methods of the `EventPublisherProtocol` class
- `_publish_validated()` method of the `KafkaEventPublisher` class

The `_consume_event()` method in the `KafkaEventSubscriber` class needs to be updated to extract the correlation ID from the header, validate it (raising an error if invalid/missing), and set the ContextVar before calling `self._translator.consume()`

### Middleware for FastAPI Apps
A new middleware function for FastAPI apps should be added to `ghga-service-commons` to validate correlation IDs upon receiving requests.
All requests should have a correlation ID already set by the gateway, so no logic is needed for ID generation.
If the correlation ID is missing or invalid, an error should be raised. After getting the validated correlation ID, the middlware should set the correlation ID ContextVar using the utility created in `hexkit`. Doing so will allow services to retrieve the correlation ID from the ContextVar without any extra work.

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 2
