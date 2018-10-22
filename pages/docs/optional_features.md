---
title: Optional platform features
keywords: platform HOBBIT
sidebar: main_sidebar
toc: true
permalink: optional_features
folder: docs
---

For the complete platform deployment see [Platform Deployment on Single Machine](/platform_deployment_single_machine) or [Platform Deployment in Cluster](/platform_deployment_cluster).

## Elasticsearch Logstack Kibana

ELK stack is enabled by default. To disable ELK stack you will need to do the following:
1. Remove LOGGING_GELF_ADDRESS env variable from platform-controller service in docker-compose.yml
1. Remove ELASTICSEARCH_HOST and ELASTICSEARCH_HTTP_PORT env variables from gui service in docker-compose.yml

If you do not use ELK stack, you will only need to run docker-compose.yml. The start up of docker-compose-elk.yml is not necessary.

## Encryption

Encryption is disabled by default. To enable encryption add the following env variables to the docker-compose.yml:
```
version: '3.3'
services:
  platform-controller:
    environment:
      AES_PASSWORD: "$AES_PASSWORD"
      AES_SALT: "$AES_SALT"

  gui:
    environment:
      AES_PASSWORD: "$AES_PASSWORD"
      AES_SALT: "$AES_SALT"

  storage-service:
    environment:
      AES_PASSWORD: "$AES_PASSWORD"
      AES_SALT: "$AES_SALT"

  analysis:
    environment:
      AES_PASSWORD: "$AES_PASSWORD"
      AES_SALT: "$AES_SALT"
```

After that you need to export $AES_PASSWORD and $AES_SALT to your shell:
```
$ export AES_PASSWORD=mysecurepassword
$ export AES_SALT=mysalt
```

You can start docker-compose normally, the encryption will be enabled.

## Resource Monitoring

For the resource monitoring you will need to start additional services. In the hobbit platform root folder execute:
```
cd platform-controller && make node-exporter cAdvisor prometheus
```

Set up the environment variables in the docker-compose.yml:
```
version: "3.3"

services:
  platform-controller:
    environment:
      - PROMETHEUS_HOST=prometheus
      - PROMETHEUS_PORT=9090
```

Now you can start docker-compose.yml normally and you will have resource monitoring enabled.

## Rabbitmq Clustering

You need to edit docker-compose.yml (or docker-compose-dev.yml) and comment out or delete rabbitmq related section:

```
# message bus
#rabbit:
#  image: rabbitmq:management
#  networks:
#    - hobbit
#    - hobbit-core
#  ports:
#    - "8081:15672"
#    # Forwarding the port for testing
#    - "5672:5672"
```

Start 3 nodes rabbitmq cluster:
```
make start-rabbitmq-cluster
```

You can navigate to http://localhost:15672/#/ and see that rabbitmq got 3 nodes available.
Now you can start docker-compose.yml normally, the HOBBIT platform will be using cluster of rabbitmq nodes.

For production use, you need to edit `rabbitmq-cluster/docker-compose.yml` and `rabbitmq-cluster/haproxy.cfg` to specify rabbitmq services for each of your worker nodes. Use placement constraints to allocate the services to different nodes, e.g.:
```
  rabbitmq1:
    deploy:
      placement:
        constraints:
          - "node.labels.org.hobbit.name==worker1"
```