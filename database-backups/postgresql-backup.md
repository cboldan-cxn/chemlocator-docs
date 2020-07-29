# PostgreSQL backups

This document contains background information and examples about how a 
Dockerized ChemLocator PostgreSQL installation can be backed up.

If you are using a non-Dockerized installation of PostgreSQL, you can ignore
the Docker-related parts, however the core backup and restore commands would
essentially stay the same.

!!! warning
    This document contains some explanations and examples about backing up a
    ChemLocator PostgreSQL database.  It is important to note that this guide
    is provided as **information only**.  Ensuring that backups of your 
    environment and databases are done regularly and are working correctly is
    the sole **responsibility of the user**.  
    

## Background information

In this section, we'll look at some background information about how
the `jpc-chemlocator` image works, and some other general concepts.  These
include the location of backups, identifying the database container, changing
to the user with privileges to the database, and how to chain these commands
into a single command which can be run directly from the host machine.
 
 
### Location of backups

From within the ChemLocator-JPC container, the default backup location is 
`/psql_backups/`.  Our `docker-compose.yml` examples typically mount this 
location as a host-mapped folder from the host machine.  On the host machine, 
it can be found as a sub-folder called `psql_backups`, next to the 
`docker-compose.yml`.

Having this location as a host-mounted folder is intended to make it easier for
clients to copy the resulting backups to a separate location for long term 
storage and verification.


### Docker containers 

Most of the commands that follow in this guide are executed from the Docker host
machine, and need to specify which Docker container to target.  This can either
be done via the container's unique identifier, or its name.  

Since the identifier can change over time (e.g. as a result of an upgrade and
recreation of the container), it is recommended to use the name.  However, both
can work.

In order to find the details of the container, the following command can be run:

```bash
sudo docker image ls | grep jpc
```

Example output:
```
c52bf6b3b296        chemlocator.azurecr.io/chemaxon/chemlocator:v3.1.2                      "wait-for-it jpc:909…"   17 hours ago        Up 17 hours         0.0.0.0:3050->8080/tcp                         cl-dev_chemlocator_1
749f38ac8d22        chemlocator.azurecr.io/chemaxon/jpc-chemlocator:v5.1.r20200122.131553   "/start-log.sh"          17 hours ago        Up 17 hours         0.0.0.0:3051->5432/tcp                         cl-dev_jpc_1
```

In this case, two lines were returned, one of which is the ChemLocator server, 
and one is the PostgreSQL & JPC server.  We can see from the image name that 
the first row is the ChemLocator server, while the second row is the database
(from `jpc-chemlocator`).  This can also be verified from the port mappings 
(second last column) and the instance name (last column).  The first column
gives the unique identifier of the container.  

In this case, the correct identifier is `749f38ac8d22`, and the container name
is `cl-dev_jpc_1`.  As stated earlier, either of these can be used, but the name
is less likely to change.

The name `cl-dev_jpc_1` can also be predicted.  It consists of two parts, the 
prefix, and a suffix, separated by an underscore (`_`).  The suffix is always
`jpc_1`, and the prefix is the folder in which the `docker-compose.yml` file
is located.  In the default recommended install location 
(`/usr/share/chemaxon/chemlocator`), the folder would be `chemlocator`, making
the image name `chemlocator_jpc_1`.  This name will be assumed for the rest of
this guide, however you should verify and replace it with the relevant one - if
different - for your installation.


### The PostgreSQL user

In the `jpc-chemlocator` image, the `root` does not have direct permissions to 
manipulate the databases.  However, it can switch to the `postgres` user.  As an 
example, you can connect to the shell of the PostgreSQL container as follows:

```bash
sudo docker exec -ti cl-dev_jpc_1 bash
```

Now you would need to switch over to the `postgres` user, as follows:

```bash
su postgres
```

No password is needed for this switch.  Now, you could run a query as follows:

```bash
psql chemlocator < "SELECT * FROM molecules LIMIT 100"
```

Once we're done, we can `exit` from the `su` process, and `exit` again from the
`bash` process running in the container.  This takes us back to the host machine.


### Running queries directly from the host machine

In the previous section, we saw that we needed to shell into the PostgreSQL 
container, switch user, and then issue a command to execute a query.  We can 
also roll all these steps into one.  The last two steps - switching of the user 
and executing a query - can be rolled into one using the `su -c` command.  
As an example, once we are in a shell as root in the PostgreSQL container, we
can issue the following command:

```bash
su postgres -c 'psql chemlocator -c "SELECT * FROM molecules LIMIT 100"'
```

This would have the same effect as first performing the `su postgres` and then
separately issuing the query, but has the advantage of being shorter.  Next, 
we can roll this step and the first step into one.  From the Docker host, we 
could use `bash -c` as an `exec` parameter, as follows:

```bash
docker exec -t chemlocator_jpc_1 bash -c "su postgres -c 'psql chemlocator -c \"SELECT * FROM molecules LIMIT 100\"'"
```

!!! note
    Importantly, since the parameter after `bash -c` needs to be surrounded by
    double quotes (`"`), we need to *escape* any existing quotes with the bash
    escape character, which is the backslash (`\`).  As a result, any existing
    `"` becomes `\"`.  Less important is that we have dropped the `-i` flag, 
    as we don't expect this shell session to need to be `--interactive`.
    

## Creating a database backup

!!! warning
    Before making a backup, please ensure that the target location (in this case
    the Docker host) has more than enough free space to accomodate the new 
    backup.  Running out of space on the host machine is a common problem.


In this section, we will use the SQL Dump (`pgdump`) method.  The official 
PostgreSQL documentation on [SQL Dump](https://www.postgresql.org/docs/12/backup-dump.html) 
gives useful background information.

The following command creates a pgdump backup:

```bash
docker exec -t chemlocator_jpc_1 bash -c "su postgres -c 'pg_dump chemlocator > /psql_backups/$(date +\"%Y_%m_%d_%I_%M_%S\").bak'"
```

The resulting backup file can be found in `psql_backups`, with the current date 
and time as the file name, with the format `yyyy-mm-dd-hh-mm-ss.bak`.  

!!! note
    It is important that you schedule regular backups for your database.  
    Additionally, it is a general best practice that you ensure the integrity
    of any backups by restoring them on a separate database, checking that the 
    process succeeds, and verifying the integrity of the resulting database. 
 
 
## Restoring a database backup

Restoring a database backup consists of two steps.  Firstly, if overwriting an 
existing database, it needs to be dropped and recreated.  Finally, the restore 
needs to be executed.  The sections below describe this process.


### Recreating the database

In order to restore a SQL Dump backup, the target database already needs to 
exist, but must be empty.  With an existing ChemLocator database, this can be
achieved by dropping and recreating the database.

!!! warning
    Dropping the existing database is not reversible.  If you wish to be able to
    restore the data in it, you should ensure you have a backup.
    
Before you will be able drop the ChemLocator database, all connections will need
to be dropped.  To do this, ensure that all direct user connections - e.g. ones
from the `pgadmin` web UI - are disconnected.  Next, stop the `chemlocator` 
docker image:

```bash
sudo docker stop chemlocator_chemlocator_1
```

Once all connections are stopped, the `chemlocator` database can be dropped:

```bash
docker exec -t chemlocator_jpc_1 bash -c "dropdb chemlocator"
```

Next, a blank DB can be created in its place:

```bash
docker exec -t chemlocator_jpc_1 bash -c "createdb -T template0 chemlocator"
```

### Restoring the backup

To restore the PG Dump backup in the `chemlocator` database, run the following
command (taking care to replace the name of the backup with the correct one 
for your chosen backup file):

```bash
docker exec -t chemlocator_jpc_1 bash -c "su postgres -c 'psql chemlocator < /psql_backups/2020_06_19_06_38_46.bak'"
```

Once complete, the database should be restored to the same state as it was at the 
time the backup was created.
