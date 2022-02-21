# Syslog-NG server on Docker

## Introduction

In this section, we'll start a Docker container with syslog-ng based on Debian.

As a central log server, the syslog-ng image exposes three different ports, where it can receive log messages:

- Syslog UDP: 514
- Syslog TCP: 601
- Syslog TLS: 6514

### Requirements

* Familiarity with [Docker](https://www.docker.com/).
* Familiarity with [syslog-ng](https://www.syslog-ng.com/community/b/blog/posts/central-log-server-docker/).
* Docker Desktop installed locally.

## Let's get the image and start the container

1. Get the [official Docker](https://hub.docker.com/r/balabit/syslog-ng) image from Docker hub.
2. Create local directories, for persistance.
3. Start the `syslog-ng` container.

### Get the official rsyslog/syslog_appliance_alpine

Open a `terminal` and change the directory to where all the `Syslog-NG` files are hosted.

```sh
cd Syslog-NG
```

### Create the directory struture

```sh
mkdir conf logs
```

> `/config` Holds the container configuration. This volume can be mounted read-only after initial population with sample files.
`/logs` This holds log files if the container is configured to write them.

### Pull the official image from Dockder hub.

```sh
docker pull balabit/syslog-ng
```

### Start Rsyslog

Note that the directory `$PWD/conf` folder should be populated with a file named `syslog-ng.conf`. See below for a minimal configuration file.

```conf
@version: 3.9

source s_net {
   udp(
     ip("0.0.0.0")
   );
   syslog(
     ip("0.0.0.0")
   );
 };
 destination d_file {
   file("/var/log/syslog");
 };
 log {source(s_net); destination(d_file); };
```

### Start syslog

Adjust the timezone, network and IP address.

```sh
docker run -d \
--name syslog1 \
--hostname syslog1 \
--env TZ='EAST+5EDT,M3.2.0/2,M11.1.0/2' \
-v $PWD/conf/syslog-ng.conf:/etc/syslog-ng/syslog-ng.conf \
-v $PWD/logs:/var/log \
--network frontend \
--ip 172.31.10.9 \
balabit/syslog-ng
```

### Test Syslog-NG

1. To test the server, start another instance of the `syslog-ng` server:

```sh
docker run -d \
--name syslog2 \
--hostname syslog2 \
--env TZ='EAST+5EDT,M3.2.0/2,M11.1.0/2' \
--network frontend \
balabit/syslog-ng
```

2. Execute a shell in the container:

```sh
docker exec -it syslog2 bash
```

At the prompt: `root@syslog2:/#`,

3. to test `601/tcp`, use the following command:

```sh
loggen -i -S -P syslog1 601
```

4. to test `514/udp`, use the following command:

```sh
loggen -i -D -P syslog1 514
```

### Clean the log file

Clean the log file from the log generated in the step above:

```sh
: > logs/syslog
```

## License

This project is licensed under the [MIT license](LICENSE).

[*^ back to top*](#Syslog-NG-server-on-Docker)
