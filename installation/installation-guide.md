# Installation Guide

This guide walks you through creating a new ChemLocator environment.

## Prerequisites

The following items are required 

 - A host machine with an up-to-date installation of Docker.  The 
   [hardware requirements](hardware-requirements.md) section describes 
   selecting appropriate hardware.
 - The version number of the latest ChemLocator and JPC Docker images.
 - A valid ChemLocator license.


## Step 1 - Create a ChemAxon Pass

A ChemAxon Pass is your login and access to download ChemAxon product installers.
It is required for the ChemLocator installation.

Navigate to [https://chemaxon.com/](https://chemaxon.com/) and click the `log in`
item in the navigation menu, and then `create account`.  Complete the form, and
click register.  Your email address will be the username used to log into the
Docker repository in later steps.

An activation email will be sent, and the `Verify Your Pass` link needs to be
clicked in order to finalise the registration.


## Step 2 - Generate an API key

An API Key will serve as the password in later steps, when logging in to the
Docker repository.  To set it up, navigate to [https://accounts.chemaxon.com/my/settings](https://accounts.chemaxon.com/my/settings), and click the `Setup my API key` button.

A random key will be generated, which you should make a copy of (as it will
be required during later steps).


## Step 3 - Docker login to hub.chemaxon.com

The ChemLocator repository is located at `hub.chemaxon.com`.  By performing
a `docker login` to this repository, you will be able to pull the ChemLocator
docker images.  You can use the following command to start the login process:

```bash
docker login hub.chemaxon.com
```

Docker will ask for your username and password.  The username should be your
email address from step 1, and the password is the API key from step 2.


### Non-interactive Docker login methods

The above login method may not be appropriate for automated scripts.  The 
following command will perform the login without user interaction:

```
echo <API-KEY> | docker login -u <E-MAIL> --password-stdin hub.chemaxon.com
```

Alternatively, the `~/.docker/config.json` can be pre-populated with the 
following:

```
{
  "auths": {
    "hub.chemaxon.com": {
      "auth": <BASE64(<E-MAIL>:<API-KEY>)>                                             
    }
  }
}
```

(Where the value for "auth" is the result of Base64-encoding the email, 
concatenated to `:` and the api key).


## Step 4 - Create a folder for the docker-compose file

From this point forward, all the instructions are to be run within the context
of the docker host machine.  After the end of this step, it will be assumed 
that you run subsequent commands from within the folder created in this step.

In the following steps, we create a `docker-compose.yml` file, which specifies
a group of Docker images, which form the ChemLocator ecosystem.  This file 
essentially represents a single ChemLocator installation (or environment), and
should be located in a folder which describes the environment.  For installations
where only a single ChemLocator environment (e.g. production) will be located,
we recommend using the path `/usr/share/chemaxon/chemlocator/`.  If you wish to
install multiple environments on the same host (e.g. test and pilot), we 
recommend creating sub-folders with short but descriptive names for each 
environment, e.g. `/usr/share/chemaxon/chemlocator/cl-test`.  

You can create and switch to this folder as follows:

```bash
sudo mkdir -p /usr/share/chemaxon/chemlocator/
cd /usr/share/chemaxon/chemlocator/
```

## Step 5 - Deploy docker-compose.yml and related files

The ChemLocator team can provide you with an appropriate example environment 
with valid `docker-compose.yml` and `.env` files, which are a good starting 
point for a new environment.  Extract these files into the folder created
during step 2 (typically: `/usr/share/chemaxon/chemlocator/`).  


## Step 6 - Update .env file

The `.env` file contains most of the settings required for ChemLocator.  The
main settings which need to be set are:

- `CL_VERSION` represents the version of ChemLocator installed, e.g. `3.2.0.7`.
- `JPC_VERSION` represents the version of the JChem for PostgreSQL Cartride that
  is used, e.g. `20.15.0.r12003`.
- `CL_PATH` is the full installation folder, e.g. `/usr/share/chemaxon/chemlocator`.  
  It is important that this path **not** have a trailing slash.
- `CL_PORT` is the port number on which the ChemLocator UI will be exposed, on the
  host machine.
- `CL_AUTHENTICATIONTYPE` sets the authentication provider type for ChemLocator.
  Valid choices are `ldap` and `azuread` (`synergy` being a reserved provider 
  for ChemAxon-hosted environments).  The `CL_AUTH_ENV_FILE` setting should also
  be set to the relevant `.env` file for your authentication provider.  
- `CL_JAVA_IMAGE` determines whether an image including CLiDE is used.  If you
  have a CLiDE license, and would like to make use of it in combination with 
  ChemLocator, change this setting to `chemlocator-java-clide` and set the
  `CL_CLIDE_KEY` value to your actual CLiDE key (which will have to be a 
  valid key for CLiDE Batch for Linux).


## Step 7 - Configure your authentication provider

Depending on your authentication provider of choice, you will need to both
configure your environment (e.g. your Azure AD settings themselves), as
well as update the relevant `.env` file (e.g. `azure.env`) with the 
required values. 


## Step 8 - Provide a `license.cxl` file

The `license.cxl` file provided in the demo template is empty (the empty file 
exists in order to prevent problems if docker-compose were to be started without 
this file being in place).

Replace this file with a real license file.


## Step 9 - Start up ChemLocator

At this point, the ChemLocator environment can be started up for the first time.
You can start it with the following command:

```bash
sudo docker-compose up
```
 
During the first start up, Docker will pull the relevant images from the public
DockerHub repository as well as the private ChemLocator repository.  

Additional tips - if you started the environment using the above command, all 
logs will be output to the console mode, and the environment will be stopped if
you press Ctrl+C, or exit the terminal session.  This is suitable for 
troubleshooting, but for production use, you should start it with the `-d` flag,
i.e. `sudo docker-compose up -d`.  This will start the containers as a background
process, and won't tie their lifecycle to the terminal.  In this case, you can
use `sudo docker-compose stop` to stop the containers.  Note: you should NOT 
use `docker-compose down`, as this will remove some of the configuration 
information from the environment.,


## Step 10 - Initial configuration of ChemLocator

ChemLocator should now successfully be running at `http://localhost:3050`.  If 
you are using the default LDAP service, you can log in with username: `admin` 
and password: `admin`.

ChemLocator now needs to be configured:

  - The ElasticSearch API url needs to be configured under Admin / Settings / 
  Freetext settings.  Under `Server Node URLs`, add `http://elasticsearch:9200`
  (or the relevant address, if you are using your own Elasticsearch instance), 
  and click the plus sign to add it to the list.  Click `Validate Settings`, and
  assuming it's successful, `Save`.  At this point, you should have a working, 
  empty copy of ChemLocator.
  - Add Content Sources for ChemLocator to index.  You will most likely want to
  add Cloud Credentials first, in order to be able to access your data in the
  cloud.
