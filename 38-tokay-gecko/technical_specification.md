# Missing Glue Code for Metadata Ingress Inter-service Communication (Tokay Gecko)
**Epic Type:** Implementation Epic

## Scope

This epic aims to fill in the missing parts in inter-service communication along the preliminary upload/ingress path provided for 1.0.

### Outline:

#### Metldata Service:

- Implement publisher for deletion and population events
- Needs to clear and repopulate own DB to track current datasets
- Needs to communicate with new claims repository endpoint to remove access grants

#### MASS:

- Add event subscriber for deletion and population events
- Add functionality to populate entities
- Add functionality to delete entities
- embedded_dataset artifact as input in form of MASS models.Resource
- Kafka Key Name: dataset_ebmedded_{id}

#### WPS:

- Question to solve: Which service produces DatasetOverview?
- Adjust event subscriber config for population events (if needed)
- Kafka Key Name: dataset_embedded_{id}

#### Auth Service Claims Repository

- Add endpoint to delete access grant claims for a specific dataset

#### Sequence Diagram for Proposed Interactions

```mermaid
sequenceDiagram
  participant Kit as Datasteward Kit
  participant API as Metldata Service
  participant MASS
  participant claims as Claims Repository
  participant WPS

  Kit ->> API: Load artifacts from all submissions
  API ->> API: Query for currently existing datasets in internal DB

  loop Each dataset
  API -->> MASS: Send deletion event for dataset
  end

  loop Each dataset
  API ->> claims: Call API to remove grant for dataset
  end

  API ->> API: Clear internal DB
  API ->> API: Repopulate DB from submission artifacts

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
