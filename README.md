# Jitsi Videobridge Integrated with Node Exporter 

This is a project rolled back to stable-4548 version for compatibility with [podium jisti patch file](https://github.com/sa-mw-dach/podium/blob/master/jitsi/sapkra-ocp.patch). Also, this includes the modified jvb container with [node exporter](https://github.com/prometheus/node_exporter) for monitoring the system informations, not its colibri stats. The original README is README-orig.md.

> This project is already patched. Don't patch this [file](https://github.com/sa-mw-dach/podium/blob/master/jitsi/sapkra-ocp.patch). 

> The docker images built in this project are for k8s/OCP, not docker environment.

## Original jvb tree
```
.
├── Dockerfile
├── Makefile
└── rootfs
    ├── defaults
    │   ├── logging.properties
    │   └── sip-communicator.properties
    └── etc
        ├── cont-init.d
        │   └── 10-config
        └── services.d
            └── jvb
                └── run
```

## jvb tree of this project 
```
.
├── Dockerfile
├── Makefile
└── rootfs
    ├── defaults
    │   ├── logging.properties
    │   └── sip-communicator.properties
    └── etc
        ├── cont-init.d
        │   └── 10-config
        └── services.d
            ├── jvb
            │   └── run
            └── node_exporter
                └── run
```
I added `rootfs/etc/services.d/run` script file to [s6-overlay](https://github.com/just-containers/s6-overlay) tree to run `node_exporter` binary.

``` bash
#!/usr/bin/with-contenv bash

if [[ -z "$EXPORTER_TCP_PORT" ]]; then
  EXPORTER_TCP_PORT=9100
fi

exec /bin/bash -c '/node_exporter --web.listen-address=":$EXPORTER_TCP_PORT"'
```
You can change the port number by setting the `EXPORTER_TCP_PORT` environment variable.

## Workflow 
### Build
``` bash
$ git clone https://github.com/kkh913/docker-jitsi-meet.git
$ cd docker-jitsi-meet
$ chmod +x base/rootfs/*.sh
$ FORCE_REBUILD=1 make
$ docker images
```

### (Optional) Push 
Your quay.io ID is needed for the followings. 
```
docker login quay.io 
docker tag jitsi/jvb:latest quay.io/<your id>/<your project name>:<version> 
docker push quay.io/<your id>/<your project name>:<version>
```

### Pull test 
For user permissions, because this image works on non-root mode. 
``` bash
# if your system don't have uid 1001, 
useradd -u 1001 -r -s /bin/false user
cp env.example .env 
./gen-passwords.sh 
mkdir -p ~/.jitsi-meet-cfg/{prosody/config,prosody/prosody-plugins-custom,jvb}
chown -R 1001:0 ~/.jitsi-meet-cfg
```
Add `EXPORTER_TCP_PORT=9888` in .env 
```
EXPORTER_TCP_PORT=9888
```

Edit docker-compose.yaml 
``` yaml
...
# Video bridge
jvb:
  image: quay.io/<your id>/<your project name>:<version>
  restart: ${RESTART_POLICY}
  ports:
    - '${JVB_PORT}:${JVB_PORT}/udp'
    - '${JVB_TCP_MAPPED_PORT}:${JVB_TCP_PORT}'
    - '${EXPORTER_TCP_PORT}:${EXPORTER_TCP_PORT}/tcp'
  volumes:
    ...
  environment:
      - EXPORTER_TCP_PORT
      ...
```


I strongly recommend you comment out all services except prosody and jvb for test. 
``` bash
docker-compose up -d 
curl localhost:9888/metrics 
```
