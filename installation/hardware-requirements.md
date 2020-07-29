# Hardware summary

A ChemLocator installation involves several Docker containers.  Depending on 
your performance requirements and available resources, you may choose to run all
the containers on a **single host**, or split it out to **multiple hosts** to improve 
performance. 


## Single-host installation

A single-host installation includes all the core ChemLocator containers (namely:
`chemlocator` and `chemlocator-java`), as well as its dependencies (`jpc`, 
`elasticsearch`) on a single server.

Resource | Minimum | Recommended |
---------|---------|-------------|
CPU      | 4 cores | 8 cores     |
Memory   | 16 GB   | 64 GB       |
Storage  | 1 TB    | 4 TB, or as required |

Please note that the minimum requirements here are only appropriate for the 
*light* usage, and will generally not provide a good experience.

!!! tip "A note about shared hosts"
    The above recommendations assume that the host running ChemLocator is not 
    running any other, unrelated Docker containers.  While this is technically 
    possible, it is discouraged in general.



## Multiple-host installation

The main benefit of splitting the containers to multiple hosts is to allow for 
better performance overall, but also specifically to maintain good search 
performance while indexing is ongoing.

!!! tip "Why multi-host?"
    There are several good reasons for choosing multiple hosts over a single 
    one.  
    
      - Horizontal vs vertical scaling: scaling across to multiple servers can
        often be cheaper than adding the same amount of resources to a single
        server.
      - Increased performance due to less resource contention: if one service
        is fully utilising a resource's performance (such as a hard disk), any
        other service will have degraded performance.  Some types of resources
        (e.g. spinning disks) do not handle parallel load well and scale 
        sub-linearly, meaning that you are better off with dedicated resources.
      - The "noisy neighbour" problem can occur even within a single ChemLocator
        installation.  As an example, indexing is a very resource intensive 
        process, and can easily impact the performance of any searches which 
        are running at the same time.  This effect is greatly lessened on 
        multi-host installations.

The following components can be split out:


### ChemLocator

Container name: `chemlocator`, roles: UI, search, indexing coordination, OCR

Resource | Minimum | Recommended |
---------|---------|-------------|
CPU      | 2 cores  | 8 cores    |
Memory   | 4 GB     | 8 GB       |
Storage  | 100 GB   | 4 TB, or as required |


### ChemLocator-Java

Container name: `chemlocator-java`, roles: indexing, OSR, chemical intelligence 

Resource | Minimum | Recommended |
---------|---------|-------------|
CPU      | 2 cores  | 8 cores    |
Memory   | 4 GB     | 16 GB      |
Storage  | 100 GB   | 100 GB     |


### JPC

Container name: `jpc`, roles: PostgreSQL and JChem Cartridge, main database and 
search provider

Resource | Minimum | Recommended |
---------|---------|-------------|
CPU      | 4 cores  | 8 cores    |
Memory   | 8 GB     | 32 GB      |
Storage  | 1 TB     | 4 TB, or as required |


### Elasticsearch

Container name: `elasticsearch`, roles: Elasticsearch free text indexing, search

Resource | Minimum | Recommended |
---------|---------|-------------|
CPU      | 4 cores  | 8 cores    |
Memory   | 8 GB     | 32 GB      |
Storage  | 1 TB     | 4 TB, or as required |



# Further information

In this section, further information and rationale is provided.


## Hardware requirements

ChemLocator hardware requirements directly depend on the size of the chemical 
data that will be used.

Because we use additional tools to provide free text search capabilities 
alongside the chemical search, these tools also add additional hardware 
requirements.

While an "all in one server" approach works, depending on the amount of data 
that needs to be handled by the product, it is recommended that multiple servers
are used for each of the specialized roles. Even with moderate data it is 
recommended that at least PostgreSQL is installed on a different server.

Ideally, as the document source increases, multiple servers should be used that 
deal with specific parts of the product:

  - ChemLocator server - for chemical indexing and searching
  - Elasic Search server(s) - for the text mining and search
  - PostgreSQL server with JChem Cartridge for PostgreSQL - for the database


## Memory

This is one of the most demanding resources of ChemLocator and the tools we use 
and when it comes to this resource the more, the better. A machine with 16 GB 
for ChemLocator alone is the ideal memory need when no additional parts (Elastic 
search, PostgreSQL) are installed on the same machine; the same value will be enough 
if ChemLocator is configured to use Lucene instead of Elastic Search as the free 
text indexing engine, but severely limits the amount of data that can be handled 
and provide high performance during indexing and querying.

If "all in one server" approach is chosen then a minimum of 64GB will be needed 
to handled moderate amounts of data, but in this case the performance hit will 
not come from memory usage. In this case the performance will highly depend on 
the disks used by the machine and their RAID configuration (if any is provided) 
as the disk resource is the slowest component in a machine. More about disk 
resource can be read below.

!!! note "Additional information"

    Elastic search memory requirements

    PostgreSQL memory requriements: at least 4 GB and should be increased as 
    database size increases to ensure optimal performance.

## CPU

Depending on the enabled features the amount of CPU needed by ChemLocator can 
grow from moderate to high. If no extra features are enabled for the chemical 
mining and indexing process, a minimum of 4 cores may be enough for moderate 
sized document sources. As the document source size grows the number of cores 
can help but will only improve slightly, in this scenario it is recommended to 
add an additional server to split the chemical mining and indexing work load.

The extra features mentioned above that can demand more CPU usage are: OCR 
(Optical Character Recognition) and annotation.

The OCR makes use of the well known Tesseract tool maintained by Google and its 
CPU requirements depends directly on the documents that are part of the document 
source.

The annotation feature can be enabled and will allow the annotation of PDF and 
HTML files (more to come in the future) and is a costly process in terms of CPU 
(and memory alike). When enabling this feature make sure that additional cores 
(2 or 4 cores) are available for the machine.

Additional CPU requirements

    Elastic search CPU requirements

    PostgreSQL


## Disks

As mentioned before, disks are the slowest part of any machine which means that 
any software can easily reach their maximum performance. If SSD 
(Solid State Drive) can be afforded then this is the way to go because they are
superior to spinning disks; even when using SSD RAID configurations are not to 
be ignored as they can provide extra performance boost. 

If instead of SSD, spinning disk drives are used then the highest RPM disks are 
recommended and to achieve higher performance RAID configurations is a must. 

The disk can be a bottleneck because large amounts of data will be stored 
physically by PostgreSQL, Elastic Search, and even the ChemLocator processing 
will use the disks if OCR/Annotation features are enabled.


## Network

A fast network is recommended when configuring ChemLocator as a farm 
(multiple server with specialized roles) with a minimum speed of 1GbE; 
if 10GbE (Gigabit Ethernet) can be provided then this would be the ideal 
scenario. Avoid using networks that is distributed across a large geographical 
area; a 100ms latency would be ideal when dealing with large networks.

Network storage must be avoided at all costs when it comes to ChemLocator parts 
(Elastic search, PostgreSQL); this means that configuring Elastic Search and/or 
PostgreSQL to store data on a NAS must be avoided because the network access 
speeds cannot compete with the drives, even the slow spinning drives because 
network failures can happen often and even if the failure is not critical 
because the data will be eventually read, essential time is lost by 
re-transmitting data over the network.

## Cloud platforms

It is common to have products installed in cloud platforms and have virtual 
machines that can provide huge amounts of memory and CPU cores. While we do not 
recommend against this approach we prefer to recommend medium-to-large virtual 
machines when it comes to ChemLocator and for the same costs have multiple 
server that split the work load; a cluster of small machines is not recommended 
because the management of these machines can become a very difficult task and
most of the times small machines have limits sets that can interfere with the 
product (e.g. I/O operations per day).

As ChemLocator chemical mining and indexing can be configured as a cluster, 
adding multiple chemical mining and indexing machines can help speed up the 
process and having Elastic Search and PostgreSQL on different machines can improve 
search times as well.
