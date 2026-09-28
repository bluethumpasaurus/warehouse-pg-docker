---
### Introduction:

The repo provides multiple Docker configurations for setting up WarehousePG in both single-node and multi-node configurations.

This instructions below leverage the repo, giving explicit instructions for deploying a WarehousePG 7x singlenode cluster running on Rocky Linux 9.

Once connected to the container, the end result will be as shown below:

```
[gpadmin@whpgdb-primary ~]$ cat /etc/redhat-release
Rocky Linux release 9.8 (Blue Onyx)
[gpadmin@whpgdb-primary ~]$
[gpadmin@whpgdb-primary ~]$ psql whpgtest -c "SELECT * from gp_segment_configuration;"
 dbid | content | role | preferred_role | mode | status | port |    hostname    |    address     |                datadir
------+---------+------+----------------+------+--------+------+----------------+----------------+---------------------------------------
    1 |      -1 | p    | p              | n    | u      | 5432 | whpgdb-primary | whpgdb-primary | /whpgdata/coordinator/whpgsne-1
    2 |       0 | p    | p              | n    | u      | 6000 | whpgdb-primary | whpgdb-primary | /whpgdata/segments/whpgdata1/whpgsne0
    3 |       1 | p    | p              | n    | u      | 6001 | whpgdb-primary | whpgdb-primary | /whpgdata/segments/whpgdata2/whpgsne1
(3 rows)

[gpadmin@whpgdb-primary ~]$
```


---
### Prerequisites:

+ The following Docker packages will be required:

```
docker-ce 
docker-ce-cli 
containerd.io 
docker-buildx-plugin 
docker-compose-plugin
```

+ You will also need an [EDB Repos 2.0 token](https://www.enterprisedb.com/docs/repos/getting_started/get_your_token/) to gain access to the EDB `gpsupp` repo, so that WarehousePG packages can be downloaded by Docker.

---
### Prerequisites:

+ The following Docker packages will be required:

```
docker-ce 
docker-ce-cli 
containerd.io 
docker-buildx-plugin 
docker-compose-plugin
```

+ You will also need an [EDB Repos 2.0 token](https://www.enterprisedb.com/docs/repos/getting_started/get_your_token/) to gain access to the EDB gpsupp repo, so that WarehousePG packages can be downloaded by Docker.


---
### Installing:

+ Clone the repo:

```
git clone https://github.com/EnterpriseDB/warehouse-pg-docker
```

+ `cd` to the downloaded repo's `WarehousePG7-from-RPMs-RH9-single-node` subdirectory:

```
cd warehouse-pg-docker/WarehousePG7-from-RPMs-RH9-single-node
```

+ Within the `warehouse-pg-docker/WarehousePG7-from-RPMs-RH9-single-node` directory, create a `data/` directory.
+ The presence of the `data/` directory will ensure the persistence of the container that you will next create.

```
mkdir -p data
```

**Note that** if you are installing with macOS + Docker Desktop, you will need to perform a one-time bind-mount workaround when creating the `data/` directory:

```
mkdir -p data
docker run --rm --platform=linux/amd64 -v "$(pwd)/data":/x alpine true
```

+ Setup your variables:

```
export EDBTOKEN=<your-EDB-Repos-2.0-token>
export EDBREPOSITORY=gpsupp
export DOCKER_BUILDKIT=1
```

+ Next, build the image specified in the `warehouse-pg-docker/WarehousePG7-from-RPMs-RH9-single-node` directory's `docker-compose.yml` file:
  + Most systems will require that you run the command with `sudo` - note that we specify `-E` in the command - this tells `sudo` to pass the variables that we previously set.


```
sudo -E docker compose build
```

+ Create, start, and run the container defined in the directory's `docker-compose.yml` file:

```
docker compose up -d
```


---
### Connecting to the Container:


+ Connect to the container:

```
docker compose exec sne bash
```

+ Once connected, run the commands below to persistently add the path to the WarehousePG binaries to the `gpadmin` user's shell startup:

```
echo "source /usr/local/greenplum-db/greenplum_path.sh" >> ~/.bashrc
source ~/.bashrc
```


+ Connect to WarehousePG with `psql` from within the container:

```
psql whpgtest
```


---
### Container control:

+ Check container health:

```
docker compose ps 
```

+ Watch startup / `gpinitsystem` output:

```
docker compose logs -f sne
```

+ Pause - the container is removed from "running", but the data/container config is kept:

```
docker compose stop
```

+ Resume - the same container is used, `gpstart` kicks in, and the data in the `data/` directory is left intact:

```
docker compose start
```

+ Stop and remove the container - the `data/` folder on disk is untouched:

```
docker compose down
```

+ Recreate the container - the process finds the existing `data/` directory , runs `gpstart`, and the cluster is returned to its former state:

```      
docker compose up -d

--- Re-run the below commands when reconnected to the container:

echo "source /usr/local/greenplum-db/greenplum_path.sh" >> ~/.bashrc
source ~/.bashrc    
```


---
# WarehousePG Docker Setup

This repository provides Docker configurations for setting up WarehousePG in both single-node and multi-node configurations.

## Container Restart

Currently none of the Docker labs will survive a container restart. This setup is not meant for production use.

While the data directories for all segment databases are mapped to the Docker host, this is meant for inspecting the directories, not to support a container restart.

## Table of Contents

- `WarehousePG6-from-RPMs-RH7-single-node`: WarehousePG v6, single node, installed from RPMs
- `WarehousePG6-from-source-RH7-single-node`: WarehousePG v6, single node, built from source code
- `WarehousePG7-from-RPMs-RH9-multi-node`: WarehousePG v7, coordinator + 2 segment hosts, installed from RPMs
- `WarehousePG7-from-RPMs-RH9-multi-node-standby-mirrors`: WarehousePG v7, coordinator + 2 segment hosts, standby coordinator and mirrors enabled, installed from RPMs
- `WarehousePG7-from-RPMs-RH9-single-node`: WarehousePG v7, single node, installed from RPMs
- `WarehousePG7-from-RPMs-RH9-single-node-not-installed`: WarehousePG v7, single node, installed from RPMs, database not configured (for trying out install options)
- `WarehousePG7-from-source-RH9-single-node`: WarehousePG v7, single node, built from source code

The `RH7` labs are using [CentOS](https://en.wikipedia.org/wiki/CentOS) 7, the `RH9` labs are using [Rocky Linux](https://en.wikipedia.org/wiki/Rocky_Linux) 9.

## Installation

You need an EDB token in order to download RPM packages:

To get a token:

- go to `https://enterprisedb.com/`
- Sign in
- Go to "My Account" (in the upper right corner)
- Select "Account Settings" from Dropdown
- Under "Profile", copy the first line, that's the "Repos 2.0" token
- Create the file "~/.edb-token" and copy the token into the file

Your token matches a specific repository. You should have received this information along with the token.
For EDB employees the personal token is for the "dev" repository.
Create the file "~/.edb-repository", add one of: "dev", "staging_gpsupp", "gpsupp".

Check your Docker settings, allow enough disk space, RAM and CPU for Docker.
For building all images consider 50-60 GB disk space.

Note: the containers build from source do not need an EDB token.

## Single Node Setup

A single node setup includes the coordinator and multiple segment databases (2 in this case) in a single machine or container. No network setup is required.

The following setups are single node:

- `WarehousePG6-from-RPMs-RH7-single-node`
- `WarehousePG6-from-source-RH7-single-node`
- `WarehousePG7-from-RPMs-RH9-single-node`
- `WarehousePG7-from-RPMs-RH9-single-node-not-installed`
- `WarehousePG7-from-source-RH9-single-node`

## Multi Node Setup

A multi node setup includes multiple machines or containers. One system is used for the coordinator, other systems for segment databases (2 segment hosts with 2 segment databases each in this case). Network connectivity is required, and provided by Docker.

The following setups are multi-node:

- `WarehousePG7-from-RPMs-RH9-multi-node` (no standby coordinator, no mirror segments)
- `WarehousePG7-from-RPMs-RH9-multi-node-standby-mirrors` (includes standby coordinator, includes mirror segments)

## Docker Desktop on macOS

On macOS, Docker Desktop can fail to correctly initialize bind-mounted data directories the first time a `linux/amd64` container (the labs in this repository are built for `linux/amd64`, since WarehousePG does not provide `arm64` packages) writes to them. This is a long-standing Docker Desktop for Mac issue, tracked upstream at [docker/for-mac#69](https://github.com/docker/for-mac/issues/69).

To work around this, the `Makefile` in each lab creates the `data/` directories and then, only on macOS with Docker Desktop, runs a throwaway `alpine` container against each of them (`docker run --rm --platform=linux/amd64 -v ... alpine true`) before starting the actual lab containers. This forces Docker Desktop to initialize the bind mount correctly ahead of time. This step is skipped on Linux and on other Docker setups, where it is not needed.

## Interactive Training

For detailed instructions on setting up WarehousePG from scratch, please refer to the [training](training.md) document.

The `WarehousePG7-from-RPMs-RH9-single-node-not-installed` lab can be used here, this container has the RPM packages pre-installed, but WarehousePG is not configured. Necessary files are available in `/home/gpadmin` in the container (use `make access` to drop into a shell once the container is started).

## Build WarehousePG From Source

The containers `WarehousePG6-from-source-RH7-single-node` and `WarehousePG7-from-source-RH9-single-node` build WarehousePG from source. Refer to the `Dockerfile` in each directory for detailed instructions how to build the database from source.
