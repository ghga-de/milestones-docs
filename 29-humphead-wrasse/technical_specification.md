# Notification Service (Humphead Wrasse)
**Epic Type:** Implementation Epic

## Scope:
This epic will include a microservice that will listen for events instructing it to send notifications to users.
It will include:
- Schema definition for email notification events
- Adding the schema to ghga-event-schemas
- Functionality for consuming those events and dispatching notifications via email

It does not include:
- Changing existing services to send notification events
- Alternative notification event types (i.e. only email notifications are supported in this epic)

## Additional Implementation Details:

### Email Notification Event
An EmailNotification event schema will be defined in the ghga-event-schemas repository.
The event schema will detail all the information needed to send a notification via email:
- email address of primary recipients (required)
- secondary (cc'd) recipient email addresses (optional)
- tertiary (bcc'd) recipients email addresses (optional)
- subject line (required)
- name of the recipient (required)
- plaintext email body (required)
The exact field names and constraints will be provided in the ghga-event-schemas repository, which is considered the source of truth.
This information will be used to create an email with a consistent format.

### Notification Service
This is a microservice dedicated to consuming EmailNotification events from the "notifications" topic in kafka.
Other types of notification events will not be handled by this service at this time, but the service could be expanded in the future if needed.
In order to utilize the notification service to send emails, publishers will need to publish an event to the "notifications" topic using the "email_notification" event type, with a payload conforming to the schema defined by EmailNotification in the ghga-event-schemas repository.
The sender's address, email signature, connection strings, and other details will be provided through configuration.

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
