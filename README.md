# docker2aci - Convert docker images to ACI

[![Build Status](https://semaphoreci.com/api/v1/projects/4472761c-2b88-41f2-b2de-bf0447a8a290/610597/badge.svg)](https://semaphoreci.com/appc/docker2aci)

docker2aci is a small library and CLI binary that converts Docker images to
[ACI][aci]. It takes as input either a file generated by "docker save" or a
Docker registry URL. It gets all the layers of a Docker image and squashes them
into an ACI image. Optionally, it can generate one ACI for each layer, setting
the correct dependencies.

All ACIs generated are compressed with gzip by default. Compression can be
disabled by specifying `--compression=none`.


## Build

Requirements: golang

	git clone git://github.com/appc/docker2aci
	cd docker2aci
	./build.sh

## Volumes

Docker Volumes get converted to mountPoints in the [Image Manifest
Schema][imageschema]. Since mountPoints need a name and Docker Volumes don't,
docker2aci generates a name by appending the path to `volume-` replacing
non-alphanumeric characters with dashes. That is, if a Volume has `/var/tmp`
as path, the resulting mountPoint name will be `volume-var-tmp`.

When the docker2aci CLI binary converts a Docker Volume to a mountPoint it will
print its name, path and whether it is read-only or not.

## Ports

Docker Ports get converted to ports in the [Image Manifest
Schema][imageschema]. The resulting port name will be the port number and the
protocol separated by a dash. For example: `6379-tcp`.

## CLI examples

```
$ docker2aci docker://busybox
Downloading sha256:55dc925c23d: [==============================] 674 KB/674 KB
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B

Generated ACI(s):
library-busybox-latest.aci
$ actool --debug validate library-busybox-latest.aci
library-busybox-latest.aci: valid app container image
```

```
$ /docker2aci --nosquash docker://quay.io/coreos/etcd:latest
Downloading sha256:f05e5379dcb: [==============================] 3.98 MB/3.98 MB
Downloading sha256:af1897d2d32: [==============================] 3.5 MB/3.5 MB
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B

Converted ports:
        name: "2379-tcp", protocol: "tcp", port: 2379, count: 1, socketActivated: false
        name: "2380-tcp", protocol: "tcp", port: 2380, count: 1, socketActivated: false
        name: "4001-tcp", protocol: "tcp", port: 4001, count: 1, socketActivated: false
        name: "7001-tcp", protocol: "tcp", port: 7001, count: 1, socketActivated: false

Generated ACI(s):
coreos-etcd-d21dd9a5886270b7c2c379c02fc548e0696b139c43bb12fdb2d9b63409717485-latest-linux-amd64-3.aci
coreos-etcd-620329641f386e62c7b0e0fa60a9acef100e71058124ddc7f1969557c72b2458-latest-linux-amd64-2.aci
coreos-etcd-9cd3f08f7ccfaad24c73757a5b4f79601f2790726d6ccdd556a82e5c9c5ddbfa-latest-linux-amd64-1.aci
coreos-etcd-9cd3f08f7ccfaad24c73757a5b4f79601f2790726d6ccdd556a82e5c9c5ddbfa-latest-linux-amd64-0.aci
```

```
$ docker save -o ubuntu.docker ubuntu
$ docker2aci ubuntu.docker
Extracting 706766fe1019
Extracting a62a42e77c9c
Extracting 2c014f14d3d9
Extracting b7cf8f0d9e82

Generated ACI(s):
ubuntu-latest.aci
$ actool --debug validate ubuntu-latest.aci
ubuntu-latest.aci: valid app container image
```

```
$ docker2aci docker://redis
Downloading sha256:c666c10c893: [==============================] 37.2 MB/37.2 MB
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:d6f52360d0a: [==============================] 1.69 KB/1.69 KB
Downloading sha256:8c3a687fd4c: [==============================] 5.93 MB/5.93 MB
Downloading sha256:15554e0e598: [==============================] 109 KB/109 KB 
Downloading sha256:3286d490a29: [==============================] 611 KB/611 KB 
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3d89b95a63: [==============================] 3.04 MB/3.04 MB
Downloading sha256:1c4db557158: [==============================] 98 B/98 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a1a961e320b: [==============================] 196 B/196 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B
Downloading sha256:a3ed95caeb0: [==============================] 32 B/32 B

Converted volumes:
        name: "volume-data", path: "/data", readOnly: false

Converted ports:
        name: "6379-tcp", protocol: "tcp", port: 6379, count: 1, socketActivated: false

Generated ACI(s):
library-redis-latest.aci
$ actool --debug validate library-redis-latest.aci
library-redis-latest.aci: valid app container image
```

[aci]: https://github.com/appc/spec/blob/master/SPEC.md#app-container-image
[imageschema]: https://github.com/appc/spec/blob/master/spec/aci.md#image-manifest-schema
