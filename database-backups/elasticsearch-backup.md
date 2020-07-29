# Elasticsearch backup

This document contains background information and examples about how a 
Dockerized ChemLocator Elasticsearch installation can be backed up.

If you are using a non-Dockerized installation of ElasticSearch, you can ignore
the Docker-related parts, however the core backup and restore commands would
essentially stay the same.  The rest of the guide assumes a Dockerized 
installation.

!!! warning
    This document contains some explanations and examples about backing up a
    ChemLocator ElasticSearch database.  It is important to note that this guide
    is provided as **information only**.  Ensuring that backups of your 
    environment and databases are done regularly and are working correctly is
    the sole **responsibility of the user**.  
   
   
The [official Elasticsearch guide for snapshots](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html) 
should be referenced, and any considerations for production environments be taken
into consideration.

## Backup strategy and prerequisites
 
In this guide, we will be using the standard REST API of Elasticsearch to create
a snapshot (and to restore it).  For this to work, the following tools and 
conditions are required:

  - A backup folder to be host-mapped from the Docker server to the Elasticsearch
    container.  The provided `docker-compose.yml` should already contain this.
  - The above folder (inside the container) to be specified as a repository 
    path in the `elasticsearch.yml` configuration file of Elasticsearch.  This
    should also be provided by the default docker-compose template.
  - Port 9200 of the Elasticsearch server to be reachable (usually via port 
    mapping on the host Docker server - provided in the template).
  - The `curl` tool to be installed, for making HTTP calls to the Elasticsearch
    API.  In our guide, this should be installed on the Docker host, and therefore
    cannot be included in our templates.
    
In case you have configured ChemLocator's docker containers yourself from scratch,
some additional information will be provided for the first three points.

**Host-mapped backup folder:** in order to make it easy to copy the resulting 
backups to a separate location, we recommend creating a folder on the Docker host,
and mapping this into the Elasticsearch container.  In our templates, we default
to having a backup folder called `es_backups` next to the `docker-compose.yml`, 
and mapping this into the container by adding the following line under the 
Elasticsearch service's volumes section:

```
      - ${CL_PATH}/es_backups:/es_backups
``` 

This will ensure that from within the container `/es_backups` is available as
a folder, and actually writes to the Docker host's folder.

Lastly, in order to be able to reach the Elasticsearch API, the port mapping to
the host machine should be enabled.  This is achieved as follows:

```
    ports:
      - "${ES_PORT:-9200}:9200"
```

This will forward port 9200 on the Docker host to port 9200 on the Elasticsearch
container, making it addressable as `http://localhost:9220`. 

!!! warning
    Please ensure that you place firewall rules on the Docker host to ensure
    that this port cannot be accessed from outside of the server.
    
 
## First-time configuration of the backup repository

Before a backup can be created, an Elasticsearch repository pointed to the 
above backup folder needs to be created.  This only needs to be done once, and
should persist unless the containers are torn down

To check whether the repository already exists, run:

```bash
curl -XGET 'http://localhost:9200/_snapshot/es_backups' 
```

If the repository exists, a response similar to the following will be shown:

```json
{"es_backups":{"type":"fs","settings":{"compress":"true","location":"/es_backups"}}}   
```

If the repository does not exist, the response would be:

```json
{"error":{"root_cause":[{"type":"repository_missing_exception","reason":"[es_backups] missing"}],"type":"repository_missing_exception","reason":"[es_backups] missing"},"status":404}
```

In order to create the repository, issue the following command:
 
```bash
curl -XPUT -H "content-type:application/json" 'http://localhost:9200/_snapshot/es_backups' -d '{"type":"fs","settings":{"location":"/es_backups","compress":true}}'
```

A success status should be returned, and the previous `GET` command can be 
repeated to confirm that the repository has been created.


## Creating a snapshot

!!! warning
    Before making a backup, please ensure that the target location (in this case
    the Docker host) has more than enough free space to accomodate the new 
    backup.  Running out of space on the host machine is a common problem.


A snapshot (in this case with the name `first-snapshot` can be created using the
following command:

```bash
curl -XPUT 'http://localhost:9200/_snapshot/es_backups/first-snapshot?wait_for_completion=true'
```
The time taken to create the backup will depend on the quantity of data in the
index.  Once successful, a message as follows will be returned:

```json
{"snapshot":{"snapshot":"first-snapshot","uuid":"xu_H_qDbRVeEVTZGVraxwQ","version_id":7060199,"version":"7.6.1","indices"["chemlocator_freetext"],"include_global_state":true,"state":"SUCCESS","start_time":"2020-06-22T14:14:32.368Z","start_time_in_millis":1592835272368,"end_time":"2020-06-22T14:14:32.368Z","end_time_in_millis":1592835272368,"duration_in_millis":0,"failures":[],"shards":{"total":1,"failed":0,"successful":1}}}
```

The repositories containing snapshots can be listed with the following command:

```bash
curl -XGET 'http://localhost:9200/_snapshot/_all?pretty'
```

The specific snapshots contained within the `es_backups` repository can then be
shown via:

```bash
curl -XGET 'http://localhost:9200/_snapshot/es_backups/_all?pretty'
```

As expected, the results are returned in JSON.


## Restoring a snapshot

The snapshot created in the previous section can be restored with the following
command:

```bash
curl -XPOST 'http://localhost:9200/_snapshot/es_backups/first-snapshot/_restore?wait_for_completion=true'
```

Please note that all the indexes in the snapshot will be restored.  The restore
process will also fail if there are any existing indexes with the same names,
and these have to be deleted manually.

!!! warning 
    Please ensure you have a backup or are sure you wish to delete the indexes,
    before issuing the following command!
    
All the indexes on the Elasticsearch container can be deleted with the following
command:
 
```bash
curl -XDELETE 'http://localhost:9200/_all'
```


