---
title: Local Platform Deployment for Testing and Development
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: platform_deployment_single_machine.html
folder: docs
---

The platform can be started pretty fast by cloning the repository and executing
```sh
make build
```
This will compile the project and execute all necessary default configuration steps described below. After that, `make start` will start the platform.

For Elasticsearch stack you need to configure `vm.max_map_count`:
```
$ sudo vim /etc/sysctl.conf
# add line:
vm.max_map_count=262144
$ sudo sysctl -p
```

Start the platform:
```
make start
```

The following interfaces will be available for you:
* localhost:8080 (GUI) - default credential below (e.g. challenge-organiser/hobbit)
* localhost:5601 (Kibana)
* localhost:8181 (Keycloak) - admin credentials: admin/H160bbit
* localhost:8081 (RabbitMQ)
* localhost:8890 (Virtuoso)
