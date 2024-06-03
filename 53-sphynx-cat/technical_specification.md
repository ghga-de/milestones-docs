# MASS Artifact Refactoring (Sphynx Cat)
**Epic Type:** Implementation Epic

Epic planning and implementation follow the
[Epic Planning and Marathon SOP](https://docs.ghga-dev.de/main/sops/sop001_epic_planning.html).

## Scope
### Outline:

The goal of this epic is to reconsider how metadata is stored in MASS and implement a
solution that reduces the document size while preserving searchability. The
services involved are `mass` and `metldata`.

MongoDB documents cannot exceed 16 MB, which is insufficient to represent
public-facing metadata with the currently implemented structure. While the data that
exists in GHGA right now has not exceeded that limit, there are some datasets that already
require a significant portion of the 16 MB limit. For instance, there is at least one
non-embedded dataset requiring over 4 MB, meaning the corresponding fully-embedded dataset
is even larger. Given the expected volume of future submissions, it is unrealistic to
assume that no submissions will exceed 16 MB in size. To resolve this, we must rework
the way metadata is stored in `mass`. 

A related problem, which can be solved as a byproduct of this epic, is that Kafka is
unable to represent larger data without being configured to do so. While we could configure
the event payload limit to be larger, any set limit would be arbitrary at best. The best
approach therefore is to reduce the size of the payloads systematically.


### Included/Required:

- Redesign artifact structure for `mass` datasets
- Update `metldata` to publish new artifact(s) in place of the previous design
- Provision sufficiently large (> 16 MB) test data
- Overhaul tests in `mass`
- Integration testing

This tech spec will be updated with the details of the new document structure(s) for
`mass` and the changes for `metldata` once solidified, including any event schema
changes for Kafka.


## API Definitions:


### Payload Schemas for Events:

TBD


## Additional Implementation Details:

The primary task is to design a database representation for the Dataset entities
that reduces the probability of any one document from exceeding the 16 MB limit (with
reasonable certainty). 

One approach could be
to split the document into smaller documents that are linked by a dataset ID. Then a
text index could be applied to the full-text documents and to the root dataset information
for querying. A challenge would be to preserve the relevance scores for the
returned datasets. Additionally, this
approach would likely require further processing for results returned by the aggregation
pipeline because that operation itself cannot return anything bigger than 16 MB*.

### Background on the relationship between `metldata` and `mass`:

Metadata is submitted to `metldata`, which applies a series of transformations in order
to produce "artifacts", or alternative representations of the original metadata, tailored
to the needs of other services. One of the consuming services of such artifacts is `mass`.
`Mass` currently gets one artifact per metadata submission, which contains all that is
needed for the metadata catalogue, and stores that as one document in its database. 
When a user performs a search with the catalog, `mass` uses the search parameters to
drive an aggregation pipeline (as defined by MongoDB). The pipeline compiles hits along
with facet information and count, and `mass` returns that information to the front end,
where it is displayed to the user.

> \* The 16 MB limit also applies documents returned by the aggregation pipeline.
> That means it is not enough to split up documents and compile them again during
> the search process. However, it is possible for transient documents to exceed the 16 MB
> limit -- that is, documents which are formed temporarily by the aggregation pipeline as
> a result of transformations or aggregations.


## Human Resource/Time Estimation:

Number of sprints required: 1

Number of developers required: 2
