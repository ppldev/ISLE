# ISLE w/ Blazegraph

Based on:  
 - [Tomcat 8.5.31](https://hub.docker.com/r/benjaminrosner/isle-tomcat/)
 - Fedora Repository 3.8.1
 - [Discovery Garden's](https://www.discoverygarden.ca/) [Trippi-Sail](https://github.com/discoverygarden/trippi-sail) v.1.0
 - [Blazegraph](https://www.blazegraph.com) 2.1.4 is used for the Fedora resource Index instead of the default [Mulgara](http://mulgara.org/)

 ## Quick Start Guide

### Requirements  
* Docker-CE or EE
* Docker-compose
* Git
* Time required < 30 minutes.
* **Windows Users**: Please open the .env and uncomment `COMPOSE_CONVERT_WINDOWS_PATHS=1`


### Release Notes

Usage of Blazegraph is almost 100% similar to typical ISLE with a few exceptions.

1) **MEMORY WARNING** Please note that the `Blazegraph` container has been initially set to use between 1 -4 GB of memory. Further testing will be required to see if lower memory settings can be used. Recommend using a laptop, workstation or server that has 16 GB minimum for this type of testing and general usage.

2) **FILE GROWTH WARNING** Please note that the `Blazegraph` container uses a journal like file to log changes `/var/bigdata/bigdata.jnl`. This file can grow quickly into the double digits GB in size with large collections. Plan on allocating sufficient but performant storage space for this path. For now using local volumes is okay but in production, bind mounting to a larger performant disk would be optional.

3) Within the `.env` file, there is a setting on line 98 that is uncommented specifically for Blazegraph. By default this setting is uncommented for the ISLE `blazegraph` branch, when pushed to master, this will default back to Mulgara.

**To Do:** Testing of the new fedora:blazegraph image with the Mulgara setting uncommented (line 95)

4) There are additional Trippi-sail related jar files in the Fedora Tomcat Web app i.e. `/usr/local/tomcat/webapps/fedora/WEB-INF/lib` directory instead of creating a separate `/opt/trippi-sail` directory. Additionally there were issues with using Tomcat 7 class-path code on a Tomcat 8 server as multiple calls and methods are deprecated without easy ways forward. To fix simply, all class-paths references were backed out, they don't appear to even work on a non-ISLE Tomcat 7 system.

5) Line 101 within the `.env` file, there is an additional variable `FEDORA_WEBAPP_HOME` which was supposed to help with setting classpaths. This might be backed out in future versions.

6) This version of ISLE uses the [isle-blazegraph](https://github.com/Islandora-Collaboration-Group/isle-blazegraph) `latest`  Dockerhub image / tag which is Blazegraph version 2.1.4

### Quick Start

1. Please read: [ISLE Release Candidate (RC): How to Test](https://docs.google.com/document/d/1VUiI_bXo6SLqqUjmInVjBg3-cs40Vj7I_92txjFUoQg/edit#heading=h.1e4943m60lsh)
2. Clone the `blazegraph` branch of the ISLE repo
    - `git clone -b blazegraph https://github.com/Islandora-Collaboration-Group/ISLE.git`
3. Change directory to the cloned directory:
    - `cd ISLE` (by default)
4. Pull the latest images:
    - `docker-compose pull`
5. Launch the ISLE stack:
    - `docker-compose up -d`
6. Please wait a few moments for the stack to fully come up.  Approximately 3-5 minutes.
7. Install Islandora on the isle-apache-ld container:
    - `docker exec -it isle-apache-ld bash /utility-scripts/isle_drupal_build_tools/isle_islandora_installer.sh`
8. QC site by creating a test collection e.g. test:collection and ingesting a few different object types e.g. pdf, tiff and jpg. Change the default Drupal search block to use `Islandora Simple Search` and search for the newly ingested items to ensure Solr is displaying newly ingested objects by a `dismax` search or specific search terms. 

Please note: The ability to navigate here https://isle.localdomain/islandora/object/islandora%3Aroot and see collections also confirms that the Resource Index is getting updated properly using Blazegraph instead of Mulgara.  

9. Once objects have been ingested, double-check that there is a triple count by running the following query here: http://isle.localdomain:8084/blazegraph/#query 

Copy and paste the following:

`SELECT (COUNT(*) AS ?triples) WHERE {?s ?p ?o}`

Then press the `execute` button at the bottom of the page. There should be a triple count higher than the initial 216 or so from startup.

10. To wrap up testing:
    - In the folder with the docker-compose.yml `docker-compose down -v` (nb: the -v removes all volumes, and will delete any work. This option **does not persist your data**)

### Quick Stop and Cleanup 
If you have been testing the stack extensively you may want to `prune` your Docker daemon as you test.
1. In the folder with the `docker-compose.yml`
    - `docker-compose down -v`
- If you would like to *completely clean your docker-daemon*:
2. If you have no other _stopped_ services that you do not want `pruned` on Docker:
    - **Note running containers are NOT pruned.**
    - `docker system prune --all`
    - answer `Y` to remove all unused volumes, images, and networks.
- OR
2. If you cannot `prune`:
    - `docker ps` and take note of any running ISLE services:
        - `docker down {list of all the running ISLE services, tab auto-complete may work}` (You may add as many containers as needed in one command.)
    - `docker image ls` and take note of all ISLE-related images:
        - `docker image rm {list of all images to be removed, tab auto-complete may work}` (Again, you may add as many as needed.)
    - `docker volume ls` and take note of all ISLE-related volumes:
        - `docker volume rm {list of all volumes to be removed, tab auto-complete may work}` (Again, you may add as many as needed.)
    - `docker network ls` and take note of all ISLE-related networks:
        - `docker network rm {list of all networks to be removed, tab auto-complete may work}` (Again, you may add as many as needed.)

### Important Notes, Ports, Pages and Usernames/Passwords
[Portainer](https://portainer.io/) is a GUI for managing Docker containers. It has been built into ISLE for your convenience.  
**Windows Users**: Please open the .env and uncomment `COMPOSE_CONVERT_WINDOWS_PATHS=1`  
**Note that both HTTP and HTTPS work** Please accept the self-signed certificate for testing when using HTTPS.

#### Locations, Ports:
* Make sure your /etc/hosts points isle.localdomain to 127.0.0.1. See original docs on how-to.
* Islandora is available at http://isle.localdomain
  * **You may need to point directly to the IP address of isle-apache, here's how:**
    - `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' isle-apache-ld`
    - Copy the IP and browse to it.  `http://{IP}/`
* Traefik is available at http://admin.isle.localdomain OR http://localhost:8080/
* Portainer is available at http://portainer.isle.localdomain OR http://localhost:9010/
* Fedora is available at http://isle.localdomain/fedora OR http://localhost:8081/
* Solr is available at http://isle.localdomain/solr OR http://localhost:8082/
* Image Services are available at http://images.isle.localdomain OR http://localhost:8083/
* Blazegraph is available at https://isle.localdomain:8084

#### Users and Passwords
Read as username:password

Islandora (Drupal) user and pass (default):
 * `isle`:`isle`

All Tomcat services come with the default users and passwords: (_includes Blazegraph Tomcat_)
* `admin`:`isle_admin`
* `manager`:`isle_manager`

Portainer's authentication can be configured: 
* By default there is no username or password required to login to Portainer.
* [Portainer Configuration](https://portainer.readthedocs.io/en/stable/configuration.html)        