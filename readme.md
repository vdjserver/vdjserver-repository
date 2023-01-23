VDJServer Repository
====================

VDJServer is a next generation immune repertoire analysis portal and
platform. VDJServer Repository is designed to be an AIRR-compliant
data repository. It is the backend services for the VDJServer Community Data Portal.

Deployments
-----------

 * Production: https://vdjserver.org -> vdj-rep-01.tacc.utexas.edu
 * Staging: https://vdj-staging.tacc.utexas.edu -> vdj-rep-02.tacc.utexas.edu
 * Development: http://localhost

Components
-----------

VDJServer-Repository is currently composed of 2 separate components:

 * [api-js-tapis](https://bitbucket.org/vdjserver/api-js-tapis.git): VDJServer implementation of the AIRR Data Commons API with JavaScript implementation for Tapis v3 metadata.
 * [stats-api-js-tapis](https://bitbucket.org/vdjserver/stats-api-js-tapis.git): VDJServer implementation of the iReceptorPlus Stats API with JavaScript implementation for Tapis API.

Configuration Procedure
-----------

All configuration procedures are the same for dockerized and non-dockerized versions of these apps.

There is one configuration file that needs to be set up to run the
repository. It can be copied from its default template.

```
cp .env.defaults .env
pico .env
```

**Configuring systemd**

You will need to set up the VDJServer Repository systemd service file
on your host machine in order to have the infrastructure automatically
restart when the host machine reboots. Copy the appropriate template.

```
sudo cp host/systemd/vdjserver-repository.airr.service /etc/systemd/system/vdjserver-repository.service

sudo systemctl daemon-reload

sudo systemctl enable docker

sudo systemctl enable vdjserver-repository
```

##Deployment Procedure

###SSL

VDJServer Repository does not handle SSL certificates directly, and is
currently configured to run HTTP internally on port 8080. It must be
deployed behind a reverse proxy in order to allow SSL connections.

###Dockerized instances (vdj-dev, vdj-staging, production)

Dockerized instances may be started/stopped/restarted using the
supplied systemd script: host/systemd/vdjserver-repository.service.
It can be accessed as follows:

```
[myuser@vdj vdjserver-repository]$ sudo systemctl <ACTION> vdjserver-repository
# <ACTION> can be either: stop, start, or restart
```

In most cases, a simple restart command is sufficient to bring up
VDJServer Repository. The restart command will attempt to stop all
running docker-compose instances, and it is generally
successful. However, if it encounters any problems then you can just
stop instances manually and try it again:

```
[myuser@vdj vdjserver-repository]$ docker ps
[s166813@OSPH-516048 (vdjserver) vdjserver-repository-irplus]$ docker ps
CONTAINER ID   IMAGE                                       COMMAND                  CREATED          STATUS          PORTS                              NAMES
2052d0d14367   vdjserver/api-js-tapis                      "bash /api-js-tapis/…"   15 minutes ago   Up 15 minutes   0.0.0.0:8020-8021->8020-8021/tcp   vdjr-airr-tapis
741d013098a3   vdjserver/stats-api-js-tapis                "bash /stats-api-js-…"   15 minutes ago   Up 15 minutes   0.0.0.0:8025->8025/tcp             vdjr-stats-tapis

[myuser@vdj vdjserver-repository]$ sudo docker-compose down vdjserver-repository
```

It is also important to note that the systemd vdjserver-repository
command will not rebuild new container instances. If you need to
build/rebuild a new set of containers, then you will need to start the
command manually as follows:

```
[myuser@vdj vdjserver-repository]$ cd docker-compose/airr
[myuser@vdj airr]$ docker compose build
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

##Accessing the Repository

For general users, the API is likely accessed with a GUI that hides
the technical details of communicating with the REST API. However, it
is useful to contact manually the API using the `curl` command to
verify that the service is operational.

** ADC API **

The top level entrypoint for the ADC API will return a simple success status heartbeat.

```
$ curl 'http://localhost:8020/airr/v1'
{"result":"success"}
```

The info entrypoint will return version and other info about the service.

```
$ curl 'http://localhost:8020/airr/v1/info'
{
  "title": "api-js-tapis",
  "description": "AIRR Data Commons API for VDJServer Community Data Portal",
  "version": "2.0.0",
  "contact": {
    "name": "VDJServer",
    "url": "http://vdjserver.org/",
    "email": "vdjserver@utsouthwestern.edu"
  },
  "license": {
    "name": "GNU AGPL V3"
  },
  "api": {
    "title": "AIRR Data Commons API",
    "version": "1.2.0",
    "contact": {
      "name": "AIRR Community",
      "url": "http://www.airr-community.org/",
      "email": "join@airr-community.org"
    },
    "description": "Major Version 1 of the Adaptive Immune Receptor Repertoire (AIRR) data repository web service application programming interface (API).\n",
    "license": {
      "name": "Creative Commons Attribution 4.0 International",
      "url": "https://creativecommons.org/licenses/by/4.0/"
    }
  },
  "schema": {
    "title": "AIRR Schema",
    "description": "Schema definitions for AIRR standards objects",
    "version": "1.4",
    "contact": {
      "name": "AIRR Community",
      "url": "https://github.com/airr-community"
    },
    "license": {
      "name": "Creative Commons Attribution 4.0 International",
      "url": "https://creativecommons.org/licenses/by/4.0/"
    }
  },
  "max_size": 1000,
  "max_query_size": 2097152
}
```

** ADC API Asynchronous Extension **

The top level entrypoint for the ADC ASYNC API will return a simple success status heartbeat.

```
$ curl 'http://localhost:8021/airr/async/v1'
{"result":"success"}
```

** iReceptorPlus Stats API **

The top level entrypoint for the STATS API will return a simple success status heartbeat.

```
$ curl 'http://localhost:8025/irplus/v1/stats'
{"result":"success"}
```

The info entrypoint will return version and other info about the service.

```
$ curl 'http://localhost:8025/irplus/v1/stats'
{
  "title": "stats-api-js-tapis",
  "description": "VDJServer Statistics API for Tapis backend",
  "version": "0.1.0",
  "contact": {
    "name": "VDJServer",
    "email": "vdjserver@utsouthwestern.edu",
    "url": "https://vdjserver.org"
  },
  "api": {
    "title": "iReceptorPlus Statistics API",
    "version": "0.3.0",
    "description": "Statistics API for the iReceptor Plus platform.\n",
    "contact": {
      "name": "iReceptor Plus",
      "url": "https://www.ireceptor-plus.com",
      "email": "info@ireceptor-plus.com"
    }
  }
}
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

- Clone submodules (recursive as some submodules have their own submodules)
$ git submodule update --init --recursive
$ git submodule foreach git checkout master
$ git submodule foreach git pull

- Follow configuration steps listed above in the "Configuration Procedure" section of this document
```

```
[myuser@vdj vdjserver-repository]$ docker-compose build
```

###Dockerized instances (local)

When deploying locally during development, similar commands are performed except manually instead of by systemctl.

To build/rebuild a new set of containers, run docker compose:

```
[myuser@vdj vdjserver-repository]$ cd docker-compose/airr
[myuser@vdj airr]$ docker compose build
```

To bring up the service:

```
[myuser@vdj vdjserver-repository]$ cd docker-compose/airr
[myuser@vdj airr]$ docker compose up
```

To bring down the service:

```
[myuser@vdj vdjserver-repository]$ cd docker-compose/airr
[myuser@vdj airr]$ docker compose up
```

