# Re-Implementation of the Metadata Schema in Schemapack
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**

## Scope
### Outline:
Reimplement the metadata model, currently implemented in LinkML, using schemapack.
This should be used to onboard the metadata team and illustrate the use of schemapack for our purposes.
Thereafter, the metadata team should be enabled to drive the metadata model further independently.


### Included/Required:
I. re-implement the entire LinkML-based model in schemapack
II. autogeneration of mermaid-based ER diagrams from a schemapack specs
III. autogenerate documentation from schemapack specs
IV. transpile schemapack-based models to a spreadsheet representation

## Implementation Details:
### I. Schemapack-based re-implementation of the Metadata Model:
- The schemapack version 0.2.0 should be used.
- following algorithm might streamline the migration:
  1. Transpile the LinkML model to JSON Schema.
  2. Using a script, isolate idividual classes into their own files.
  3. Write a config file (e.g. in json; possibly even automatically extract from LinkML)
     that lists all the relation properties by class.
  4. Write a script that removes the relation properties defined in step 3 from the JSON schemas
     derived in step 2. The resulting JSON schemas can serve as content schemas to be referenced
     by the schemapack generated in next step.
  5. Write a script that automatically generates a schemapack definition using the Class and relation
     information inferred from the intermediary JSON Schemas produced in step 2 and the config
     specified in step 3. Also migrate existing identifier and relation descriptions.
  6. Manually curate modalities and cardinalities in the schemapack generated in step 5.
  7. Make sure the scripts used to successfully migrate to schemapack are persisted in the
     git history.
  8. Cleanup by removing:
     - the LinkML model
     - the migration scripts and configs
     - all intermediary artifacts
  9. Automatically generate ER diagrams and markdown descriptions to document the model.
  10. Refactor the directory structure to adapt to the new implementation

### II. autogeneration of mermaid-based ER diagrams from a schemapack specs:
- To be implemented into the schemapack library with a dedicated CLI command
- The mermaid language is used as output (visualization must be performed
  elsewhere)
- The following example schemapack:
  ```yaml
  schemapack: 0.2.0
  description: A simple schemapack # with no root class but with descriptions.
  classes:
    File:
      id:
        propertyName: alias
      content: ../../content_schemas/File.schema.json
    Dataset:
      id:
        propertyName: alias
      content: ../../content_schemas/Dataset.schema.json
      relations:
        files:
          targetClass: File
          mandatory:
            origin: true
            target: true
          multiple:
            origin: true
            target: true
    ```
- Should be transpiled to mermaid as:
  ```
  erDiagram
      File }|--|{ Dataset: "Dataset.files"
      Dataset {
          string dac_contact
      }
      File {
          string filename
          string format
          string checksum
          int size
      }
  ```
- visualized as:
  ```mermaid
  erDiagram
      File }|--|{ Dataset: "Dataset.files"
      Dataset {
          string dac_contact
      }
      File {
          string filename
          string format
          string checksum
          int size
      }
  ```




## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: multiple
