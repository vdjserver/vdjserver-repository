# VDJServer Repository

VDJServer is a next generation immune repertoire analysis portal and
platform. VDJServer Repository is an AIRR-compliant data repository with additional
administrative interfaces. It provides ADC services for the VDJServer Community Data Portal.

## Deployments

 * Production: https://vdjserver.org
 * Staging: https://vdj-staging.tacc.utexas.edu
 * Development: http://localhost

## Components

VDJServer Repository is currently composed of 3 separate components:

 * [adc-api-js-tapis](https://github.com/vdjserver/adc-api-js-tapis): VDJServer implementation of the AIRR Data Commons API with JavaScript implementation for Tapis v3 metadata.
 * [adc-async-api-js-tapis](https://github.com/vdjserver/adc-async-api-js-tapis): VDJServer implementation of the AIRR Data Commons ASYNC API with JavaScript implementation for Tapis v3 metadata.
 * [stats-api-js-tapis](https://github.com/vdjserver/stats-api-js-tapis): VDJServer implementation of the iReceptorPlus Stats API with JavaScript implementation for Tapis v3 metadata.

## Configuration Procedure

You will need to clone the parent project and all submodules in order to set up an instance of VDJServer Repository.

```
# Clone project
$ git clone https://github.com/vdjserver/vdjserver-repository.git

$ cd vdjserver-repository

# Clone submodules
$ git submodule update --init --recursive
```

All configuration procedures are based upon docker containers of the components.

There is one configuration file that needs to be set up to run the
repository. It can be copied from its default template.

```
cp .env.defaults .env
pico .env
```

## Deployment Procedure

### SSL

VDJServer does not handle SSL certificates directly, and is currently configured to run HTTP internally on port 8080. It must be deployed behind a reverse proxy in order to allow SSL connections.

Review the [VDJServer administration guide](https://vdjserver.readthedocs.io/en/latest/admin/admin.html) for details about configuring nginx with SSL.

### Docker compose build

There are a set of docker compose files for building and starting all of the components.

```
$ cd docker-compose/airr
$ docker compose build
```

If everything has build properly, then can manually bring up the system with:

```
$ docker compose up
```

Verify no unexpected error messages. Verify you can access the website GUI. Manually bring down the system with:

```
$ docker compose down
```

### Configuring systemd

You will need to set up the VDJServer systemd service file on your host machine in order to have the VDJ web infrastructure automatically restart when the host machine reboots.
Verify the paths point to the appropriate docker-compose directory.

```
sudo cp host/systemd/vdjserver-repository.airr.service /etc/systemd/system/vdjserver-repository.service

sudo systemctl daemon-reload

sudo systemctl enable docker

sudo systemctl enable vdjserver-repository

sudo systemctl start vdjserver-repository
```

### Running the test suites (staging, production)**

The test suites for each component can be run as during development, just change the
URL host to the appropriate server.

## Accessing the Repository

For general users, the API is likely accessed with a GUI that hides
the technical details of communicating with the web API. However, it
is useful to contact manually the API using the `curl` command to
verify that the service is operational.

### ADC API

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

### ADC API Asynchronous Extension

The top level entrypoint for the ADC ASYNC API will return a simple success status heartbeat.

```
$ curl 'http://localhost:8021/airr/async/v1'
{"result":"success"}
```

### iReceptorPlus Stats API

The top level entrypoint for the STATS API will return a simple success status heartbeat.

```
$ curl 'http://localhost:8025/irplus/v1/stats'
{"result":"success"}
```

The info entrypoint will return version and other info about the service.

```
$ curl 'http://localhost:8025/irplus/v1/stats/info'
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


## Development Guidelines

**Code Style**

 * Code should roughly follow Google Javascript Style Guide conventions: <https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml>.

 * A jscs.rc file (Javascript Code Style Checker) file has been provided in the project repo, and all developers are encouraged to use it.

 * A git pre-commit hook is available via the file pre-commit.sh. To use it, just symlink it as follows: ```ln -s ../../pre-commit.sh .git/hooks/pre-commit```

 * Spaces are preferred over tabs, and indentation is set at 4 spaces.

 *  Vimrc settings: ```set shiftwidth=4, softtabstop=4, expandtab```

**Git Structure and Versioning Process**

 * This project uses the Git Flow methodology for code management and development: <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>.

 * New development and features should be done on branches that are cloned from the **develop** branch, and then merged into this branch when completed. Likewise, new release candidates should be branched from **develop**, and then merged into **master** once they have been tested/verified. Once a release branch is ready for production, it should be merged into **master** and tagged appropriately. Every deployment onto production should be tagged following semantic versioning 2.0.0 standards: <http://semver.org/>.

**Development Setup**

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

**Dockerized instances (local)**

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
[myuser@vdj airr]$ docker compose down
```

**Running the test suites (local)**

Look in each component test directory for a test suite and readme.
