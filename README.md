Limesurvey Docker container
==========

* This repo is a fork of https://github.com/adamzammit/limesurvey-docker. 
* The base image is at docker hub: https://hub.docker.com/r/acspri/limesurvey/

*Note: at the moment this repo is only used for documenting my workflow and providing a docker-compose file. There are no changes to the original [acspri/limesurvey image](https://hub.docker.com/r/acspri/limesurvey/), which is the one that is used in the workflow.* 

**<a name="contents">Contents:**
* **[Usage DEMO](#usage-demo)**:
* **[Docker tech docs](#docker-tech-doc)**:
* **[Useful Docker commands](#useful-docker-commands)**:
* [Original documentation](#orginal-docs): the orginal documentation, kept for reference.



**This container is for my Limesurvey DEMO purposes**:

It will fire up a local working instance of Limesurvey, in which you can create surveys, install themes, plugins, etc. The db content, installed theme and plugins will be locally persisted between container runs (as *docker volumes*)

It is not suitable for development or testing as this setup does not share code folders with the host.


## <a name="usage-demo">Usage DEMO
The original `acspri/limesurvey` README.MD [instructions](https://github.com/adamzammit/limesurvey-docker) are retained further below, they detail further customization possibilities. This is my own simplified - guaranteed-to-work - workflow:   


### Install (once)
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
Check out https://hub.docker.com/r/acspri/limesurvey/ for the available tags (= ls versions). With no tag specified, e.g. `image: acspri/limesurvey`, the latest stable version of LS that the image contains is pulled.

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

# Use dev version (work in progress)

* spin up
* copy files to host dir
* share volume with container

docker cp <containerId>:/file/path/within/container /host/path/target

e.g. docker cp 8bbaed94363f:/var/www/html C:\LSdocker2\html


# Deprecated clean up:



## Fire it up WITH directory mapping / code sharing 

**Create directories for code sharing** if you haven't already: create two folders on your host machine, one for *plugins* and another for *uploads*, e.g. `C:\LSdocker\plugins` and `C:\LSdocker\upload`. 

Then **start container** with directory mapping, referencing the newly created folders, e.g.:

**Does not work right now!**!!!
Read more here: https://www.ionos.com/community/server-cloud-infrastructure/docker/understanding-and-managing-docker-container-volumes/


**What I need is a folder on the host file system that contains the content of the 
html folder in the container, sp that I can edit the files on the host file system.**

see this answer
https://stackoverflow.com/questions/39176561/copying-files-to-a-container-with-docker-compose/39181484#39181484

...specifically this comment:

@Tarator yes indeed, the right hand side is not copied to the host anymore. I'll update the answer. As for a way to copy on container start, you can override the startup command with something like this docker run -v /dir/on/host:/hostdir php sh -c "cp -rp /var/www/html/* /hostdir && exec myapp". Don't forget to use exec to invoke the final command so that it is assigned PID1. That will make sure that myapp receives termination signals (Ctrl-C for instance). – Bernard Apr 24 '17 at 10:07



```bash
cd limesurvey-docker
docker-compose up -v C:\LSdocker\plugins:/var/www/html/plugins -v C:\LSdocker\upload:/var/www/html/upload
```

Visit http://localhost:8082 or http://localhost:8082/admin and log in with *admin* and *password* (specified in the `docker-composer.yml` file).

You can now edit the code of plugins and themes directly in the shared folders on your host machine. To **circumvent LS caching** you also need to  

**Troubleshoot:**

I you have previously started the container with NO folder mapping OR you want to map to other folders, you need to first run the command below, otherwise the folder mapping will have no effect:

remove container
```bash
docker-compose rm
```

Remove images and containers: https://devopsheaven.com/docker/volumes/purge/devops/2018/05/25/purge-docker-images-containers-networks-volumes.html

------

NEW

docker exec -it limesurvey-docker_limesurvey_1 cp -R /var/www/html cp -R /var/www/html /var/www/html2
docker exec -it limesurvey-docker_limesurvey_1 sed -ri -e 's!/var/www/html!/var/www/html2!g' /etc/apache2/sites-available/*.conf 

docker exec -it limesurvey-docker_limesurvey_1 grep --include={*.conf} -rnl "/etc/apache2/" -e "/var/www/html" | xargs -i@ sed -i s!/var/www/html!/var/www/dev!g @

docker exec -it limesurvey-docker_limesurvey_1 cd /etc/apache2/ && grep -rli "/var/www/html" *.conf | xargs -i@ sed -i s!/var/www/html!/var/www/dev!g @

docker exec -it -w /etc/apache2 limesurvey-docker_limesurvey_1 grep -rli "/var/www/html" *.conf | xargs -i@ sed -i s!/var/www/html!/var/www/dev!g @

find ./ -type f -exec sed -i s!/var/www/html!/var/www/dev!g {} \;

docker exec -it find /etc/apache2/ -name "*.conf" -exec sed -i s!/var/www/html!/var/www/dev!g {} \;

docker exec -it find /etc/apache2/ -name "*.conf" -exec sed -i s!/var/www/html!/var/www/dev!g {} \;

-

docker exec -it limesurvey-docker_limesurvey_1 /bin/bash find /etc/apache2/ -name "*.conf" -exec sed -i s!/var/www/html!/var/www/dev!g {} \;

docker exec -it limesurvey-docker_limesurvey_1 sh -c 'find /etc/apache2/ -name "*.conf" -exec sed -i s!/var/www/html!/var/www/dev!g {} \; '

more /etc/apache2/sites-available/000-default.conf

docker exec -it limesurvey-docker_limesurvey_1 sh -c "find /etc/apache2/ -name '*.conf' -exec sed -i s!/var/www/html!/var/www/dev!g {} \; "


docker exec -it limesurvey-docker_limesurvey_1 /etc/init.d/apache2 reload

docker cp limesurvey-docker_limesurvey_1:/var/www/html/ C:\LSdocker2\html
 
 -

1. create shared volume 
1. change apache conf tp point to new folder
1. restart apache
1. manually copy files 


    command:
      - sed -ri -e 's!/var/www/html!/var/www/html2!g' /etc/apache2/sites-available/*.conf 
      - sed -ri -e 's!/var/www/!/var/www/html2!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
    volumes:
      - C:\LSdocker2\html:/var/www/html2


command: bash -c "find /etc/apache2/ -name '*.conf' -exec sed -i s!/var/www/html!/var/www/dev!g {} \;" 
Edit `docker-composer.yml` file if yo need to alter other settings. Also check out the documentation below, e.g. for linking to external msyql instance, outside the container. 




* **todo** Find a way to have phpmaydamin access to the db?



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

