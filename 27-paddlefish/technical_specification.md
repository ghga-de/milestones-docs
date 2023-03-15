# Implementation of encryption and parallel file part processing in uploads and downloads for ghga-connector (Paddlefish)

**Epic Type:** Implementation Epic

**Attention: Please do not put any confidential content here.**

## Scope:

This epic covers two features to be implemented in the ghga-connector:

1. Crypt4GH compliant encryption of files to be uploaded
2. Parallelization of file part uploads/downloads


## Implementation Details:

### Encryption

Crypt4GH encryption needs the user's private key and the GHGA public key.
For now, both these keys should be provided by adding configuration options that expect the respective paths to the key files.
As the connector already expects a user public key, the private key should belong to the same keypair.

Once the work package service is finished, this will change slightly.
Following changes, which are not part of this epic, need to be made then:
- Obtain GHGA and user public key from external source (work package service or possibly API call for GHGA key)
- Verify the provided keypair matches the announced public key

The implementation should check if a provided file is already Crypt4GH encrypted and reject such files for now.
Such a check could interrogate the magic bytes in the file header and additionally verify that the file ending is correct.
For non-rejected files, encryption should produce a Cypt4GH compliant temporary on disk file that is subsequently uploaded.

A possible issue might arise with the Crypt4GH private key being unreadable by the connector due to strict permissions.
This should be considered and might need additional time for investigation and experimentation.

### File Part Parallelization

File transfer operations should benefit of parallelization over the parts of each file. In an inital exploration step, the possbilities of how to achieve this goal schould be investigated.
Possible mechanisms for scheduling and managing parallel up-/download tasks include multiprocessing and async queues.
In addition, replacing the fully synchronous requests library with one supporting asynchronous operations like httpx or aiohttp might yield further performance improvements.

The subsequent implementation task is fully dependent on the results of the exploration. While the details are not described here, it is a part of the epic and should transform the acquired knowledge into corresponding code changes.

## Human Resource/Time Estimation:

Number of sprints required: 1-2

Number of developers required: 2
