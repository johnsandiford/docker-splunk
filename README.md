# Welcome to the Splunk Docker GitHub repository

This is the official repository for the Splunk Enterprise and Splunk universal forwarder Docker effort. It contains Dockerfiles that you can use to build [Splunk](https://splunk.com) Docker images.

To learn more about the Splunk Enterprise Docker image, see the [Splunk Enterprise Docker image README](https://github.com/splunk/docker-splunk/blob/master/enterprise/README.md).

To learn more about the Splunk universal forwarder image, see the [Splunk universal forwarder Docker image README](https://github.com/splunk/docker-splunk/blob/master/universalforwarder/README.md).

<<<<<<< HEAD
* `6.5`, `6.5.0`, `latest` - Splunk Enterprise
* `6.5-light`, `6.5.0-light`, `latest-light` - Splunk Light
* `6.5-forwarder`, `6.5.0-forwarder`, `latest-forwarder` - Splunk Universal Forwarder
=======
# What is Splunk Enterprise?
>>>>>>> splunk/master

Splunk Enterprise is the platform for operational intelligence. The software lets you collect, analyze, and act upon the untapped value of big data that your technology infrastructure, security systems, and business applications generate. It gives you insights to drive operational performance and business results.

# Get help and support

<<<<<<< HEAD
> NOTE: I'm working at Splunk, but this is not an official Splunk images.
> I build them in my free time when I'm not at work. I have some knowledge
> about Splunk, but you should think twice before putting them in
> production. I run these images on my own home server just for
> my personal needs. If you have any issues - feel free to open a
> [bug](https://github.com/johnsandiford/docker-splunk/issues).
=======
More information about the Docker images and how to pull and run them is available in the README for each image.
>>>>>>> splunk/master

If you have questions or need support, you can:

* Post a question to [Splunk Answers](http://answers.splunk.com)
* Join the [Splunk Slack channel](http://splunk-usergroups.slack.com)
* Visit the #splunk channel on [EFNet Internet Relay Chat](http://www.efnet.org)
* Send an email to [docker-maint@splunk.com](mailto:docker-maint@splunk.com)

<<<<<<< HEAD
### Version

* Version: `6.5.0`
* Build: `59c8927def0f`

## Installation

Build the image locally.

```bash
git clone https://github.com/johnsandiford/docker-splunk.git
cd docker-splunk/splunk
docker build --tag="$USER/splunk" .
```

## Quick Start

To manually start Splunk Enterprise container 

```bash
docker run --hostname splunk -p 8000:8000 -d --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" johnsandiford/splunk:6.5.0
```

This docker image has two data volumes `/opt/splunk/etc` and `/opt/splunk/var` (See [Data Store](#data-store)). To avoid losing any data when container is stopped/deleted mount these volumes from docker volume containers (see [Managing data in containers](https://docs.docker.com/userguide/dockervolumes/))

```bash
docker run --name vsplunk -v /opt/splunk/etc -v /opt/splunk/var busybox
docker run --hostname splunk --name splunk --volumes-from=vsplunk -p 8000:8000 -d --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" johnsandiford/splunk:6.5.0
```

Or if you use [docker-compose](https://docs.docker.com/compose/)

```
version: '2'
services:

  splunk:
    image: johnsandiford/splunk:6.5.0
    environment: 
      - SPLUNK_START_ARGS=--accept-license --answer-yes --no-prompt
    hostname: splunk
    volumes:
      - config:/opt/splunk/etc
      - data:/opt/splunk/var
    ports:
      - 8000:8000
volumes:
  config:
    driver: local
  data:
    driver: local
```

## Configuration

### Data Store

This image has two data volumes

* `/opt/splunk/etc` - stores Splunk configurations, including applications and lookups
* `/opt/splunk/var` - stores indexed data, logs and internal Splunk data

### User

Splunk processes are running under `splunk` user.

### Ports

Next ports are exposed

* `8000/tcp` - Splunk Web interface (Splunk Enterprise and Splunk Light)
* `8089/tcp` - Splunk Services (All Splunk products)
* `8191/tcp` - Application KV Store (Splunk Enterprise)
* `9997/tcp` - Splunk Indexing Port (not used by default) (Splunk Enterprise)
* `1514` - Network Input (not used by default) (All Splunk products)
* `8088` - HTTP Event Collector

> We are using `1514` instead of standard `514` syslog port because ports below 
> 1024 are reserved for root access only. See [Run Splunk Enterprise as a different or non-root user](http://docs.splunk.com/Documentation/Splunk/latest/Installation/RunSplunkasadifferentornon-rootuser).

### Entrypoint

You can execute Splunk commands by using

```
docker exec splunk entrypoint.sh splunk version
```

*Splunk is launched in background. Which means that when Splunk restarts (after some configuration changes) - the container will not be affected.*

### Hostname

It is recommended to specify `hostname` for this image, so if you will recreate Splunk instance you will keep the same hostname.

### Basic configuration using Environment Variables

> Some basic configurations are allowed to configure Indexers/Forwarders using
environment variables. For more advanced configurations please use your own
configuration files or deployment server.

- `SPLUNK_ENABLE_DEPLOY_SERVER='true'` - enable deployment server on Indexer.
    - Available: *splunk* image only.
- `SPLUNK_DEPLOYMENT_SERVER='<servername>:<port>` - [configure deployment
    client](http://docs.splunk.com/Documentation/Splunk/latest/Updating/Configuredeploymentclients).
    Set deployment server url.
    - Example: `--env SPLUNK_DEPLOYMENT_SERVER='splunkdeploymentserver:8089'`.
    - Available: *splunk* and *forwarder* images only.
- `SPLUNK_ENABLE_LISTEN=<port>` - enable [receiving](http://docs.splunk.com/Documentation/Splunk/latest/Forwarding/Enableareceiver).
    - Additional configuration is available using `SPLUNK_ENABLE_LISTEN_ARGS`
        environment variable.
    - Available: *splunk* and *light* images only.
- `SPLUNK_FORWARD_SERVER=<servername>:<port>` - [forward](http://docs.splunk.com/Documentation/Splunk/latest/Forwarding/Deployanixdfmanually)
    data to indexer.
    - Additional configuration is available using `SPLUNK_FORWARD_SERVER_ARGS`
        environment variable.
    - Additional forwarders can be set up using `SPLUNK_FORWARD_SERVER_<1..30>`
        and `SPLUNK_FORWARD_SERVER_<1..30>_ARGS`.
    - Example: `--env SPLUNK_FORWARD_SERVER='splunkindexer:9997' --env
        SPLUNK_FORWARD_SERVER_ARGS='method clone' --env
        SPLUNK_FORWARD_SERVER_1='splunkindexer2:9997' --env
        SPLUNK_FORWARD_SERVER_1_ARGS='-method clone'`.
    - Available: *splunk* and *forwarder* images only.
- `SPLUNK_ADD='<monitor|add> <what_to_monitor|what_to_add>'` - execute add command,
    for example to [monitor files](http://docs.splunk.com/Documentation/Splunk/latest/Data/MonitorfilesanddirectoriesusingtheCLI)
    or [listen](http://docs.splunk.com/Documentation/Splunk/latest/Data/Monitornetworkports) on specific ports.
    - Additional add commands can be executed (up to 30) using
        `SPLUNK_ADD_<1..30>`.
    - Example `--env SPLUNK_ADD='udp 1514' --env SPLUNK_ADD_1='monitor /var/log/*'`.
    - Available: all images.
- `SPLUNK_CMD='any splunk command'` - execute any splunk command.
    - Additional commands can be executed (up to 30) using
        `SPLUNK_CMD_<1..30>`.
    - Example `--env SPLUNK_CMD='edit user admin -password random_password -role
        admin -auth admin:changeme'`.
- `SPLUNK_START_ARGS='--accept-license --answer-yes --no-prompt"` - Splunk requires you to accept the
  license, I (maintainer of this image) don't want do that for you, so please
  add this on your own or start container with `-it` switch. Also in case of upgrades
  Splunk might ask you for migration steps, this is why it is good to have
  `--answer-yes --no-prompt` in it as well.

#### Example

> This is just a simple example to show how configuration works, do not consider
> it as a *best practice* example.

```
> echo "Creating docker network, so all containers will see each other"
> docker network create splunk
> echo "Starting deployment server for forwarders"
> docker run -d --net splunk \
    --hostname splunkdeploymentserver \
    --name splunkdeploymentserver \
    --publish 8000 \
    --env SPLUNK_ENABLE_DEPLOY_SERVER=true \
    --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" \
    johnsandiford/splunk
> echo "Starting indexer 1"
> docker run -d --net splunk \
    --hostname splunkindexer1 \
    --name splunkindexer1 \
    --publish 8000 \
    --env SPLUNK_ENABLE_LISTEN=9997 \
    --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" \
    johnsandiford/splunk
> echo "Starging indexer 2"
> docker run --rm --net splunk \
    --hostname splunkindexer2 \
    --name splunkindexer2 \
    --publish 8000 \
    --env SPLUNK_ENABLE_LISTEN=9997 \
    --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" \
    johnsandiford/splunk
> echo "Starting forwarder, which forwards data to 2 indexers by cloning events"
> docker run -d --net splunk \
    --name forwarder \
    --hostname forwarder \
    --env SPLUNK_FORWARD_SERVER='splunkindexer1:9997' \
    --env SPLUNK_FORWARD_SERVER_ARGS='-method clone' \
    --env SPLUNK_FORWARD_SERVER_1="splunkindexer2:9997" \
    --env SPLUNK_FORWARD_SERVER_1_ARGS="-method clone" \
    --env SPLUNK_ADD='udp 1514' \
    --env SPLUNK_DEPLOYMENT_SERVER='splunkdeploymentserver:8089' \
    --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" \
    johnsandiford/splunk:forwarder
```

After that you will be able to forward syslog data to the *udp* port of
container *forwarder* (we do not publish port, so only from internal
containers). You should see all the data on both indexers. Also you should see
forwarder registered with deployment server.

## Upgrade from previous version

Upgrade example below

```
# Use data volume container to persist data between upgrades
docker run --name vsplunk -v /opt/splunk/etc -v /opt/splunk/var busybox
# Start old version of Splunk Enterprise
docker run --hostname splunk --name splunk --volumes-from=vsplunk -p 8000:8000 -d --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" johnsandiford/splunk:6.5.2
# Stop Splunk Enterprise container
docker stop splunk
# Remove Splunk Enterprise container
docker rm -v splunk
# Start Splunk Enterprise container with new version
docker run --hostname splunk --name splunk --volumes-from=vsplunk -p 8000:8000 -d --env SPLUNK_START_ARGS="--accept-license --answer-yes --no-prompt" johnsandiford/splunk:6.5.0
```
=======
>>>>>>> splunk/master
