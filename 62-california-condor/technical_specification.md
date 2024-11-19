# DB Versioning (California Condor)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

## Scope
### Outline:
This epic is for the implementation of the database versioning concept used to
transition data for a given database when a relevant schema change or systematic content
update occurs. Database versioning also provides a way to detect whether the current
database instance is the expected version or not. 


### Included/Required:
- Initial implementation on single service:
  - Add database version
  - Logic that checks database version at startup
  - Outer migration logic:
    - Locking mechanism
    - Start migration from detected version
  - Refinements and abstraction of common logic if applicable
- Apply database versioning to remaining services


## Additional Implementation Details:

### General Migration Logic

Each microservice owns its own database, so any discussion of migrations can be assumed
to concern a single microservice. According to
[the ADR](https://github.com/ghga-de/adrs/pull/28), a given service will use a
single value to denote the version of its entire database, as opposed to versioning a
collection or the schema used for a document.

The *current* database version will be stored in a dedicated collection. The *expected*
database version will be stored in application code. When a service starts up, the
first action will be to compare the actual current version number against the expected
value. If the expected version number is not found in the database version collection
and the lock is not set, the service can start the migration process.

```json
// Database version collection upon migrating a database from version 1 to version 2:
[
  {
    "version": 2,
    "completed": ISODate("2024-11-18T09:30:00Z"),
    "duration_sec": 25
  },
  {
    "version": 1,
    "completed": ISODate("2024-10-07T09:00:00Z"),
    "duration_sec": 22
  }
]
```

The migration process will form a chain, where discreet migration logic exists for
every database version to migrate the data from version X to X+1. Most of the time, the
database will be current or, at most, one version behind. However, if a database restore
occurs and the data happens to be older, the migration process will begin at the
appropriate step in the migration chain and continue until the data is totally migrated.
Migration code should be preserved at least until there is no possibility of 
encountering the corresponding database version again.


### Services with Multiple Entrypoints

Several services operate with more than one instance simultaneously because one serves
as a REST API and another consumes Kafka events (for example). Obviously, only one
migration process should occur in these situations, rather than executing for each
instance. This can be solved if we ensure the migration process only runs as part of
the startup for one entrypoint, e.g. the REST API. We can "lock" the database to signal
to any other potential instances that there is already a migration in progress:

```json
// locking collection with one document
[
  {
    "migration_in_progress": true
  }
]
```

The first instance to obtain the lock is allowed to proceed, all others wait. This has
the benefit of preventing concurrent migrations if services are scaled. An alternative
to a dedicated collection is to use the database version collection -- if the
expected database version exists but the timestamp is missing, then that means a
migration is already underway.

After preventing simultaneous migrations, we need to optimize how other instances
operate while a migration is underway and consider how to handle read and 
write requests. One way would be to accept some downtime and
block service instances until the migration is complete. This might be a
sufficient initial solution because database sizes are small enough that migrations will
be completed quickly. If we have to eliminate downtime, we could perform shadow
migrations and write the results to temporary collections while the old service
version continues to handle requests. When the migration is complete, the old service
can be taken offline and the collections swapped out. That's a little more complex.

### Reverse Migrations

There might be some situation where we need to apply the reverse of a migration.
If DB version 5 is applied, but we later find that we should have stayed with version 4,
then we need to move to version 6. Version 6 is not treated as some special 'undo'
version increment; the migration logic merely happens to move the data to the same
structure contained in version 4.

### Migration Structure

![Migration structure](./images/db%20migrations%20white%20bg.png)

In the image above, the green section indicates the top-level migration logic, most of
which can be implemented in a library like `hexkit`.  
The *rounded* orange boxes represent distinct migrations between database versions.  
The *square* orange boxes show common logic that can be abstracted into a library.  
The red-dotted items would be performed once for each batch of documents if a collection
were to be processed in batches.  
The gray box shows where per-collection migration logic would occur.

### Errors During Migration

If an error prevents a migration from finishing, then we should discard the processed
entries, unset the lock document, and log the error. The cleanup is straightforward if
migrated documents are stored in a temporary collection that can be dropped, rather than
modifying the original collection directly (in which case we would need to reverse
changes, which could be difficult). It's important that we test migrations thoroughly
to avoid extended downtime from unexpected errors.

### Testing Migrations

Abstracted logic should be tested wherever it lives, like `hexkit`.
Tests should cover the following, but the list is not exhaustive:
- The locking mechanism
- Error handling
- Logging
- Database version detection
- Selecting the right migration to start with
- What to do when no matching migration exists

When we test individual migration code, like the code that will update collection A for
the migration from database version X to Y, we should use some mock data that represents
documents in the database.
Tests should at least verify that the migration code applies the right changes.


### Monitoring

Migration progress should be reported at periodic intervals, and the migration duration
should be both logged and stored in the database along with a timestamp so we can
identify performance issues early on.


## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: 1
