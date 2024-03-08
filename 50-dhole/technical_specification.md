# Rewriting existing metldata transformations
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

**Attention: Please do not put any confidential content here.**

## Scope
### Outline:
The aim is to rewrite all existing metldata transformations to use schemapack.


### Included/Required:
- add possibility to use custom embedding profile to configure the denormalization
  in schemapack
- migrating the already schemapack-based transformations to spec version 0.2.0
  (see https://github.com/ghga-de/metldata/tree/poc/src/metldata/schemapack_/builtin_transformations):
  - delete_properties
  - infer_relations
- re-implement the following LinkML-based transformations to use schemapack
  (see https://github.com/ghga-de/metldata/tree/poc/src/metldata/builtin_transformations):
  - merge_slots
- re-evaluate and potentially refactor the following transformations:
  - aggregate


### Not included:
- Reimplementation of the custom_ebeddings transformation is not required since it will already
  covered by builtin functionality of schemapack
- Reimplementation of the add_accessions transformation is postponed since it might be
  moved handled by the submission store and not be implemented as transformation.
- A full reimplementation of the GHGA transformation workflow (this depends on other changes to
  be in place first and would make this epic dependent on other lines of work).
- The normalize_model transformation is not needed anymore

## Additional Details:

### Embedding Profile to Configure Denormalization:

This helps to control which relations (and relations of relations) will be embedded and
which won't. It can even be used to deal with circular dependencies that would otherwise
lead to an exception when denomalizing.

Here is an example:

Given the following rooted schemapack:
```yaml
schemapack: 0.2.0
description: A schema used to describe relationships between team members.
classes:
  Person:
    id:
      propertyName: name
    content: ../../content_schemas/AnyObject.schema.json` # Any object allowed
    relations:
      teammates:
        targetClass: Person # refering to itself
        mandatory:
          origin: false
          target: false
        multiple:
          origin: true
          target: true
      manager:
        targetClass: Person # refering to itself
        mandatory:
          origin: false
          target: false
        multiple:
          origin: true
          target: false

  rootClass: Person
```

There is the following datapack:
```yaml
datapack: 0.2.0
resources:
    Person:
        Alice:
            content: {}
            relations:
                teammates:
                    - Bob
                    - Charlie
                manager: Dave
        Bob:
            content: {}
            relations:
                teammates:
                    - Alice
                    - Charlie
                manager: Dave
        Charlie:
            content: {}
            relations:
                teammates:
                    - Alice
                    - Bob
                manager: Dave
        Dave:
            content: {}
            relations:
                teammates: []
                manager: Eve
        Eve: # the boss of Dave
            content: {}
            relations:
                teammates: []
                manager: null
rootResource: Alice
```

Running a denormalization without configuration would not work because there are
circular dependencies between Alice, Bob, and Charlie regarding the teammates relation.

To resolve that, the following embedding profile can be used to configure the
denormalization process:

```yaml
teammates:
  # do embed teammates, however, for each embedded teammate the following embedding
  # restrictions apply:
  teammates: false # do not embed teammates of teammates
  manager: false # do not embed managers of teammates
manager: true # do embed the managers and all the nested relations that the manager
              # might have
```

The outcome would be the following denormalized data:
```yaml
name: Alice
teammates:
  - name: Bob
    teammates:
      - Alice
      - Charlie
    manager:
      - Dave
  - name: Charlie
    teammates:
      - Alice
      - Bob
    manager:
      - Dave
manager:
  name: Dave
  teammates: []
  manager:
    name: Eve
    teammates: []
    manager: null
```

### Immutability of Datapack and Schemapack Objects:
Currently, the pydantic models for interacting with DataPack and SchemaPack
definitions are not fully frozen, yet, however in the future they will be.
Thus, while re-implementing the transformations, modifications to a
DataPack or SchemaPack object should never be done in place but
using methods that would also work on frozen pydantic models (e.g.
the `model_copy(update={...})` method). This is also true for the already
migrated transformation.

### Refactor merge_slots transformation:
The original merge_slots transformation should be replaced by two new transformations:

A. For merging content properties:
- nested properties (of nested JSON objects) must be supported
- if the source properties are not of the same type, a union type is created (AnyOf)
- the pydantic Config might look like this:
  ```python
  from typing_extensions import TypeAlias

  ContentPropertyPath: TypeAlias = str
  content_prop_path_description = (
    "In the simplest case, content property path contains the name of a property"
    + " of interest inside of the content (schema) of a resource (or class)."
    + " However, it might also be used to identify properties of nested objects within"
    + " the content. To do so, the name of the property in the parent class can be"
    + " separated using a dot ('.') from the name of the property in the child class."
    + " If a property name itself contains a dot it can be escaped by '\.' to not be"
    + " interpreted as a nested path."
  )

  class ContentPropertyMergeConfig(BaseSettings):
      """Specify content properties that shall be merged into one new property."""

      model_config = SettingsConfigDict(extra="forbid")

      class_name: str = Field(
        ...,
        description = (
          "The name of the class to which the source (and the merged) properties (will)"
          + " belong."
        )
      )
      source_properties: list[ContentPropertyPath] = Field(
          ...,
          description=(
            "A list of paths to content properties of this class that shall be merged."
            + " {content_prop_path_description}"
          )
          min_length=2,
      )
      merged_property: ContentPropertyPath = Field(
          ..., description=(
            "The path of the new property that will contain a list of values that"
            + " were present in the source properties. Please note, this property will"
            + " will always be a list even if the source properties contained single"
            + " values."
            + " {content_prop_path_description}"
          )
      )
      merged_description: Optional[str] = Field(
          None,
          description="A description of the new content property.",
      )

      # potentially add validators
  ```

B. For merging relation properties:
- the source properties must have the same targetClass
- the pydantic Config might look like this:
  ```python
  class RelationPropertyMergeConfig(BaseSettings):
      """Specify relation properties that shall be merged into one new property."""

      model_config = SettingsConfigDict(extra="forbid")

      class_name: str = Field(
        ...,
        description = (
          "The name of the class to which the source (and the merged) properties (will)"
          + " belong."
        )
      )
      source_properties: list[str] = Field(
          ...,
          description="A list of relation properties that shall be merged."
          min_length=2,
      )
      merged_property: ContentPropertyPath = Field(
          ..., description=(
            "The name of the new property that will containing the merged list (union"
            + " set) of relation."
          )
      )
      merged_description: Optional[str] = Field(
          None,
          description="A description of the new relation property.",
      )

      # potentially add validators
  ```

### Refactor aggregate transformation:
TBD.

## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: multipe
