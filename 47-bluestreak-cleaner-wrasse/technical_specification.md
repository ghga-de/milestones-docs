# Automatically remove stale content from buckets (Bluestreak Cleaner Wrasse)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).


## Scope
### Outline:

This epic aims to provide mechanisms to remove no longer needed objects from intermediate objectstorage buckets, i.e. every bucket except the permanent one.
To this end, two questions have to be solved for each bucket:
 - Which service is responsible for cleaning which bucket?
 - How is the cleaning job initiated, i.e. by event or periodically?

### Included/Required:

#### Datasteward Kit:
Functionality to actually call the PCS and request file deletion is not yet implemented.
The DS-Kit should be the correct place for this and a new (sub)command exposing this functionality should be made available.

#### Upload Controller:
Currently, incoming data accumulates in the inbox bucket. Instead, data should be removed asap, once it's no longer needed in the inbox during upload. This should be handled when receiveing the validation success/failure event from the interrogation room.

Additionally, a peridic job should check if files remain in the inbox, that are no longer needed and (for now) log affected objects. As objects should be purged asap, objects remaining in the bucket for an accepted/rejected/cancelled upload mean there is a probably a logic bug somewhere.

A Purge Controller Endpoint should be added to immediately remove all file information for a specified file.


#### Interrogation Room:
Currently, incoming data accumulates in the staging bucket. Instead, data should be removed asap, once it's no longer needed in staging during upload. This should be handled once the file is sucessfully registered and moved by the internal file registry.

Additionally, a peridic job should check if files remain in staging, that are no longer needed and (for now) log affected objects. As objects should be purged asap, objects remaining in the bucket mean there is a probably a logic bug somewhere.


#### Download Controller:
The download controller already has functionailty to respond to PCS events and a periodic cleaning command.
However, the cleaning command only takes into account one given bucket and should probably perform the job for all configured buckets.

### Optional:


### Not included:


## User Journeys (optional)

This epic covers the following user journeys:



## API Definitions:

### Payload Schemas for Events:


## Additional Implementation Details:


## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 1
