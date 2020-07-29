# Uninstallation

!!! warning
    This process is not reversible without backups!
    
## Uninstalling a single-host installation with `docker-compose`

Connect to the host machine where ChemLocator is installed using `ssh`.

Change into the folder where the ChemLocator installation is, using `cd`.  The 
installation recommends using `/usr/share/chemaxon/chemlocator`.  To ensure that
the folder is correct, check that a file called `docker-compose.yml` exists.

To completely tear down the environment and remove any Docker volumes created, 
use the following command (prefix with `sudo ` on Linux distributions which 
make use of it, such as Debian or Ubuntu):

```bash
docker-compose down -v
```

To finalise uninstallation, delete the installation folder:

```bash
rm -rf /usr/share/chemaxon/chemlocator
```

!!! warning
    Be very careful with `rm -rf` commands, as they silently and recursively 
    delete the target folder.  Ensure that the path you provide is the correct 
    one.
    
    
## Uninstalling a multi-host installation
 
The process is identical to the single-host process above.  For each of the 
hosts used, repeat the process.
