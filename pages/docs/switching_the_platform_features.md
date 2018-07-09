---
title: Switching the Platform Features On/Off
keywords: platform HOBBIT
sidebar: main_sidebar
toc: false
permalink: switching_the_platform_features.html
folder: docs
---

The following features of the HOBBIT platform can be turned on and off:
1. ELK stack logging
1. Encryption for communication between the components
1. Resource monitoring

For the complete platform deployment see [Platform Deployment on Single Machine](/platform_deployment_single_machine) or [Platform Deployment in Cluster](/platform_deployment_cluster.html).

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