# Autogenerate Docs from Schemapack
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**

## Scope
### Outline:


### Included/Required:
I. Autogenerate documentation from schemapack definition  

## Implementation Details:

### I. Autogenerate documentation from schemapack specs
- To be implemented to the schemapack library with a dedicated CLI command
- to keep the scope small, the command should
  deliberatively have no config option and should be
  tailored towards our immediate requirements while not
  being specific to the submission model
- Markdown documents should be generated as output to
  describe every class, their contents, and their
  relations to each other
- An overview document contains:
  - a title as inferred from the schemapack definition (if
    not present, the file name without the file extension
    is used instead)
  - an optional description of the overall schemapack
    as inferred from the schemapack definition
  - a ER diagram showing all classes and their relations (exclude
    content information for simplicity)
  - a link to a separate document with a more verbose ER diagram
    (including content information) is provided
  - a table listing for each class the name of the class in the first
    column, the description of the class (obtained from the content
    schema) in the second column, and link to a document with details for
    the class in the third column
- Per class, a document with details is provided.
  - The separation of concern between describing the content
    and the relations should be reflected in the documentation
  - A first paragraph shows the top level description of the class
    (obtained from the content schema)
  - A second paragraph describes the identifier of this class
    (mentionin the property name and the description as per
    the schemapack definition)
  - A third paragraph describes the structure of the content schema.
    The [jsonschema2md](https://pypi.org/project/jsonschema2md/) library
    (or an alternative) is used to automatically translate the content
    schema with all its structural information and plain text descriptions
    into markdown. The readme generation for the config schema in the
    microservice repository template might act as reference
    (see https://github.com/ghga-de/microservice-repository-template/blob/main/scripts/update_readme.py#L153
    and https://github.com/ghga-de/microservice-repository-template/tree/main?tab=readme-ov-file#configuration).
  - A fourth paragraph lists all relations this class defines to other classes:
    - For each relation, the following is specified:
      - the name of the relation
      - the description of the relation (if there is any)
      - A description of the modality and cardinality in text.
- an example documentation for a simple model (as
  described
  [here](./examples/schemapack/simple_relations.schemapack.yaml))
  is provided [here](./examples/docs/overview.md)
- optioned might be explored how this output is best integrated into mkdocs projects

## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: multiple
