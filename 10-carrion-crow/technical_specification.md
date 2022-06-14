# File Tech Stack Exploration (Carrion Crow)
**Epic Type:** Exploratory Epic

## Scope:
A scope definition can be found here: https://wiki.verbis.dkfz.de/x/YQH9Cg

## Components of Exploration:
This epic covers principle technologies that might be used in the context of file handling


### Crypt4GH

### Checksum validation

Provide advantages and disadvantages of common checksum functions. Compare the following widely used checksum algorithms:

- MD5
- CRC-32
- CRC-32C
- SHA-1
- SHA-256

List advantages and disadvantages of each algorithm. Compare checksum calculations for the same file with each algorithm. Use the following file sizes:

- Small file: Between 10 and 100 MiB
- Medium file: Between 1 and 10 GiB
- Expected big file: ~ 50 GiB
- Maximum possible file~ 150 GiB


### FastQC

### Volume/ Hardware Level encryption


## Material and Resources:

### Realistic Sample files:
Take samples from the Genome in a bottle (GIAB) project: https://ftp-trace.ncbi.nih.gov/ReferenceSamples/giab/data/
Another source for publicly available genome files would be the 1000 Genomes project: http://ftp.1000genomes.ebi.ac.uk/
### crypt4GH:
- crypt4GH Documentation: http://samtools.github.io/hts-specs/crypt4gh.pdf
- crypt4GH python utility: https://crypt4gh.readthedocs.io/en/latest/
- crypt4GH rust utility: https://docs.rs/crypt4gh/latest/crypt4gh/

### FastQC
- FastQC Documentation: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/

### Volume/Hardware Level Encryption
- IBM COS Documentation: https://cloud.ibm.com/docs/cloud-object-storage
- IBM COS expert in Heidelberg: Koray
- Ceph Documentation: https://docs.ceph.com/en/quincy/
- Ceph Storage in Tübingen deployed by Sardina Systems (info@sardinasystems.com)
- Ceph Storage & de.NBI expert in Tübingen: Moritz


## Human Resource/Time Estimation:

Number of sprints required: 2

Number of developers required: 2
