# Database backups overview

This section provides information about backing up the ChemLocator databases.
    
!!! warning
    The sections on database backups are provided as examples.  While we always
    try to ensure that our documentation is up to date and correct, it is 
    ultimately the responsibility of the client to ensure that regular backups
    are made, verified, and that the processes for restoring them are well 
    understood.
    
    Please read the "Responsibility for the database backups" section below 
    carefully.
    
    
## Responsibility for the database backups

Any plan for operating an enterprise system should include mechanisms for 
ensuring the backup of any databases involved.  With ChemLocator, the databases
involved are the PostgreSQL / JPC main database (which contains the core 
chemical index) and the ElasticSearch repository (which contains the free text
index).

While the ChemLocator team strives to provide good information, suggested 
practices, and assistance, it is ultimately the sole responsibility of the client
to ensure that their environment is operated in a safe manner.  From the 
perspective of database integrity, a good plan for hosting ChemLocator needs to
include:

  - A **mechanism for backing up the PostgreSQL database**.
  - A **mechanism for backing up the ElasticSearch repository**.
  - Ensuring that these backups happen at **regular intervals**, and before any 
    major changes, such as version upgrades.
  - Preferably, the **PostgreSQL and ElasticSearch backups should occur at the 
    same time**, since in the event of a problem they would need to be restored 
    together, and having backups from different periods would risk an 
    inconsistency
  - It is highly preferable that backups take place while there is **no active
    indexing** happening in ChemLocator.
  - Since backups can become invalid due to software errors or storage issues,
    it is a best practice to have a process in place to verify the validity
    of backups, for mission critical systems.
  - It is highly recommended that the restor process be tested to ensure both
    that the process works as intended, and that IT staff would be able to 
    respond quickly to a failure. 


## ChemLocator databases
 
Your indexed data is stored in two primary places within ChemLocator: the 
PostgreSQL/JPC database, and the Elasticsearch indexes.  These are the primary
entities which need to be backed up, in order to ensure that a recovery can be
made in the event of a server failure.
 
The next two sections cover the backup process for each of these in detail.


