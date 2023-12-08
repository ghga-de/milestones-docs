# Structured Logging Tools (Topi Antelope)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**

## Scope
### Outline:
The aim of this epic is to implement tools that enable consistent structured logging,
with the result being a set of minimally-invasive, configurable logging tools. In addition,
`uvicorn` needs to be configured such that its formatting is consistent with other logs.

### Included/Required:
- Logging Tools in Hexkit
- Consolidating Uvicorn Logging
- Pilot Repository

### Not Included:
- Updating services to use new logging tools.


## Additional Implementation Details:

### Logging Tools in `Hexkit`
There needs to be a way to easily invoke a logger object with minimal boilerplate code.
There should also be a dedicated logging configuration class that subclasses `BaseSettings`.

The configuration should at least include the log level for the service.
The configuration should be consumed by a logger factory, which can then be used to
create local logger objects with the configured log level and requisite formatting control.
The logs should be emitted in a JSON string format (one line). An example log message is as
follows (formatted for ease of reading):

```json
{
	"timestamp": "2023-12-04T15:30:00Z",
	"service": "ucs",
	"instance": "001",
	"level": "ERROR",
	"correlation_id": "d826361b-3734-4590-b3e8-3cbed68b9236",
	"message": "The file with ID 123xyz is already in the inbox.",
    "details": {
		"file_id": "123xyz"
	}
}
```
Additional configurable properties could include a way to disable uvicorn integration
(e.g. for easier reading during development), or a format string that can be used to override
the JSON output format.

To summarize, this task includes all items in `hexkit`:
- Logging Configuration Class
- Logger factory
- The subclass implementations of any support `logging` classes, such as
Formatter and LoggerAdapter.

### Consolidating Uvicorn Logging
The `ApiConfigBase` in `ghga-service-commons` class should be redefined to subclass from
the new `LoggingConfig` class (name subject to change) in `hexkit.`
The library also needs to be modified so uvicorn's logging configuration can be changed.
Uvicorn uses a special formatter class and comes with colored output. However, it does
not include certain standard information by default, such as a timestamp.
Uvicorn should be configured or set up such that any resulting log messages retain the
same JSON format. The logs should be shipped with timestamps and all relevant information
listed in the example above.

### Pilot Repository
A single repository (which is to be determined) will be updated as part of this epic.
The purpose of this is to verify the aforementioned changes and identify any problems
before rolling out the changes to other services.

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
