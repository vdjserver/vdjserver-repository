VDJServer-Repository
====================

VDJServer is a next generation immune repertoire analysis portal and
platform. VDJServer-Repository is designed to be an AIRR-compliant
data repository.

##Deployments

 * Development: https://vdjserver.org -> vdj-rep-01.tacc.utexas.edu
 * Production: TBD

The current deployment configuration has the API and database on a single node.

##Components

VDJServer-Repository is currently composed of 2 separate components:

 * [vdjserver-mongodb](https://bitbucket.org/vdjserver/vdjserver-mongodb.git): The Mongo database.
 * [api-js-mongodb](https://bitbucket.org/vdjserver/api-js-mongodb.git): VDJServer implementation of the AIRR Common Repository API service with JavaScript implementation for MongoDB repository.

##Configuration Procedure

All configuration procedures are the same for dockerized and non-dockerized versions of these apps.

**Configuring vdjserver-mongodb**

The default docker-compose setup starts mongo with authentication on,
and no users exists in the default image. To setup the database, need
to decide:

* Where mongo will store its files on host disk. (e.g. /disk/mongodb)

* Name of database in mongo where collections will be stored (e.g. airr)

* Name and password for mongo service account. This account will have
  admin privileges for managing mongo.

* Name and password for guest account. This account will only have
  read access on the database for performing queries.

Make sure not to accidently commit the dbsetup file with usernames and
passwords into the git repository.

```
# Modify dbsetup.js with appropriate settings
cd vdjserver-mongodb
cp dbsetup.defaults dbsetup.js
emacs dbsetup.js

# Start up temporary mongo service, note mapping of mongo data directory and dbsetup
docker run -v /vdjZ/mongodb:/data/db -v $PWD:/dbsetup --name vdjr-mongo vdjserver/vdjserver-mongo

# Run setup script
docker exec -it vdjr-mongo mongo admin /dbsetup/dbsetup.js

# Stop mongo and get rid of name
docker stop vdjr-mongo
docker rm vdjr-mongo

# Edit docker-compose.yml and put in mapping of mongo data directory
```

**Configuring api-js-mongodb**

There is one configuration file that needs to be set up to run the
API. It can be copied from its default template.

```
cd api-js-mongodb
cp .env.defaults .env
emacs .env
```

**Configuring systemd**

You will need to set up the VDJServer-Repository systemd service file
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
VDJServer-Repository. The restart command will attempt to stop all
running docker-compose instances, and it is generally
successful. However, if it encounters any problems then you can just
stop instances manually and try it again:

```
[myuser@vdj vdjserver-web]$ sudo docker ps
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
2cd9a811064d        ireceptor/repository-mongo     "docker-entrypoint..."   4 weeks ago         Up 4 weeks          27017/tcp                vdjr-mongo
7e4163c4418c        ireceptor/service-js-mongodb   "node --harmony /s..."   4 weeks ago         Up 4 weeks          0.0.0.0:8080->8080/tcp   vdjr-ireceptor-api

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

##Accessing the Repository

For general users, the API is likely accessed with a GUI that hides
the technical details of communicating with the REST API. However, it
is useful to contact manually the API using the `curl` command to
verify that the service is operational.

** iReceptor API **

The top level entrypoint for the API will return a simple success status heartbeat.

```
$ curl 'https://vdj-staging.tacc.utexas.edu/ireceptor/v2'
{"result":"success"}
```

The info entrypoint will return version and other info about the service.

```
$ curl 'https://vdj-staging.tacc.utexas.edu/ireceptor/v2/info'
{"name":"vdjserver-ireceptor-node","description":"VDJServer API for iReceptor","version":"0.1.0"}
```

Neither of those entrypoints access the database, and this command will query both the study metadata and sequences data, returning a JSON data for both.

```
curl -X POST --data-urlencode "ir_project_sample_id=8485700680582295065-242ac11c-0001-012" 'https://vdj-staging.tacc.utexas.edu/ireceptor/v2/sequences_summary'
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
