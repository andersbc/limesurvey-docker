Limesurvey Docker container
==========

* This repo is a fork of https://github.com/adamzammit/limesurvey-docker. 
* The base image is at docker hub: https://hub.docker.com/r/acspri/limesurvey/

*Note: at the moment this repo is only used for documenting my workflows and providing a docker-compose file for a setup that works for me. There are no changes to the original [acspri/limesurvey image](https://hub.docker.com/r/acspri/limesurvey/), which is the one that is used in the workflows.* 

**<a name="contents">Contents:**
* **[Usage DEMO](#usage-demo)**: Workflow for setting up a LS installation for **demo** purposes.
* **[Usage DEV](#usage-dev)**: Workflow for setting up a LS installation for **development** purposes. The code folder will be shared with the host system. 
* **[Docker tech docs](#docker-tech-doc)**: Notes on what this repo does.
* **[Useful Docker commands](#useful-docker-commands)**: commands for listing, deleting, etc. containers/volumes.
* [Original documentation](#orginal-docs): The original README.md documentation, kept for reference.



**This container is for my Limesurvey DEMO purposes**:


## <a name="usage-demo">Usage DEMO

*Use this workflow for demonstrating LS. It will fire up a local working instance of Limesurvey, in which you can create surveys, install themes, plugins, etc. The db content, installed theme and plugins will be locally persisted between container runs, as *docker volumes*. No code sharing in this workflow, so you can't access the code (see [Usage DEV](#usage-dev) for this).*


### <a name="usage-demo-install">Install (once)
```bash
# Clone this repo
git clone https://github.com/andersbc/limesurvey-docker.git
```

*...then specify Limesurvey version* in `docker-compose.yml`:
```yml
# Specify the LS version with a tag. In this example 4.1.7:
services:
  limesurvey:
    image: acspri/limesurvey:4.1.7

```
Check out https://hub.docker.com/r/acspri/limesurvey/ for the available tags (= ls versions). With no tag specified, e.g. `image: acspri/limesurvey`, the latest stable version of LS is pulled.

Save `docker-compose.yml` and you are ready to fire it up.

### Fire up
```bash
cd limesurvey-docker
# Using the docker-compose.yml: 
docker-compose up
# Wait for the operations to end.
# See troubleshoot if you get 'MySQL Connection Error'
```
Visit http://localhost:8082 or http://localhost:8082/admin and log in with *admin* and *password* (as specified in the `docker-composer.yml` file)

Surveys, answers, installed themes and plugins, etc. will be retained between runs.

**Troubleshot 'MySQL Connection Error'**: just ctrl+c and run `docker-compose up` again.

## <a name="usage-dev">Usage DEV

*Use this workflow for **developing plugins and themes**. The LS php code will be shared with the host system, so you can edit it in place. The db content will be locally persisted between container runs as a *docker volume*.* 

### Daily usage
Once you have completed **Install and Setup** below, simply run this command from the root of this library:
```bash
docker-compose -f docker-compose-dev.yml 
```
Visit http://localhost:8082 or http://localhost:8082/admin and log in with *admin* and *password* (as specified in the `docker-composer.yml` file)

...you can then live edit the code, in the shared folder specified during install.


### Install and setup (once)
```bash
# Clone this repo
git clone https://github.com/andersbc/limesurvey-docker.git
```

#### Edit `docker-compose-dev.yml`

Specify the LS version and the path to the folder on your host system that you want shared, as exemplified below. 

In this example we use LS version `4.1.7` and  **`C:\LSdocker2\html`** (yes, a windows machine):

```yml
# Specify the LS version with a tag. In this example 4.1.7:
services:
  limesurvey:
    image: acspri/limesurvey:4.1.7
    # ...
    volumes:
    - C:\LSdocker2\html:/var/www/dev
  # ... (leave other lines as is)

```
(...see Usage Demo for more about LS versions)


#### Setup code sharing
Start containers:
```bash
docker-compose -f docker-compose-dev.yml up
```

...then get the name of the *limesurvey* container with `docker ps`. In the commands below we will assume it is `limesurvey-docker_limesurvey_1` and that the shared host folder is **`C:\LSdocker2\html`**. Replace with whatever  you have.

```bash
# String replace '/var/www/html' -> '/var/www/dev' in apache conf files.
# '/var/www/dev' is what the shared host folder points to
docker exec -it limesurvey-docker_limesurvey_1 sh -c "find /etc/apache2/ -name '*.conf' -exec sed -i s!/var/www/html!/var/www/dev!g {} \; "

# Reload apache
docker exec -it limesurvey-docker_limesurvey_1 /etc/init.d/apache2 reload

# Copy contents of /var/www/html/ to host shared folder
# They will now also be visible in /var/www/dev
docker cp limesurvey-docker_limesurvey_1:/var/www/html/ C:\LSdocker2\html
```

The website should now be running from the files in the shared host folder. Visit http://localhost:8082 or http://localhost:8082/admin and log in with *admin*/*password*. Happy coding!

**Troubleshot 'MySQL Connection Error'**: just ctrl+c and run `docker-compose -f docker-compose-dev.yml up` again.

**FIX slow docker on windows**: Exclude the shared folder from Windows Defender. See highest rated comment here: https://github.com/docker/for-win/issues/1936


## <a name="docker-tech-doc">Docker tech documentation - what does it do?

* The docker-compose creates three linked containers: limesurvey, mysql and mail. 
* The [acspri/limesurvey](https://hub.docker.com/r/acspri/limesurvey/) docker image and corresponding github Dockerfile only pertains to the *Limesurvey* image. The other two images are maintained elsewhere.
* The acspri/limesurvey container also creates two *volumes* corresponding to Limesurvey *plugins* and *upload* directories, where plugins and themes are persisted. This happens in `Dockerfile`.
* The mysql container also creates a persisted docker volume for the db (it must).

## <a name="useful-docker-commands">Useful docker commands

* **list all containers and container id's**: `docker ps -a`
* **list all volumes and their id's**: `docker volumes ls`
* **See what volumes a specific container uses**: `docker inspect [containerId]`, then localize `Mounts` in the output. Or use terse form: `docker inspect -f "{{ .Mounts }}" [containerId]`
* **Delete the volumes**: WARNING - all data is lost: `docker-compose down -v`. Then do `docker-compose up` to recreate for scratch. You may have to run it twice to get rid of `MySQL Connection Error: ()`.
* **Remove all containers and/or volumes on system**: `docker system prune` respectively `docker volume prune`. 
* **Remove specific volumes**: List the volumes `docker volumes ls`, then delete by id: `docker volume rm [id1] [id2] [id3]`. To get the ids volumes the container uses see **See what volumes a specific container uses**. 
* **Remove specific containers**: `docker container rm [id1] [id2] [id3]`.


Also see https://www.ionos.com/community/server-cloud-infrastructure/docker/understanding-and-managing-docker-container-volumes/

and: https://github.com/docker/compose/issues/4476 


# <a name="orginal-docs">Original docs

**These are the unmodified original instructions from https://github.com/adamzammit/limesurvey-docker kept reference.**


This docker image is for Limesurvey on apache/php in its own container. It accepts environment variables to update the configuration file. On first run it will automatically create the database if a username and password are supplied, and on subsequent runs it can update the administrator password if provided as an environment variable.

Volumes are specified for plugins and upload directories for persistence.


# Tags

-    latest - Tracks LimeSurvey latest stable release (https://www.limesurvey.org/stable-release)
-    lts - Tracks LimeSurvey LTS release (https://www.limesurvey.org/lts-releases-download)
-    development - Tracks LimeSurvey development release (https://www.limesurvey.org/development-release)


# How to use this image 

```console
$ docker run --name some-limesurvey --link some-mysql:mysql -d acspri/limesurvey
```

The following environment variables are also honored for configuring your Limesurvey instance. If Limesurvey is already installed, these environment variables will update the Limesurvey config file.

-	`-e LIMESURVEY_DB_HOST=...` (defaults to the IP and port of the linked `mysql` container)
-	`-e LIMESURVEY_DB_USER=...` (defaults to "root")
-	`-e LIMESURVEY_DB_PASSWORD=...` (defaults to the value of the `MYSQL_ROOT_PASSWORD` environment variable from the linked `mysql` container)
-	`-e LIMESURVEY_DB_NAME=...` (defaults to "limesurvey")
-	`-e LIMESURVEY_TABLE_PREFIX=...` (defaults to "" - set this to "lime_" for example if your database has a prefix)
-	`-e LIMESURVEY_ADMIN_USER=...` (defaults to "" - the username of the Limesurvey administrator)
-	`-e LIMESURVEY_ADMIN_PASSWORD=...` (defaults to "" - the password of the Limesurvey administrator)
-	`-e LIMESURVEY_ADMIN_NAME=...` (defaults to "Lime Administrator" - The full name of the Limesurvey administrator)
-	`-e LIMESURVEY_ADMIN_EMAIL=...` (defaults to "lime@lime.lime" - The email address of the Limesurvey administrator)
-	`-e LIMESURVEY_DEBUG=...` (defaults to 0 - Debug level of Limesurvey, 0 is off, 1 for errors, 2 for strict PHP and to be able to edit standard templates)
-	`-e LIMESURVEY_SQL_DEBUG=...` (defaults to 0 - Debug level of Limesurvey for SQL, 0 is off, 1 is on - note requires LIMESURVEY_DEBUG set to 2)
-	`-e LIMESURVEY_USE_INNODB=...` (defaults to '' - Leave blank or don't set to use standard MyISAM database. Set to any value to use InnoDB (required for some cloud providers))
-	`-e MYSQL_SSL_CA=...` (path to an SSL CA for MySQL based in the root directory (/var/www/html). If changing paths, escape your forward slashes. Do not set or leave blank for a non SSL connection)

If the `LIMESURVEY_DB_NAME` specified does not already exist on the given MySQL server, it will be created automatically upon startup of the `limesurvey` container, provided that the `LIMESURVEY_DB_USER` specified has the necessary permissions to create it.

If you'd like to be able to access the instance from the host without the container's IP, standard port mappings can be used:

```console
$ docker run --name some-limesurvey --link some-mysql:mysql -p 8080:80 -d acspri/limesurvey
```

Then, access it via `http://localhost:8080` or `http://host-ip:8080` in a browser.

If you'd like to use an external database instead of a linked `mysql` container, specify the hostname and port with `LIMESURVEY_DB_HOST` along with the password in `LIMESURVEY_DB_PASSWORD` and the username in `LIMESURVEY_DB_USER` (if it is something other than `root`):

```console
$ docker run --name some-limesurvey -e LIMESURVEY_DB_HOST=10.1.2.3:3306 \
    -e LIMESURVEY_DB_USER=... -e LIMESURVEY_DB_PASSWORD=... -d acspri/limesurvey
```

## ... via [`docker-compose`](https://github.com/docker/compose)

Example `docker-compose.yml` for `limesurvey`:

```yaml
version: '2'

services:

  limesurvey:
    image: acspri/limesurvey
    ports:
      - 8082:80
    environment:
      LIMESURVEY_DB_PASSWORD: example
      LIMESURVEY_ADMIN_USER: admin
      LIMESURVEY_ADMIN_PASSWORD: password
      LIMESURVEY_ADMIN_NAME: Lime Administrator
      LIMESURVEY_ADMIN_EMAIL: lime@lime.lime

  mysql:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: example
```

Run `docker-compose up`, wait for it to initialize completely, and visit `http://localhost:8082` or `http://host-ip:8082`.

# Supported Docker versions

This image is officially supported on Docker version 1.12.3.

Support for older versions (down to 1.6) is provided on a best-effort basis.

Please see [the Docker installation documentation](https://docs.docker.com/installation/) for details on how to upgrade your Docker daemon.

Notes
-----

This Dockerfile is based on the [Wordpress official docker image](https://github.com/docker-library/wordpress/tree/8ab70dd61a996d58c0addf4867a768efe649bf65/php5.6/apache)

