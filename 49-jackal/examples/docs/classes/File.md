# File

*A file is an object that contains information generated from a process, either an Experiment or an Analysis.*

## Identifier

*An identifier used by the submitter.*

The identifier uses the property name `alias`.

## Content

- **`checksum`** *(string)*: A sha256 checksum of the file. This is used to verify the integrity of the file.
- **`filename`** *(string)*: The name of the file.
- **`format`** *(string)*: The format of the file. This should be a valid file extension, e.g. 'txt', 'csv', 'json', 'xml', 'pdf', 'png', 'jpg', 'mp4', 'mp3', etc.
- **`size`** *(integer)*: The size of the file in bytes.

## Relations

No relations are established to other classes.
