# Prerequisite: Docker

ChemLocator is shipped as a set of Docker images, in order to minimise 
installation complexity, and to guarantee consistency.

!!! info "What is Docker?"
    While Docker has become quite mainstream recently, it is still relatively 
    normal not to have any experience with it, hence a short introduction:
    
    Docker is a tool which allows applications to be packaged complete with all
    their dependencies into *images*.  These images are little bit like 
    lightweight snapshots of an OS, the application, and any extra libraries it
    might need.  An image is used to create a *container*, which is quite similar
    to a VM, with several important distinctions (such as containers not 
    reserving large amounts of unused memory for themselves, unlike VMs).
     

## Docker version

It is recommended that a relatively recent version of the Docker Engine is used 
to run ChemLocator containers.  At the time of writing, ChemLocator images are 
built using version `19.03.8`.

You can test the version of Docker installed by running:

```bash
docker --version
```

## Docker-compose

A typical ChemLocator installation requires multiple Docker images to work 
together.  While this can be set up with manual Docker commands, we provide an
example `docker-compose.yml` file which describes and connects the required images
in a correct manner.  Some installations of Docker do not ship with the 
`docker-compose` tool by default, and it needs to be installed separately.  You 
should ensure that is installed.  For Ubuntu, the installation is as follows:

```bash
sudo apt install docker-compose
```



### Troubleshooting possible 

In some cases, it is possible for the `docker login` command to no longer function
correctly after the installation of `docker-compose`.  If you recieve the following
error message after a login attempt:

> Error saving credentials: error storing credentials - err: exit status 1, out: 'Cannot autolaunch D-Bus without X11 $DISPLAY'

Then it is possible that the Docker installation needs to be fixed.  In our experience
the following command tends to fix the above issue: 

```
sudo apt install gnupg2 pass
```

Alternatively, searching for the error on the internet provides more in depth 
discussions and solutions.


## Host operating system

While Docker itself can be installed on Windows, Mac OS, and Linux, it is really
a Linux native technology.  The versions of Docker for Windows and Mac OS work by
making use of a single Linux Virtual Machine instance to provide the native 
functionality needed, which is less than ideal.  This is perfectly fine for
demo environments, but **for production environments, a host machine running 
Linux is recommended.**


## Linux distribution selection, and installing Docker

Docker is well supported on just about all Linux distributions, and you are 
free to choose your organisation's favourite distro.  If you do not have a 
preferred choice, the ChemLocator team recommends either CentOS, or Ubuntu.

Official guides can be found on [docs.docker.com](https://docs.docker.com):

 - [Install on CentOS](https://docs.docker.com/engine/install/centos/)
 - [Install on Debian](https://docs.docker.com/engine/install/debian/)
 - [Install on Fedora](https://docs.docker.com/engine/install/fedora/)
 - [Install on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)


## Increasing memory and CPU limits

By default, a Linux-based Docker installation will not place any limits on CPU
and memory usage.  There is no further action needed.

If you re running a ChemLocator environment using either the Docker Desktop for
Windows or Mac OS, there are limits imposed by default, which cause a number of 
problems.  (To repeat the above advice: it is **not** recommended to run 
ChemLocator on a Windows or Mac OS host in production environments.)  Navigate
to the `Resources` section of the settings, and increase the CPU, memory, and 
disk space allocations as appropriate.


## Setting Docker to start up automatically

On `systemd`-based distributions such as Ubuntu, the docker service can be set
to start automatically if the server is rebooted.  This is done as follows:

```bash
sudo systemctl enable docker
```

The first time after an installation, it may be necessary to start docker:

```bash
sudo systemctl start docker
```

