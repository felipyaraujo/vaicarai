
# OTRS 5 Ticketing System
[![Docker Stars](https://img.shields.io/docker/stars/juanluisbaptiste/otrs.svg?style=flat-square)](https://hub.docker.com/r/juanluisbaptiste/otrs/)
[![Docker Pulls](https://img.shields.io/docker/pulls/juanluisbaptiste/otrs.svg?style=flat-square)](https://hub.docker.com/r/juanluisbaptiste/otrs/)

Unofficial [OTRS 5 Free](http://www.otrs.com/software/) docker image. This repository contains the
*Dockerfiles* and all other files needed to build and run the container.

We also include a *MariaDB Dockerfile* for a pre-configured image with the [required database settings](http://otrs.github.io/doc/manual/admin/stable/en/html/installation.html).

The OTRS image doesn't include a SMTP service, decoupling applications into multiple containers makes it 
much easier to scale horizontally and reuse containers. If you don't have access to a SMTP server, you 
can instead link against this [SMTP relay](https://github.com/juanluisbaptiste/docker-postfix) postfix container.

These images are based on the [official CentOS images](https://registry.hub.docker.com/_/centos/) and 
include the latest OTRS version. Older images will be tagged with the OTRS version they run. 

*Mod_perl* is enabled by default.

OTRS 4 image sources are still available in otrs-4_0_x branch.

### Build instructions

We use `docker-compose` to build the images. Clone this repo and then:

    cd docker-otrs
    sudo docker-compose build

This command will build all the images and pull the missing ones like the [SMTP relay](https://github.com/juanluisbaptiste/docker-postfix).

You can also find a prebuilt image from [Docker Hub](https://registry.hub.docker.com/u/juanluisbaptiste/otrs/), 
which can be pulled with this command:

    sudo docker pull juanluisbaptiste/otrs:latest

### How to run it

By default, when the container is run it will load a default vanilla OTRS 
installation (`OTRS_INSTALL=no`) that is ready to be configured as you need. However, you can load 
a backup or run the installer by defining one of these env variables:

* `OTRS_INSTALL=restore` Will restore the backup specified by `OTRS_BACKUP_DATE` 
environment variable. 
* `OTRS_BACKUP_DATE` is the backup name to restore, in the same *date_time* format that the OTRS backup
script uses, for example `OTRS_BACKUP_DATE="2015-05-26_00-32"` (This is the notation that the backup script that comes with OTRS uses). Backups must be inside the */var/otrs/backups* directory (you should host mount it).
* `OTRS_DROP_DATABASE=yes` Will drop the otrs database it if already exists (by default the container will fail if the database already exists).

You need to mount that backups volume from somewhere, it can be from another volume (using *--volumes-from*) or mounting 
a host volume which contains the backup files.

* `OTRS_INSTALL=yes` Will run the installer which you can access at:

    http://localhost/otrs/install.pl

If you are running the container remotely, replace *localhost* with the server's hostname. 

There are also some other environment variables that can be set to customize
the default install:

* `OTRS_HOSTNAME` Sets the container's hostname (autogenerated if not defined).
* `OTRS_SYSTEM_ID` Sets the system's ticketing ID.
* `OTRS_ADMIN_EMAIL` Sets the admin user email.
* `OTRS_ORGANIZATION` Sets the organization name (ex: MyCompany Ltd.)
* `OTRS_DB_PASSWORD` otrs user database password. If it's not set the password will be randomly generated (recommended).
* `OTRS_ROOT_PASSWORD` root@localhost user password. Default password is `changeme`.
* `OTRS_POSTMASTER_FETCH_TIME` Sets the time interval (at otrs user's crontab) OTRS should fetch emails from the configured postmaster accounts. This value it's 10 minutes by default, but I like to set it to 5 minutes. Email fetching can be disabled altogether by setting this variable to 0 (useful while configuring).
* `OTRS_LANGUAGE` Set the default language for both agent and customer interfaces (For example, "es" for spanish).
* `OTRS_TICKET_COUNTER` Sets the starting point for the ticket counter.
* `OTRS_NUMBER_GENERATOR` Sets the ticket number generator, possible values are : *DateChecksum*, *Date*, *AutoIncrement* or *Random*.


Those environment variables is what you can configure by running the installer for a default install.

After adjusting the [`docker-compose.yml`](https://github.com/juanluisbaptiste/docker-otrs/blob/master/docker-compose-prod.yml), you can test the containers with `docker-compose`:

    sudo docker-compose build
    sudo docker-compose up

This will bring up all needed containers, link them and mount volumes according 
to the `docker-compose.yml` configuration file. 

The default database password is `changeme`, to change it, edit the `docker-compose.yml` file and change the 
`MYSQL_ROOT_PASSWORD` environment variable on the `mariadb` image definition before 
running `docker-compose`.

To start the containers in production mode use this [`docker-compose.yml`](https://github.com/juanluisbaptiste/docker-otrs/blob/master/docker-compose-prod.yml) file that points to images to be pulled and run instead of `Dockerfiles` being built:

    sudo docker-compose -f docker-compose-prod.yml -p companyotrs up -d

After the containers finish starting up you can access the OTRS system at the following
addresses:

### Administration Interface
    http://$OTRS_HOSTNAME/otrs/index.pl
    
### Customer Interface
    http://$OTRS_HOSTNAME/otrs/customer.pl

### Changing OTRS default skins

The default skins and logos for the agent and customer interfaces can be controlled with the following
environment variables:

To set the agent interface skin set `OTRS_AGENT_SKIN` environment variable, for example:

    OTRS_AGENT_SKIN: "ivory"

To set the agent Interface logo set `OTRS_AGENT_LOGO`:

    OTRS_AGENT_LOGO: skins/Agent/ivory/img/your_logo.png

You can also control the logo's size and placement (set in px units):

    OTRS_AGENT_LOGO_HEIGHT: 50
    OTRS_AGENT_LOGO_RIGHT: 40
    OTRS_AGENT_LOGO_TOP: 5
    OTRS_AGENT_LOGO_WIDTH: 240

To set the customer interface skin set `OTRS_CUSTOMER_SKIN` environment variable, for example:

    OTRS_CUSTOMER_SKIN: "ivory"

To set the customer Interface logo set `OTRS_CUSTOMER_LOGO`:

    OTRS_CUSTOMER_LOGO: skins/Customer/ivory/img/your_logo.png

You can also control the logo's size and placement (set in px units):

    OTRS_CUSTOMER_LOGO_HEIGHT: 50
    OTRS_CUSTOMER_LOGO_RIGHT: 40
    OTRS_CUSTOMER_LOGO_TOP: 5
    OTRS_CUSTOMER_LOGO_WIDTH: 240


If you are adding your own skins, the easiest way is create your own `Dockerfile` inherited from this image and then `COPY` the skin files there. You can also set all the environment variables in there too, for example:

    FROM juanluisbaptiste/otrs:latest
    MAINTAINER Foo Bar <foo@bar.com>
    ENV OTRS_AGENT_SKIN mycompany
    ENV OTRS_AGENT_LOGO skins/Agent/mycompany/img/logo.png
    ENV OTRS_CUSTOMER_LOGO skins/Customer/default/img/logo_customer.png

    COPY skins/ $SKINS_PATH/
    RUN mkdir -p $OTRS_ROOT/Kernel/Config/Files/
    COPY skins/Agent/MyCompanySkin.xml $OTRS_ROOT/Kernel/Config/Files/

### Using host-mounted data containers

If you want to store OTRS MySQL and configuration files outside the containers then you need to modify the `docker-compose.yml` file to map those directories to a directory in the host (available on both OTRS 4 & 5 images). First, modify the data containers to map the volume directories to the host:

    data:
      image: centos/mariadb:latest
      volumes:
      - ./volumes/mysql:/var/lib/mysql
      - /etc/localtime:/etc/localtime:ro
      command: /bin/true
    data-otrs:
      image: juanluisbaptiste/otrs:latest
      volumes:
      - ./volumes/config:/opt/otrs/Kernel
      - ./backups:/var/otrs/backups
      - /etc/localtime:/etc/localtime:ro
      command: /bin/true

#### Note ####
Make sure that the directories on the docker host for both OTRS configuration and the MySQL data containers has the correct permissions to be accessed from within the containers. The `volumes/mysql` directory should be owned by the MySQL user (27) and the `volumes/config` directory must be owned by id 500 and group id 48. Before running `docker-compose up` make sure permissions are ok:

    chown 27 volumes/mysql
    chown 500:48 volumes/config

### Backing up the container configuration

Run `/opt/otrs/scripts/otrs_backup.sh` script to create a full backup that will be copied to */var/otrs/backups*. If you mounted that directory as a host volume then you will have access to the backups files from the docker host server. You can setup a periodic cron job on the host that runs the following command:

    docker exec otrs /opt/otrs/scripts/otrs_backup.sh
