VDJServer-Repository
====================

VDJServer is a next generation immune repertoire analysis portal and
platform. VDJServer-Repository is designed to be an AIRR-compliant
data repository.

##Deployments

 * Development: TBD
 * Production: https://vdjserver.org -> bowie.corral.tacc.utexas.edu

The current deployment configuration has the API and database on a single node.

##Components

VDJServer-Repository is currently composed of 3 separate components:

 * [repository-mongodb](https://github.com/sfu-ireceptor/repository-mongodb): The Mongo database.
 * [service-js-mongodb](https://github.com/sfu-ireceptor/service-js-mongodb): iReceptor API service with JavaScript implementation for MongoDB repository.
 * [api-js-mongodb](https://github.com/airr-community/api-js-mongodb): AIRR Common Repository API service with JavaScript implementation for MongoDB repository.

##Configuration Procedure

All configuration procedures are the same for dockerized and non-dockerized versions of these apps.

**Configuring repository-mongodb**

**Configuring service-js-mongodb**

**Configuring systemd**

You will need to set up the VDJServer-Repository systemd service file
on your host machine in order to have the infrastructure automatically
restart when the host machine reboots.

```
sudo cp host/systemd/vdjserver-repository.service /etc/systemd/systems/vdjserver-repository.service

sudo systemctl daemon-reload

sudo systemctl enable docker

sudo systemctl enable vdjserver-repository
```

##Deployment Procedure

###SSL

VDJServer-Repository does not handle SSL certificates directly, and is
currently configured to run HTTP internally on port 8080. It must be
deployed behind a reverse proxy in order to allow SSL connections.

###Dockerized instances (vdj-dev, vdj-staging, production)

Dockerized instances may be started/stopped/restarted using the
supplied systemd script: host/systemd/vdjserver-repository.service.
It can be accessed as follows:

```
[myuser@vdj vdjserver-web]$ sudo systemctl <ACTION> vdjserver-repository
# <ACTION> can be either: stop, start, or restart
```

In most cases, a simple restart command is sufficient to bring up
Vdjserver-Repository. The restart command will attempt to stop all
running docker-compose instances, and it is generally
successful. However, if it encounters any problems then you can just
stop instances manually and try it again:

```
[myuser@vdj vdjserver-web]$ sudo docker ps
CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS              PORTS                                      NAMES
fdc7c3119366        vdjserverweb_nginx:latest    "/root/nginx-config-   32 minutes ago      Up 32 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   vdjserverweb_nginx_1
adfecbce3e55        vdjserverweb_vdjapi:latest   "/bin/sh -c '/usr/bi   32 minutes ago      Up 32 minutes       8443/tcp                                   vdjserverweb_vdjapi_1

[myuser@vdj vdjserver-web]$ sudo docker-compose down vdjserver-repository
```

It is also important to note that the systemd vdjserver-repository
command will not rebuild new container instances. If you need to
build/rebuild a new set of containers, then you will need to start the
command manually as follows:

```
[myuser@vdj vdjserver-web]$ sudo docker-compose build
```

After your build has completed, you can then use systemd to deploy it:

```
[myuser@vdj vdjserver-web]$ sudo systemctl restart vdjserver-repository
```

Systemd will only restart a running service if the "restart" command is used; remember that using the "start" command twice will not redeploy any containers.


**Docker Compose Files**

There are two docker-compose files: one for general use ("docker-compose.yml"), and one that has been adjusted for use in a production environment ("docker-compose.prod-override.yml"). These files are meant to be overlayed and used together in a production environment: https://docs.docker.com/compose/extends/#different-environments

Using the production config will send all log information to syslog.

Example of using the production overlayed config:

```
docker-compose -f docker-compose.yml -f docker-compose.prod-override.yml build

docker-compose -f docker-compose.yml -f docker-compose.prod-override.yml up
```


##Development Guidelines

**Code Style**

 * Code should roughly follow Google Javascript Style Guide conventions: <https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml>.

 * A jscs.rc file (Javascript Code Style Checker) file has been provided in the project repo, and all developers are encouraged to use it.

 * A git pre-commit hook is available via the file pre-commit.sh. To use it, just symlink it as follows: ```ln -s ../../pre-commit.sh .git/hooks/pre-commit```

 * Spaces are preferred over tabs, and indentation is set at 4 spaces.

 *  Vimrc settings: ```set shiftwidth=4, softtabstop=4, expandtab```

**Git Structure and Versioning Process**

 * This project uses the Git Flow methodology for code management and development: <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>.

 * New development and features should be done on branches that are cloned from the **develop** branch, and then merged into this branch when completed. Likewise, new release candidates should be branched from **develop**, and then merged into **master** once they have been tested/verified. Once a release branch is ready for production, it should be merged into **master** and tagged appropriately. Every deployment onto production should be tagged following semantic versioning 2.0.0 standards: <http://semver.org/>.

##Development Setup

You will need to clone down the parent project and all submodules in order to set up a local instance of vdjserver.

```
- Clone project
$ git clone git@bitbucket.org:vdjserver/vdjserver-repository.git

cd vdjserver-repository

- Clone submodules
$ git submodule update --init
$ git submodule foreach git checkout master
$ git submodule foreach git pull

- Follow configuration steps listed above in the "Configuration Procedure" section of this document
```
