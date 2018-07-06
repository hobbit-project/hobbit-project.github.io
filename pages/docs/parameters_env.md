---
title: Env. Variables
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: parameters_env.html
folder: docs
---

Environment variables for platform components can be set in `docker-compose.yml`.

| Component | Parameter | Value | Meaning |
|--|--|--|--|
| platform-controller | `DOCKER_HOST` | Locator | The location of the Docker daemon the platform will use for creating, stopping and observing containers. |
| platform-controller | `HOBBIT_REDIS_HOST` | Host name | Host name of Redis. |
| platform-controller | `GITLAB_USER` | Gitlab user name | The user name used to access the [central repo of the platform](http://git.project-hobbit.eu). If it is not available, the local platform can not access benchmarks or systems from the central repository. |
| platform-controller | `GITLAB_EMAIL` | Gitlab user mail address | The users mail address used in the central repository. |
| platform-controller | `GITLAB_TOKEN` | Gitlab user security token | The users security token used to access the central repository. |
| platform-controller | `LOGGING_GELF_ADDRESS_KEY`| URL | Setting a URL enables the logging via [gelf logging driver](https://docs.docker.com/engine/admin/logging/overview/) for all containers created by the platform. |
| platform-controller | `DEPLOY_ENV` | `production`, `testing` or `develop` | Type of deployment. A `production` deployment (default) will remove all containers created by the platform. When set to `testing` the platform will remove only containers that exit with an exit code `0`. Components with a different exit code are not removed and can be inspected by the user later on. When set to `develop` no containers will be removed by the platform. |
| platform-controller | `DOCKER_AUTOPULL` | `0` or `1` | If this flag is set to `1` the platform will automatically pull the latest image of a container from the central repository. For local development, it may be necessary to turn this off by setting it to `0`. (default `1`) |
| platform-controller | `CONTAINER_PARENT_CHECK` | `0` or `1` | If this flag is set to `1` the platform will only accept container creation requests that are comming from known containers. For local development, it may be necessary to turn this off by setting it to `0`. (default `1`) |
| gui | `KEYCLOAK_AUTH_URL` | URL | The URL with which the Keycloak user management can be accessed from the users browser. |
| gui | `KEYCLOAK_DIRECT_URL` | URL | The URL with which the Keycloak user management can be directly accessed by the gui component. |
| gui | `CHECK_REALM_URL` | `true` or `false`| With this flag, the realm check can be turned off. It has to be set to `false` if the `KEYCLOAK_AUTH_URL` and `KEYCLOAK_DIRECT_URL` are different. (default `true`) |
| storage-service| `SPARQL_ENDPOINT_URL` | URL | The URL of the SPARQL endpoint used to store data. |
| storage-service| `SPARQL_ENDPOINT_USERNAME` | User name | The user name used to authenticate the service at the SPARQL endpoint. |
| storage-service| `SPARQL_ENDPOINT_PASSWORD` | Password | The password used to authenticate the service at the SPARQL endpoint. |
| platform-controller, gui, analysis, storage-service | `HOBBIT_RABBIT_HOST` | Host name | Host name of the RabbitMQ broker used for the communication with other components. |
