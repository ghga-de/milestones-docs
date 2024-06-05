# Preliminary Storage Selection (Madagascan Sunset Moth)
**Epic Type:** Implementation Epic

## Scope
### Outline:

This epic aims to change the current quasi hardcoded solution for propagating the correct storage location of files uploaded via the Datasteward Kit to a user facing decision.
Instead of setting a storage alias in the config of the File Ingest Service, the alias information should be part of the information the DS Kit sends to the File Ingest Service.

In addition, the File Ingest Service now needs to communicate with the Well Known Value Service to validate the storage aliases sent by the DS Kit.

### Included/Required:

There are two possibilities to add storage alias information in the DS Kit:

1) Add a global config option or command line argument that is applied to the current batch upload
2) Manually provide a storage alias for each of the files uploaded

As this only affects the metadata ingest phase, the first solution might be easier to handle. 
File metadata is ingested in batches and each batch would be assigned one storage alias/location.
As file metadata is read in from a directory, different metadata could be grouped in  different directories according to storage location.
An explicit command line argument would be the preferred in this setting, as to not upload with the wrong storage location set by accident.

The second option could be realized by a mapping file, but has more potential for manual error and issues with missing entries.

## API Definitions:

### RESTful/Synchronous:

The following existing File Ingest Service endpoints need to be changed:

- POST /legacy/ingest
- POST /federated/ingest_metadata

The expected payload for both endpoints now includes the storage alias.

In addition, the Well Known Value Service needs to provide a new value at the existing

- GET /values/{value_name}

endpoint. 
The proposed value name is `storage_aliases` and this endpoint should return a list of all configured storage aliases.

## Additional Implementation Details:

The Datasteward Kit/File Ingest Service interaction for both non-secret ingest endpoints is changed slightly, by only including the storage alias in the payload sent and received. Theoretically this results in an additional overhead of calling the WKVS for each payload received, but as aliases should not be removed, results can simply be cached in memory and only re-requested when an alias is not found in the local cache.

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
