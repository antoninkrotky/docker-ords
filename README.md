# ORDS Dockerfile

<!-- TOC depthFrom:2 insertAnchor:true -->

- [Pre-Install](#pre-install)
- [Build ORDS Docker Image](#build-ords-docker-image)
- [Run Container](#run-container)
  - [Generate Configuration](#generate-configuration)
  - [Configuration Exists](#configuration-exists)
  - [Health Check](#health-check)
  - [Logs](#logs)
- [Container Parameters](#container-parameters)
- [Development](#development)

<!-- /TOC -->

This docker container will run ORDS in standalone mode.

_The reason why this image is not posted on [Docker Hub](https://hub.docker.com) is due to Oracle requiring that you download and accept their terms and conditions for ORDS._

<a id="markdown-pre-install" name="pre-install"></a>
## Pre-Install


Download [ORDS](http://www.oracle.com/technetwork/developer-tools/rest-data-services/downloads/index.html). I'll assume that this is stored in your `~/Downloads` directory. The downloaded file will look like `~/Downloads/ords.18.1.1.95.1251.zip`

```bash
-- Assuming that you have a folder called ~/docker/ords
cd ~/docker/ords

-- Clone this repo
git clone https://github.com/martindsouza/docker-ords.git
cd docker-ords

-- Extract the ords.war file from the ords.zip download
unzip ~/Downloads/ords.*.zip ords.war
```

<a id="markdown-build-ords-docker-image" name="build-ords-docker-image"></a>
## Build ORDS Docker Image

Note: tagging with ORDS version number allows you to have multiple ORDS images for each ORDS release.

```bash
cd ~/docker/ords/docker-ords
ORDS_VERSION=18.3.0
docker build -t ords:$ORDS_VERSION .
```

<a id="markdown-run-container" name="run-container"></a>
## Run Container

This image allows you to run the ORDS container to either generate the ORDS configuration or use an existing one. Full explanations of all the supported [parameters](#parameters) is below.

<a id="markdown-generate-configuration" name="generate-configuration"></a>
### Generate Configuration

Running the ORDS container this way will setup the ORDS configuration in the mapped volume drive (`/opt/ords`) and only needs to be run once. The following commands demonstrate how to do this. It also has an optional `--rm` parameter that will remove the container after first use. This is used since we can use a "simpler" `run` command once the ORDS configuration exists. It also prevents the `-e` variables which contain the passwords to be attached to a container. 

```bash
#Note: DB_PORT is NOT the port that you mapped to your Oracle DB Docker image. It's the port that the database natively has open.
# It's recommended to leave it as 1521
# Optional: If DB is in another Docker machine include: --network=<docker_network_name> \
# ~/docker/ords/ords-18.1.1/config is the directory where the ORDS configuration will be saved. If it doesn't exist Docker will create it.
docker run -it --rm \
  --network=oracle_network \
  -e TZ=America/Edmonton \
  -e DB_HOSTNAME=oracle \
  -e DB_PORT=1521 \
  -e DB_SERVICENAME=orclpdb513.localdomain \
  -e APEX_PUBLIC_USER_PASS=oracle \
  -e APEX_LISTENER_PASS=oracle \
  -e APEX_REST_PASS=oracle \
  -e ORDS_PASS=oracle \
  -e SYS_PASS=Oradoc_db1 \
  --volume ~/docker/ords/ords-18.1.1/config:/opt/ords \
  --volume ~/docker/apex/5.1.3/images:/ords/apex-images \
  -p 32513:8080 \
  ords:18.1.1
```

On your laptop go to [localhost:32513/ords](http://localhost:32513/ords).

<a id="markdown-configuration-exists" name="configuration-exists"></a>
### Configuration Exists

In this case ORDS will assume that your configuration exists (found in the mapped `/opt/ords` folder). A few differences to note from previous `run` command:

- Includes a `name` attribute. This will allows to reference the container by name when using `docker start ords` and `docker stop ords`.
- Does not include all database login information (since exists in configuration file)
- Does not self-remove
- `-d` (detached mode - optional) is used so that it does not lock the current terminal screen

```bash
docker run -it -d \
  --name=ords \
  --network=oracle_network \
  -e TZ=America/Edmonton \
  --volume ~/docker/ords/ords-18.1.1/config:/opt/ords \
  --volume ~/docker/apex/5.1.4/images:/ords/apex-images \
  -p 32514:8080 \
  ords:18.1.1
```

<a id="markdown-health-check" name="health-check"></a>
### Health Check

This container includes a healthcheck to ensure that ORDS is working properly:

```bash
docker ps

# Should result in something like the following
# Note the (healthy) status
CONTAINER ID  IMAGE        COMMAND                 CREATED       STATUS                  PORTS                    NAMES
b7694a2d62ba  ords:18.1.1  "/ords/config-run-or…"  15 hours ago  Up 15 hours (healthy)   0.0.0.0:32513->8080/tcp  ords
```

<a id="markdown-logs" name="logs"></a>
### Logs

If you want to see the logs from ORDS that would normally output on the screen you can run:

```bash
docker logs ords
```

Where `ords` is the name of your container.

<a id="markdown-container-parameters" name="container-parameters"></a>
## Container Parameters
Parameter | Description
--- | ---
`--name` | Optional: Name to label container
`--network` | Optional: If your database is part of a Docker network, attach this container to same network.
`-e TZ` | Optional: Timezone for ORDS
`-e DB_HOSTNAME` | Hostname of Oracle DB
`-e DB_PORT` | TNS port to DB. Note: If on a Docker Network don't use the mapped port. Most likely this will always be `1521`
`-e DB_SERVICENAME` | DB servicename
`-e APEX_PUBLIC_USER_PASS` | `APEX_PUBLIC_USER` password (to be created)
`-e APEX_LISTENER_PASS` | `APEX_LISTENER` password (to be created)
`-e APEX_REST_PASS` | `APEX_REST` password (to be created)
`-e ORDS_PASS` | `ORDS_PUBLIC_USER` password (to be created)
`-e SYS_PASS` | `SYS` password
`--volume <local dir>:/ords/apex-images` | Directory that contains images for APEX
`--volume <local dir>:/opt/ords`  | Optional: Directory to/that contains ORDS config. If this is not provided, the configuration will be saved in the container and will **not** be available if the container is deleted. If `defaults.xml` is not found in the folder ORDS will try to install.
`-p 1234:8080`  |  Port mapping, `8080` is the port in the container and can not be modified.


<a id="markdown-development" name="development"></a>
## Development

Please read the [development](docs/development.md) documentation for more info on how to help develop this Docker image.
