# Dataset

*A dataset that is a collection of files.*

## Identifier

*An identifier used by the submitter.*

The identifier uses the property name `alias`.

## Content

Properties that are defined as part of the content of this class.

- **`dac_contact`** *(string, format: email)*: The email address to contact the DAC for this dataset.

## Relations

Relations that are established to other classes.

- **`files`** (*to class [`File`](File.md)*):
    - **description:** The files contained in this dataset.
    - **modality & cardinality:**
      - A dataset MUST reference at least one (MAY reference multiple)
        instance(s) of the `File` class.
      - In turn, each instance of the `File` class MUST be referenced
        by at least one (MAY be referenced by multiple) instance(s)
        of the `Dataset` class through this relation.
