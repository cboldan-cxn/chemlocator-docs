# Upgrade

This document details the process for upgrading an _existing_ ChemLocator 
docker installation.  If you have not installed ChemLocator yet, the 
[installation guide](../installation/installation-overview.md) is appropriate.

## Step 1 - Connect to Docker host and find ChemLocator installation

Connect to the host machine where the ChemLocator installation is located, using 
`ssh`.  Next, find the folder where ChemLocator is installed.


!!! note
    We typically recommend `/usr/share/chemaxon/chemlocator/` as the default folder.
    In case there are multiple environments hosted (e.g. production, testing, etc),
    there may be further subfolders.  The folder we are looking for is the one which
    contains the `docker-compose.yml`, for the environment we want to upgrade. 

Change into this folder via the `cd` command.

All of the following steps assume you are in this folder.


## Step 2 - Docker login to chemlocator.azurecr.io

The ChemLocator repository is located at `chemlocator.azurecr.io`.  By performing
a `docker login` to this repository, you will be able to pull the ChemLocator
docker images.  To start the login, use the following command:

```bash
docker login chemlocator.azurecr.io
```

The ChemLocator team should be able to provide you with a username and password.


## Step 3 - Stop the ChemLocator containers

Stop the ChemLocator containers using the following command:

```bash 
sudo docker-compose stop
```

Warning: do NOT use `docker-compose down`, as this results in some configuration
being lost.


## Step 4 - Update version number in docker-compose.yml

Note: this step might be optional for development environments.

Edit the `docker-compose.yml` file with any text editor.  Version number you are
currently using will appear in two places, in both cases following the `image` 
node, just under the `chemlocator` and `chemlocator-java` nodes, respectively
(see examples below).  Both of these needs to be updated to the latest version 
number, which should have been provided to you, and will be of the format 
`v1.2.3.4`.  As an example, changing form version `v0.9.2.0` to `v1.0.0.0`:

**Before the change:**

```yaml
  chemlocator:
    image: chemlocator.azurecr.io/chemaxon/chemlocator:v0.9.2.0
```

and

```yaml
  chemlocator-java:
    image: chemlocator.azurecr.io/chemaxon/chemlocator-java:v0.9.2.0
```

**After the change:**

```yaml
  chemlocator:
    image: chemlocator.azurecr.io/chemaxon/chemlocator:v1.0.0.0
```

and

```yaml
  chemlocator-java:
    image: chemlocator.azurecr.io/chemaxon/chemlocator-java:v1.0.0.0
```

Note: if you are going to be using CLiDE, you should also perform this process
for the line with the `chemlocator-java-clide` image, instead of 
`chemlocator-java`.


## Step 5 - Docker-compose pull to update images

You can use the `docker-compose pull` command to update the images mentioned in
the `docker-compose.yml` file.  

```bash 
sudo docker-compose pull
```

Docker will go through each of the images mentioned in the file, and pull the
latest version of the images specified. 


## Step 6 - Verify ChemLocator versions

This step is optional, and can be skipped.

Since we are going to have a lot of console logs output, it is recommended
that you clear your console via the `clear` command.

Start up the ChemLocator containers as a foreground process as follows:

```bash 
sudo docker-compose up
```

There are two checks we can make to see that the images have been updated:

 - In the beginning of the console logs (you will probably need to scroll up
   several screens worth), you should see messages containing the word 
   "recreating", applied to the containers `chemlocator` and `chemlocator-java`.
   This is the first indication that an upgrade of images has been detected by
   Docker, and has been performed.
 - Open the ChemLocator UI and log in.  In the top right, you should see your
   username.  Clicking it brings up a sub-menu - click "Profile" here.  One
   of the labels on this screen contains the version number, which should match
   what is in your `docker-compose.yml` file.
   
In preparation for starting ChemLocator as a background process in the next step,
first stop it by pressing Ctrl+C in the console, and waiting for all the 
containers to stop.


## Step 7 - Start the ChemLocator images as a background process
 
 Start `docker-compose` with the `-d` flag, to indicate that it should run in 
 the background:
 
```bash 
sudo docker-compose up -d
```
