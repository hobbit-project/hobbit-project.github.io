---
title: Volumes
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: parameters_volumes
folder: docs
---

Volumes for platform components are configured in `docker-compose.yml`.

| Component | Volume | Meaning |
|--|--|--|
| platform-controller | `/var/run/docker.sock` | Should be mounted to the socket of the Docker daemon if `DOCKER_HOST` is set to `unix:///var/run/docker.sock`. |
| platform-controller | `/usr/src/app/config` | (Optional) A directory containing the [`config.yml`](https://github.com/hobbit-project/platform/wiki/Configuration-parameters#controller-config) file.
| keycloak | `/opt/jboss/keycloak/standalone/data/db` | Should be mounted to the directory containing the database of Keycloak. |
| virtuoso | `/opt/virtuoso-opensource/var/lib/virtuoso/db` | Should be mounted to the directory containing the database of Virtuoso. |
