[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-kafka/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-kafka/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/kafka)](https://hub.docker.com/r/bitnami/kafka/)
[![Slack](http://slack.oss.bitnami.com/badge.svg)](http://slack.oss.bitnami.com)

# What is kafka?
Apache Kafka is a distributed streaming platform used for building real-time data pipelines and
streaming apps. It is horizontally scalable, fault-tolerant, wicked fast, and runs in production in
thousands of companies. Kafka requires a connection to a Zookeeper service.

[https://kafka.apache.org/](https://kafka.apache.org/)


# TL;DR;

## Docker Compose

```yaml
version: '2'
services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092:9092'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* Bitnami images are built on CircleCI and automatically pushed to the Docker Hub.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.

# Get this image

The recommended way to get the Bitnami Kafka Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/kafka).

```bash
docker pull bitnami/kafka:latest
```

To use a specific version, you can pull a versioned tag. You can view the
[list of available versions](https://hub.docker.com/r/bitnami/kafka/tags/)
in the Docker Hub Registry.

```bash
docker pull bitnami/kafka:[TAG]
```

If you wish, you can also build the image yourself.

```bash
docker build -t bitnami/kafka:latest https://github.com/bitnami/bitnami-docker-kafka.git
```

# Persisting your data

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

**Note!**
If you have already started using your database, follow the steps on
[backing up](#backing-up-your-container) and [restoring](#restoring-a-backup) to pull the data from your running container down to your host.

The image exposes a volume at `/bitnami/kafka` for the Kafka data and configurations. For persistence you can mount a directory at this location from your host. If the mounted directory is empty, it will be initialized on the first run.

Using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092:9092'
    volumes:
      - /path/to/kafka-persistence:/bitnami/kafka
```

# Connecting to other containers

Using [Docker container networking](https://docs.docker.com/engine/userguide/networking/), a Kafka server running inside a container can easily be accessed by your application containers.

Containers attached to the same network can communicate with each other using the container name as the hostname.

## Using the Command Line

In this example, we will create a Kafka client instance that will connect to the server instance that is running on the same docker network as the client.

### Step 1: Create a network

```bash
$ docker network create app-tier --driver bridge
```

### Step 2: Launch the Zookeeper server instance

Use the `--network app-tier` argument to the `docker run` command to attach the Zookeeper container to the `app-tier` network.

```bash
$ docker run -d --name zookeeper-server \
    --network app-tier \
    bitnami/zookeeper:latest
```

### Step 2: Launch the Kafka server instance

Use the `--network app-tier` argument to the `docker run` command to attach the Kafka container to the `app-tier` network.

```bash
$ docker run -d --name kafka-server \
    --network app-tier \
    -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
    bitnami/kafka:latest
```

### Step 3: Launch your Kafka client instance

Finally we create a new container instance to launch the Kafka client and connect to the server created in the previous step:

```bash
$ docker run -it --rm \
    --network app-tier \
    -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
    bitnami/kafka:latest kafka-topics.sh --list  --zookeeper zookeeper-server:2181
```

## Using Docker Compose

When not specified, Docker Compose automatically sets up a new network and attaches all deployed services to that network. However, we will explicitly define a new `bridge` network named `app-tier`. In this example we assume that you want to connect to the Kafka server from your own custom application image which is identified in the following snippet by the service name `myapp`.

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    networks:
      - app-tier
  kafka:
    image: 'bitnami/kafka:latest'
    networks:
      - app-tier
  myapp:
    image: 'YOUR_APPLICATION_IMAGE'
    networks:
      - app-tier
```

> **IMPORTANT**:
>
> 1. Please update the `YOUR_APPLICATION_IMAGE` placeholder in the above snippet with your application image
> 2. In your application container, use the hostname `kafka` to connect to the Kafka server

Launch the containers using:

```bash
$ docker-compose up -d
```

# Configuration


The configuration can easily be setup with the Bitnami Kafka Docker image using the following environment variables:

- `ALLOW_PLAINTEXT_LISTENER`: Allow to use the PLAINTEXT listener. Default: **no**
- `KAFKA_PORT_NUMBER`: Kafka port. Default: **9092**
- `KAFKA_BROKER_ID`: ID of the Kafka node. Default: **-1**, unless `BROKER_ID_COMMAND` is set.
- `BROKER_ID_COMMAND`: the command to issue in order to determine the ID of the Kafka node.  Ignored if `KAFKA_BROKER_ID` is of nonzero length.
- `KAFKA_NUM_NETWORK_THREADS`: The number of threads handling network requests.
- `KAFKA_NUM_IO_THREADS`: The number of threads doing disk I/O. Default: **3**
- `KAFKA_SOCKET_SEND_BUFFER_BYTES`: The send buffer (SO_SNDBUF) used by the socket server. Default: **102400**
- `KAFKA_SOCKET_RECEIVE_BUFFER_BYTES`: The receive buffer (SO_RCVBUF) used by the socket server. Default: **102400**
- `KAFKA_SOCKET_REQUEST_MAX_BYTES`: The maximum size of a request that the socket server will accept (protection against OOM). Default: **104857600**
- `KAFKA_LOGS_DIRS`: A comma separated list of directories under which to store log files. Default: **/opt/bitnami/kafka/data**
- `KAFKA_DELETE_TOPIC_ENABLE`: Switch to enable topic deletion or not, default value is false. Default: **false**
- `KAFKA_LISTENERS`: The address the socket server listens on. Default: **PLAINTEXT://:9092**
- `KAFKA_ADVERTISED_LISTENERS`: Hostname and port the broker will advertise to producers and consumers. Default: **PLAINTEXT://:9092**
- `KAFKA_NUM_PARTITIONS`: The default number of log partitions per topic. Default: **1**
- `KAFKA_NUM_RECOVERY_THREADS_PER_DATA_DIR` The number of threads per data directory to be used for log recovery at startup and flushing at shutdown. Default: **1**
- `KAFKA_LOG_FLUSH_INTERVAL_MESSAGES`: The number of messages to accept before forcing a flush of data to disk. Default: **10000**
- `KAFKA_LOG_FLUSH_INTERVAL_MS`: The maximum amount of time a message can sit in a log before we force a flush. Default: **1000**
- `KAFKA_LOG_RETENTION_HOURS`: The minimum age of a log file to be eligible for deletion due to age. Default: **168**
- `KAFKA_LOG_RETENTION_BYTES`: A size-based retention policy for logs. Default: **1073741824**
- `KAFKA_SEGMENT_BYTES`: The maximum size of a log segment file. When this size is reached a new log segment will be created. Default: **1073741824**
- `KAFKA_LOG_RETENTION_CHECK_INTERVALS_MS`: The interval at which log segments are checked to see if they can be deleted. Default: **300000**
- `KAFKA_ZOOKEEPER_CONNECT`: Comma separated host:port pairs, each corresponding to a Zookeeper Server. Default: **localhost:2181**
- `KAFKA_ZOOKEEPER_CONNECT_TIMEOUT_MS`: Timeout in ms for connecting to zookeeper. Default: **6000**
- `KAFKA_INTER_BROKER_USER`: Kafka inter broker communication user. Default: admin. Default: **admin**
- `KAFKA_INTER_BROKER_PASSWORD`: Kafka inter broker communication password. Default: **bitnami**
- `KAFKA_BROKER_USER`: Kafka client user. Default: **user**
- `KAFKA_BROKER_PASSWORD`: Kafka client user password. Default: **bitnami**
- `KAFKA_ZOOKEEPER_USER`: Kafka Zookeeper user. No defaults
- `KAFKA_ZOOKEEPER_PASSWORD`: Kafka Zookeeper user password. No defaults
- `KAFKA_CERTIFICATE_PASSWORD`: Password for certificates. Default: **bitnami1**
- `KAFKA_HEAP_OPTS`: Kafka's Java Heap size. Default: **-Xmx1024m -Xms1024m**



```bash
docker run --name kafka -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 bitnami/kafka:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    volumes:
      - 'zookeeper_data:/bitnami/zookeeper'
  kafka:
    image: 'bitnami/kafka:0'
    ports:
      - '9092:9092'
    volumes:
      - 'kafka_data:/bitnami/kafka'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181

volumes:
  zookeeper_data:
    driver: local
  kafka_data:
    driver: local
```

## Security

The Bitnami Kafka docker image disables the PLAINTEXT listener for security reasons.
You can enable the PLAINTEXT listener by adding this environment variable, but remember that this
configuration is not recommended for production.

```
ALLOW_PLAINTEXT_LISTENER=yes
```

In order to configure SASL authentication over SSL, you should just define the proper listener by
passing the following env vars:

```
KAFKA_LISTENERS=SASL_SSL://:9092
KAFKA_ADVERTISED_LISTENERS=SASL_SSL://:9092
```

You can use your own certificates for SSL. You can drop your Java Key Stores files into
`/bitnami/kafka/conf/certs`.
If the JKS is password protected (recommended), you will need to provide it to get access to the keystores:

`KAFKA_CERTITICATE_PASSWORD=myCertificatePassword`

This script will help you with the creation of the JKS and certificates. Use the same password for
all them:

https://raw.githubusercontent.com/confluentinc/confluent-platform-security-tools/master/kafka-generate-ssl.sh



### InterBroker communications

By default, communications that happens between brokers are authenticated.
You can provide your own credentials using this environment variables:


- `KAFKA_INTER_BROKER_USER`: Kafka inter broker communication user. Default: **admin**
- `KAFKA_INTER_BROKER_PASSWORD`: Kafka inter broker communication password. Default: **bitnami**

### Kafka client configuration

By default, any Kafka client needs to authenticate before can connect to a broker.
You can provide your own credentials using this environment variables:

- `KAFKA_BROKER_USER`: Kafka client user. Default: **user**
- `KAFKA_BROKER_PASSWORD`: Kafka client user password. Default: **bitnami**


### Kafka ZooKeeper client configuration

In order to authenticate Kafka against a Zookeeper server with SASL authentication you should provide
the next environment variables:

- `KAFKA_ZOOKEEPER_USER`: Kafka Zookeeper user. No defaults.
- `KAFKA_ZOOKEEPER_PASSWORD`: Kafka Zookeeper user password. No defaults.

Below you can see a complete Docker Compose example:


```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
     - '2181:2181'
    environment:
      - ZOO_ENABLE_AUTH=yes
      - ZOO_SERVER_USERS=kafka
      - ZOO_SERVER_PASSWORDS=kafka_password
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_LISTENERS=SASL_SSL://:9092
      - KAFKA_ADVERTISED_LISTENERS=SASL_SSL://:9092
      - KAFKA_ZOOKEEPER_USER=kafka
      - KAFKA_ZOOKEEPER_PASSWORD=kafka_password
```

### Connecting services with security enabled

In order to get the required credentials to consume and  produce messages you need to provide the
credentials in the client. If your Kafka client allows it, use the credentials you've provided.

While producing and consuming messages using the `bitnami/kafka` image, you'll need to point to the
`consumer.properties` and/or `producer.properties` file, which contains the needed configuration
to work. You can find this files in the `/bitnami/kafka/config` directory

Use this to generate messages using a secure setup

```bash
export KAFKA_OPTS="-Djava.security.auth.login.config=/opt/bitnami/kafka/conf/kafka_jaas.conf"
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic test --producer.config /opt/bitnami/kafka/conf/producer.properties
```
Use this to consume messages using a secure setup

```bash
export KAFKA_OPTS="-Djava.security.auth.login.config=/opt/bitnami/kafka/conf/kafka_jaas.conf"
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic test --consumer.config /opt/bitnami/kafka/conf/consumer.properties
```
If you use other tools to use your Kafka cluster, you'll need to provide the required information.
You can find the required information in the files located at `/bitnami/kafka/conf` directory.

## Setting up a Kafka Cluster

A Kafka cluster can easily be setup with the Bitnami Kafka Docker image using the following environment variables:

 - `KAFKA_ZOOKEEPER_CONNECT`: Comma separated host:port pairs, each corresponding to a Zookeeper Server.


Create a Docker network to enable visibility to each other via the docker container name

```bash
docker network create app-tier --driver bridge
```

### Step 1: Create the first node for Zookeeper

The first step is to create one Zookeeper instance.

```bash
docker run --name zookeeper \
  --network app-tier \
  -p 2181:2181 \
  bitnami/zookeeper:latest
```

### Step 2: Create the first node for Kafka

The first step is to create one Kafka instance.

```bash
docker run --name kafka1 \
  --network app-tier \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -p 9092:9092 \
  bitnami/kafka:latest
```

### Step 2: Create the second node

Next we start a new Kafka container.

```bash
docker run --name kafka2 \
  --network app-tier \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -p 9092:9092 \
  bitnami/kafka:latest
```

### Step 3: Create the third node

Next we start another new Kafka container.

```bash
docker run --name kafka3 \
  --network app-tier \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  -p 9092:9092 \
  bitnami/kafka:latest
```
You now have a Kafka cluster up and running. You can scale the cluster by adding/removing slaves without incurring any downtime.

With Docker Compose, topic replication can be setup using:

```yaml
version: '2'

services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
     - '2181:2181'
  kafka1:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
  kafka2:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
  kafka3:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092'
    environment:
      - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
```

Then, you can create a replicated topic with:

```bash
root@kafka1:/# /opt/bitnami/kafka/bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --topic mytopic --partitions 3 --replication-factor 3
Created topic "mytopic".

root@kafka1:/# /opt/bitnami/kafka/bin/kafka-topics.sh --describe --zookeeper zookeeper:2181 --topic mytopic
Topic:mytopic   PartitionCount:3        ReplicationFactor:3     Configs:
        Topic: mytopic  Partition: 0    Leader: 2       Replicas: 2,3,1 Isr: 2,3,1
        Topic: mytopic  Partition: 1    Leader: 3       Replicas: 3,1,2 Isr: 3,1,2
        Topic: mytopic  Partition: 2    Leader: 1       Replicas: 1,2,3 Isr: 1,2,3
```

## Configuration
The image looks for configuration in the `config/` directory of `/bitnami/kafka`.

```
docker run --name kafka -v /path/to/my_custom_conf_directory:/bitnami/kafka bitnami/kafka:latest
```
After that, your changes will be taken into account in the server's behaviour.

### Step 1: Run the Kafka image

Run the Kafka image, mounting a directory from your host.

Using Docker Compose:

```yaml
version: '2'

services:
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092:9092'
    volumes:
      - /path/to/kafka-persistence:/bitnami/kafka
```

### Step 2: Edit the configuration

Edit the configuration on your host using your favorite editor.

```bash
vi /path/to/kafka-persistence/config/server.properties
```

### Step 3: Restart Kafka

After changing the configuration, restart your Kafka container for changes to take effect.

```bash
docker restart kafka
```

or using Docker Compose:

```bash
docker-compose restart kafka
```


# Logging

The Bitnami Kafka Docker image sends the container logs to the `stdout`. To view the logs:

```bash
docker logs kafka
```

or using Docker Compose:

```bash
docker-compose logs kafka
```

You can configure the containers [logging driver](https://docs.docker.com/engine/admin/logging/overview/) using the `--log-driver` option if you wish to consume the container logs differently. In the default configuration docker uses the `json-file` driver.

# Maintenance

## Backing up your container

To backup your data, configuration and logs, follow these simple steps:

### Step 1: Stop the currently running container

```bash
docker stop kafka
```

or using Docker Compose:

```bash
docker-compose stop kafka
```

### Step 2: Run the backup command

We need to mount two volumes in a container we will use to create the backup: a directory on your host to store the backup in, and the volumes from the container we just stopped so we can access the data.

```bash
docker run --rm -v /path/to/kafka-backups:/backups --volumes-from kafka busybox \
  cp -a /bitnami/kafka:latest /backups/latest
```

or using Docker Compose:

```bash
docker run --rm -v /path/to/kafka-backups:/backups --volumes-from `docker-compose ps -q kafka` busybox \
  cp -a /bitnami/kafka:latest /backups/latest
```

## Restoring a backup

Restoring a backup is as simple as mounting the backup as volumes in the container.

```bash
docker run -v /path/to/kafka-backups/latest:/bitnami/kafka bitnami/kafka:latest
```

or using Docker Compose:

```yaml
version: '2'

services:
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092:9092'
    volumes:
      - /path/to/kafka-backups/latest:/bitnami/kafka
```

## Upgrade this image

Bitnami provides up-to-date versions of Kafka, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

### Step 1: Get the updated image

```bash
docker pull bitnami/kafka:latest
```

or if you're using Docker Compose, update the value of the image property to
`bitnami/kafka:latest`.

### Step 2: Stop and backup the currently running container

Before continuing, you should backup your container's data, configuration and logs.

Follow the steps on [creating a backup](#backing-up-your-container).

### Step 3: Remove the currently running container

```bash
docker rm -v kafka
```

or using Docker Compose:


```bash
docker-compose rm -v kafka
```

### Step 4: Run the new image

Re-create your container from the new image, [restoring your backup](#restoring-a-backup) if necessary.

```bash
docker run --name kafka bitnami/kafka:latest
```

or using Docker Compose:

```bash
docker-compose start kafka
```

# Notable Changes

## 0.10.2.1-r3

- The kafka container has been migrated to a non-root container approach. Previously the container run as `root` user and the kafka daemon was started as `kafka` user. From now own, both the container and the kafka daemon run as user `1001`.
  As a consequence, the configuration files are writable by the user running the kafka process.

## 0.10.2.1-r0

- New Bitnami release


# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-kafka/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-kafka/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-kafka/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# Community

Most real time communication happens in the `#containers` channel at [bitnami-oss.slack.com](http://bitnami-oss.slack.com); you can sign up at [slack.oss.bitnami.com](http://slack.oss.bitnami.com).

Discussions are archived at [bitnami-oss.slackarchive.io](https://bitnami-oss.slackarchive.io).

# License

Copyright (c) 2015-2018 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
