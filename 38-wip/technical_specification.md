# Missing Glue Code for Ingress Inter-service Communication (Tokay Gecko)
**Epic Type:** Implementation Epic

## Scope

This epic aims to fill in the missing parts in inter-service communication along the preliminary upload/ingress path provided for 1.0.

### Outline:

#### Metldata Service:

- Implement publisher for deletion and population events
- Needs to know about MASS and WPS DB and be able to query them
- Invesigate: Communication with MASS DB might be possible through existing API

#### MASS:

- Add event subscriber for deletion and population events
- Add functionality to populate entities
- Add functionality to delete entities
- embedded_dataset artifact as input in form of MASS models.Resource
- Kafka Key Name: dataset_ebmedded_{id}

#### WPS:

- Question to solve: Which service produces DatasetOverview?
- Add event subscriber config for deletion and population events
- Add functionality to delete datasets
- Kafka Key Name: dataset_embedded_{id}?


#### Sequence Diagram for Proposed Interactions

```mermaid
sequenceDiagram
  participant Kit as Datasteward Kit
  participant API as Metldata Service
  participant MASS
  participant MASS_DB as MASS Database
  participant WPS
  participant WPS_DB as WPS Database

  Kit ->> API: Load artifacts from all submissions

  API ->> MASS_DB: Query known datasets
  MASS_DB ->> API: Return known datasets
  loop Each dataset
  API -->> MASS: Send deletion event for dataset
  end

  API ->> WPS_DB: Query known datasets
  WPS_DB ->> API: Return known datasets
  loop Each dataset
  API -->> WPS: Send deletion event for dataset
  end

  loop Each dataset
  API -->> MASS: Inform about new dataset
  end

  loop Each dataset
  API -->> WPS: Inform about new dataset
  end
```

Kafka topic has to be the same for deletion and creation to guarantee order for events with same key - use type to distinguish

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 2
