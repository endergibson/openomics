# Documentation #
**Temporary note:**
_This section is currently under development. For the time being its purpose is to communicate our vision and activate the discussions in the [public mail list](http://groups.google.com/group/openomics), so rigour, order or connection are not (yet) a main concern._


---

## Goal ##
### `OpenOmics` platform for bioinformatics ###
`OpenOmics` will be a public P2P network able to run over untrusted private heterogeneous nodes (home computers, dedicated servers, etc.). From a technical perspective, it will constitute an extensible infrastructure designed for **collaboration**, over which different applications could be created to solve specific problems. All applications running over this infrastructure will share the same open principles and philosophy that we think are needed for a true altruistic collaboration and also will benefit from several inherited features like high scalability, high availability, resilience and performance; demanded by actual bioinformatics needs. Moreover, this infrastructure would provide some transversal collaborative services (identity, reputation, naming and messaging) to be used by the applications.
### Population database application ###
The original motivation of `OpenOmics` was to create a global repository of personal genotypic, phenotypic, environmental data; and so, achieve an optimal exploitation scenario for identifying patterns in the human genome responsible of different phenotypic consequences. This is the goal of this first `OpenOmics` application.
This application the will let the users to store genotypic, phenotypic and environmental data about themselves (in the case of individual users) or about other people authorizing their publication (in case of corporate users, like hospitals) and perform searches over the global dataset, including data contributed by others. Strict semantics will be used in order to have a curated, comparable dataset. The system will allow novel advanced search capabilities, not available in pure P2P systems today, like faceted search and taxonomy of result sets, or range queries; very useful for pattern identification over the dataset.
The schema used for this purpose is referred furthermore as _[Population schema](#Population_schema.md)_.
### Genotype-based databases unification ###
This second application is aimed at storing actual knowledge (findings, conclusions, studies, etc.) relative to certain human genotypes, for example publications from [dbSNP](https://www.ncbi.nlm.nih.gov/SNP/), [ICGC](http://icgc.org/), [COSMIC](http://cancer.sanger.ac.uk/), [ESP6500](http://evs.gs.washington.edu/EVS/), [DGVA](http://www.ebi.ac.uk/dgva/) etc. The system would provide a single repository of information, making much simpler the utilization of these genomic data sources for end-users (researchers and bioinformatics companies mainly) benefiting from common semantics and API, transparent updates, no download nor preprocess nor validations needed, etc.
This information could be easily integrated under a different schema, defined later as _[Knowledge schema](#Knowledge_schema.md)_.
### Addressed bottlenecks ###
`OpenOmics` addresses several bottlenecks present in bioinformatics nowadays:
  * **Unification:** Being ([perpetually](FAQ#Why_P2P?.md)) open-use and non-proprietary, all parties would feel comfortable publishing their data in this network. Thus, from a global perspective, the amount of data would be maximized and an optimal scenario for pattern recognition would be achieved.
  * **Interoperability:** Both, publishers (users storing data) and consumers (users querying data), would interact with a single repository. Private networks of `OpenOmics` could be used by corporations and hospitals through inter-connected mechanisms (see [multitier topology section](#Multitier_topology.md)).
  * **Scalability:** The amount of genomic data is expected to grow sharply in the future and `OpenOmics` would be able to grow accordingly.


---

## Overview ##
### Deterministic genotype representation ###
Determining the genotype of an individual is in itself a challenging problem. Unfortunately, nowadays it is also a non-deterministic problem, depending on the chosen sequencing technology, the uncertainty of the input, arbitrary thresholds and parameterization, alternative algorithms etc.

Genomic data is processed from a raw and difficult to exploit format (bound to the sequencing technology) into more distilled and useful formats, but at the expense of loosing fairness. Typical formats for bioinformatic pipelines in NGS analysis are: [FASTQ](http://en.wikipedia.org/wiki/FASTQ_format) -> [SAM](http://samtools.sourceforge.net/SAMv1.pdf) -> [VCF](http://www.1000genomes.org/wiki/Analysis/Variant%20Call%20Format/vcf-variant-call-format-version-41).

Our current vision is that `OpenOmics` must deal with a deterministic genotype representation, despite knowing that determinism is impossible to achieve nowadays. Determining the genotype is not, currently, in the scope of this project.

This deterministic genotype representation would allow a more effective and direct exploitation, given that, on the other hand, errors in genotyping, would tend to loose significance when more and more data is added given that the system is targeted to perform statistical studies. Additionally, the [collaborative features](#Collaboration.md) of the network, would lead to more and more curated dataset.

In section _[Record structure](#Record_structure.md)_ we explain some ideas for a new genotype representation, based on a reference sequence, and allowing the specification of both differences (variants) and equalities (matches) of the sample genotype to that reference sequence, in a lightweight manner.

### Network ###
`OpenOmics` is thought as a public global network of collaborating private nodes or **peers**; nodes belonging to different parties (individuals or organizations) and yielding local storage in order to create a distributed and decentralized database of genomic-based records with a well defined [structure](#Record_structure.md).

Peers would act on behalf of an **user**. Users of the system could add, delete, and query information (see [user API section](#User_API.md)) through their local peer. Each user and peer would have a verifiable [reputation](#Reputation.md).

Peers would be publicly responsible of the storage of a fraction (region) of global dataset ([partitioning](#Key_partitioning.md)).

On start-up a peer would get a list of active peers (given by at least one active previous known peer) and the fraction of the dataset each peer is storing. Then the peer would ask those other peers, with regions intersecting its own, for updates. After that, with a ([pseudo](#Eventual_consistency.md)) fresh version of the data, the peer would be operative.

On a user search, the peer would identify the subset the known peers, storing data potentially being in the result set of the search. Target peers would be chosen from this subset according complex criteria ([RTT](http://en.wikipedia.org/wiki/Round-trip_delay_time), [hop distance](http://en.wikipedia.org/wiki/Hop_(networking)), peer activity, peer capacity, redundancy, ...). Then, subqueries would be sent to those target peers and their responses aggregated into the final single response (see [federated search section](#Federated_search.md)). Redundant subqueries should be forwarded in order to detect inconsistencies and possible malicious peers.

On a user adding action (and removing), the local peer would proceed in a similar way, sending the new records to the known proper peers. Receiving peers would verify data integrity and authorship (see [identity section](#Identity.md)), update their local indexes, and continue propagating the data to other interested peers unknown by the original sender.


---

## Requirements ##
### Query Model ###
### Identity ###
Identity of the participants is a key of system. Each user could have an permanent identity, with a reputation bound to it.
All interaction with other peers should be done under the credentials of the user, possibly affecting this reputation, and all the content added to the dataset should be also associated to that identity, Also the integrity of the data should be verifiable.

[Public-key cryptography](http://en.wikipedia.org/wiki/Public-key_cryptography) could be used for this purpose. A [public-key infrastructure](http://en.wikipedia.org/wiki/Public-key_infrastructure) should be created, over the peers to store inmutable public keys of the users, with a simple put/get API. put() only valid to register new indentities.

### Collaboration ###
#### Reputation ####
A reputation system is needed both for peers and users, in order to make the network stable despite of misbehavior.

Peers in the system can be selfish, malicious, or faulty. Other peers must automatically detect this behaviour and ignore the peer in last case. On the other hand users can also act in a malicious way, or make contributions of poor quality. The network must have mechanisms for users to valorate other users, and for getting another user's reputation.
#### Messaging ####
A persistent public messaging system is needed for users to collaborate.
### Eventual consistency ###
[ACID](http://en.wikipedia.org/wiki/ACID) (Atomicity, Consistency, Isolation, Durability) is a set of properties that guarantee that database transactions are processed reliably. In the context of databases, a single logical operation on the data is called a transaction. It is well known that ACID guarantees tend to have poor [availability](http://en.wikipedia.org/wiki/Availability). In order to gain availability, `Openomics` is thought to be an eventually consistent data store; that is all updates reach all replicas (nodes responsible of the storage of the updated record) eventually.


---

## Functional notes ##
### User API ###
From a user perspective the system could provide this simple API:
  * Add record
  * Delete record
  * Query records
### Record structure ###
A vital element of the network will involve defining the format to be used when describing the genomic profile of a sample. This aspect has already generated some debate as if it is important or not to demand a precise record structure, the debate will probably continue but it has become clear that minimum requirements must be imposed in order to address the bottleneck that this project attempts to confront.
As stated before and in order to open this section to further discussion an elementary schema is here proposed for representing the genotype in a deterministic way, followed by an additional section, depending on the schema, with other optional fields that could inform about the age, gender, phenotypic traits, diseases, locale, drug responses etc.

[JSON](http://en.wikipedia.org/wiki/JSON) representation is chosen for explanation purposes, given its readability. Record serialization mechanisms must be analyzed and discussed in depth further on.

#### Genotype section ####
Fields:
  * **refVersion:**  Reference version; version of the assembly defined by http://www.ncbi.nlm.nih.gov/projects/genome/assembly/grc/
  * **seqId:** Sequence id, for example chromosome or contig name, as defined by the assembly.
  * **start:** First nucleotide position referred, in the reference sequence.
  * **end:** End nucleotide position referred, in the reference sequence.
  * **ref:** Optional field. Genotype in the reference sequence between start and end (inclusive). Useful for validation purposes.
  * **alt:** Multipurpose field representing the variant, match, or unknown information.

For example, this genotype representation is capable of describing:
  * **Variant regions**
    * Simple point mutations, predominantly [SNPs](http://en.wikipedia.org/wiki/Single-nucleotide_polymorphism) and [INDELs](http://en.wikipedia.org/wiki/Indel):
```
{
  "refVersion": "GRCh37",
  "seqId": "Chr1",
  "start": 156514,
  "end": 156514,
  "ref": "T",
  "alt": "CA"
}
```
    * Gross deletion:
```
{
  "refVersion": "GRCh37",
  "seqId": "Chr1",
  "start": 156514,
  "end": 556514,
  "alt": "-"
}
```
    * Chromosomal rearrangements:
```
{
  "refVersion": "GRCh37",
  "seqId": "Chr1",
  "start": 156514,
  "end": 556514,
  "alt": "Chr2:505651-605651"
}
```
    * [CNVs](http://en.wikipedia.org/wiki/Copy-number_variation):
```
{
  "refVersion": "GRCh37",
  "seqId": "Chr1",
  "start": 156514,
  "end": 156516,
  "alt": "4>50"
}
```
> > Note: The '4' part of the _alt_ property value (copy number in the sequence reference) is redundant but it's useful for validation purposes.
  * **Matches (Reference compliant regions):**
```
{
  "refVersion": "GRCh37",
  "seqId": "Chr1",
  "start": 156514,
  "end": 156516,
  "alt": "."
}
```
  * **Omitted regions (Regions with unkown genotype):**
```
{
  "refVersion": "GRCh37",
  "seqId": "Chr1",
  "start": 156514,
  "end": 156516,
  "alt": "?"
}
```

This representation introduces several advantages:
  1. Allows to represent not only variants (differences in genotype to the reference sequence), but also regions matching the reference, and unknown regions. This is extremely important in order to compute genotype frecuencies for a locus.
  1. Representing genotype per region, allowing to represent large or gross variants and to collapse contiguous matching or omitted positions, with very little penalization on size.
  1. Perfectly suits the kind of [queries](#Query_Model.md) desired for the system
<a href='Hidden comment: 
With the additional purpose of defining two additional categories that have been underrated by some formats:
This two additional categories will permit discriminating those genomic intervals that have been sequenced and comply with the reference genome:
From those regions where no variation can be established owing to a lack of information:
'></a>
#### Context ####
  * **User id:** Uploader user id
  * **Sample id:** Id of the sample (aggregation of records)
  * **Record id:** Autoincremental integer for each sample
  * **Type:** Type of sample (germinal/somatic)
#### Population schema ####
  * **context:**
  * **genotype:**
  * **phenotype:**
  * **environmetalInfo:**
  * **digitalSignature:**
Example of _Population schema_:
```
{
  "type": "object",
  "properties": {
    "context": {
      "type": "object",
      "properties": {
        "recordId": {
          "type": "integer"
        },
        "userId": {
          "type": "string"
        },
        "type": {
          "type": "string",
          "enum": [
            "germinal",
            "somatic"
          ]
        },
        "sampleId": {
          "type": "string"
        }
      }
    },
    "genotype": {
      "type": "object",
      "properties": {
        "refVersion": {
          "type": "string",
          "enum": [
            "GRCh37",
            "GRCh38"
          ]
        },
        "ref": {
          "type": "string"
        },
        "start": {
          "type": "integer"
        },
        "alt": {
          "type": "string"
        },
        "seqId": {
          "type": "string",
          "enum": [
            "Chr1",
            "Chr2",
            "Chr3",
            "Chr4",
            "Chr5",
            "Chr6",
            "Chr7",
            "Chr8",
            "Chr9",
            "Chr10",
            "Chr11",
            "Chr12",
            "Chr13",
            "Chr14",
            "Chr15",
            "Chr16",
            "Chr17",
            "Chr18",
            "Chr19",
            "Chr20",
            "Chr21",
            "Chr22",
            "ChrMT",
            "ChrX",
            "ChrY"
          ]
        },
        "end": {
          "type": "integer"
        }
      }
    },
    "phenotype": {
      "type": "object",
      "properties": {
        "drugResponses": {
          "type": "array"
        },
        "traits": {
          "type": "array",
          "items": {
            "type": "any"
          }
        },
        "diseases": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "snomedCTId": {
                "type": "string"
              },
              "age": {
                "type": "integer"
              }
            }
          }
        }
      }
    },
    "environmetalInfo": {
      "type": "any"
    },
    "digitalSignature": {
      "type": "any"
    }   
  }
}
```
#### Knowledge schema ####
  * **context:**
  * **genotype:**
  * **knowledge:**
  * **digitalSignature:**

Example of _Knowledge schema_:
```
{
  "type": "object",
  "properties": {
    "context": {
      "type": "object",
      "properties": {
        "recordId": {
          "type": "integer"
        },
        "userId": {
          "type": "string"
        },
        "type": {
          "type": "string",
          "enum": [
            "germinal",
            "somatic"
          ]
        },
        "sampleId": {
          "type": "string"
        }
      }
    },
    "genotype": {
      "type": "object",
      "properties": {
        "refVersion": {
          "type": "string",
          "enum": [
            "GRCh37",
            "GRCh38"
          ]
        },
        "ref": {
          "type": "string"
        },
        "start": {
          "type": "integer"
        },
        "alt": {
          "type": "string"
        },
        "seqId": {
          "type": "string",
          "enum": [
            "Chr1",
            "Chr2",
            "Chr3",
            "Chr4",
            "Chr5",
            "Chr6",
            "Chr7",
            "Chr8",
            "Chr9",
            "Chr10",
            "Chr11",
            "Chr12",
            "Chr13",
            "Chr14",
            "Chr15",
            "Chr16",
            "Chr17",
            "Chr18",
            "Chr19",
            "Chr20",
            "Chr21",
            "Chr22",
            "ChrMT",
            "ChrX",
            "ChrY"
          ]
        },
        "end": {
          "type": "integer"
        }
      }
    },
    "knowledge": {
      "type": "object",
      "properties": {
        "diseaseAssociations": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "publications": {
                "type": "array",
                "items": {
                  "type": "string"
                }
              },
              "level": {
                "type": "string",
                "enum": [
                  "causing",
                  "related",
                  "reducing"
                ]
              },
              "target": {
                "type": "object",
                "properties": {
                  "snomedCTId": {
                    "type": "string"
                  },
                  "age": {
                    "type": "integer"
                  }
                }
              }
            }
          }
        },
        "traitAssociations": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "publications": {
                "type": "array",
                "items": {
                  "type": "string"
                }
              },
              "level": {
                "type": "string",
                "enum": [
                  "causing",
                  "related",
                  "reducing"
                ]
              },
              "target": {
                "type": "any"
              }
            }
          }
        }
      }
    },
    "digitalSignature": {
      "type": "any"
    }
  }
}
```


---

## Architectural notes ##
### Multilayer design ###
We are starting to indentify some important concerns at the logical level. Here is the current image of the system we have now. The reading is: despite of being focused on a very specific [goal](#Goal.md), the system could be extended in the future to support new types of applications, according to the needs of the [Global Alliance](FAQ#What_about_the_Global_Alliance?.md).

![http://wiki.openomics.googlecode.com/git/layers.png](http://wiki.openomics.googlecode.com/git/layers.png)

### Multitier topology ###
Despite the motivation of the project is to create a public, collaborative network (see the [Manifesto](Manifesto.md)), a multitier global architecture can be created to support different levels of data privacy.

**Public network**

The public network corresponds with the top tier in the diagram. This public network is aimed to store:
  1. Public information associated to DNA genotypes. (See [Genotype-based databases unification](#Genotype-based_databases_unification.md))
  1. Personal information about genotype and phenotype, to be uploaded by the data owner or a authorized third party. (See [Population database application](#Population_database_application.md))
![http://wiki.openomics.googlecode.com/git/network.gif](http://wiki.openomics.googlecode.com/git/network.gif)

**Private networks**

`OpenOmics` should also be deployable on private networks (lower levels in the diagram), allowing the storage of data under stronger privacy constraints (hospitals, research laboratories, confederations, etc).
Mechanisms for connecting those networks should also be developed, allowing data to eventually emerge to the public tier.

Specific details of these private infrastructures depend on local regulations and particular organization issues and are out of the scope of the project. These aspects are being addressed by the [Global Alliance initiative](FAQ#What_about_the_Global_Alliance?.md).

### Federated search ###
http://en.wikipedia.org/wiki/Federated_search
### Key partitioning ###
### Data versioning ###
## Indentified problems ##
## References ##