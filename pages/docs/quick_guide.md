---
title: Quick Guide
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: quick_guide.html
folder: docs
---

{% include note.html content="We have a [video tutorial](https://www.youtube.com/watch?v=ktAtwU55M6s) covering the quick local deployment." %}

1. Make sure you've got all [requirements](/requirements.html) installed.

1. Clone the repository:
```
git clone https://github.com/hobbit-project/platform && cd platform
```

1. Initialize Docker Swarm and create the necessary docker networks (the used subnets are `172.16.100.0/24`, `172.16.101.0/24` and `172.16.102.0/24`):
```
docker swarm init
make create-networks
```

1. Decide whether user management should be used. Have a look at the section [User Management](user-management).

1. Configure how the platform should access benchmarks and systems. At least one of the two options has to be enabled.
  * Use the [HOBBIT gitlab instance](#hobbit-gitlab-credentials)
  * Use [local metadata files](enable-local-metadata-files)

1. The [ELK stack](#elk-stack-for-log-access) can be added as central storage of log messages. This is only suggested for larger deployments.

1. Start the platform:
```
docker-compose up -d 
```

1. Initialize the Virtuoso storage. This needs to be done only when the platform is started for the first time. Note that the platform has to be running while the following command is executed:
```
./run-storage-init.sh
```

That's it!

After the platform startup, the following interfaces will be available for you:
* [localhost:8080](http://localhost:8080/)
(GUI; if Keycloak is used the default credentials are: `challenge-organiser`:`hobbit`, `system-provider`:`hobbit` and `guest`:`hobbit`)
* [localhost:8081](http://localhost:8081/)
(RabbitMQ)
* [localhost:8890](http://localhost:8890/)
(Virtuoso, default credentials are: `HobbitPlatform`:`Password` and `dba`:`Password`)
* [localhost:8181](http://localhost:8181/)
(Keycloak, if deployed; admin credentials are: `admin`:`H16obbit`)
* [localhost:5601](http://localhost:5601/)
(Kibana, available if deployed together with the ELK stack)

Now, when you've got a running platform,
you can [benchmark a system](/benchmarking.html).

## Optional Steps

### User Management

The platform can be started with a local user management. This enforces that users have to authenticate themselfs before starting any experiment. Unauthenticated users (i.e., guests) can only see experiment results. We use Keycloak for the user management. The platform can be run with Keycloak. In that case, it must be configured as described below.

For a simple, local setup, we suggest to run the platform [without Keycloak](#remove-keycloak).

#### Configure Keycloak

Set the permissions for Keycloak's database with
```
make set-keycloak-permissions
```

#### Remove Keycloak

Add the following environment variable for `gui` (`hobbitproject/hobbit-gui`) service in `docker-compose.yml`:
```diff
   gui:
     # ...
     environment:
+      - USE_UI_AUTH=false
```

Remove `keycloak` service section from `docker-compose.yml`.

Note that anyone with access to HOBBIT UI would be able to run experiments with this setup.

### HOBBIT GitLab credentials

HOBBIT GitLab credentials are needed for accessing private benchmarks and systems
uploaded to [git.project-hobbit.eu](https://git.project-hobbit.eu).
Register there and write down your username and email.
Go to User Settings (click on your user picture in the upper right corner --> Settings) -> Access Token and generate a personal access token for your HOBBIT instance. Write down your token.

You can either set your credentials in `docker-compose.yml`
or export them in your environment (perhaps, using [direnv](https://direnv.net/)):
```
export GITLAB_USER=max.power
export GITLAB_EMAIL=max.power@project-hobbit.eu
export GITLAB_TOKEN=1234567890
```

**Note** that if you don't have an account, you can still access public benchmarks and systems. For that, you should either remove the following lines from your `docker-compose.yml` file or comment them out using the `#` character.
```yaml
services:
  platform-controller:
    ...
    environment:
      ...
      GITLAB_USER: "${GITLAB_USER}"
      GITLAB_EMAIL: "${GITLAB_EMAIL}"
      GITLAB_TOKEN: "${GITLAB_TOKEN}"
```

### Enable local metadata files

Instead of using Gitlab, the platform can be enabled to gather metadata about systems and benchmarks from a local directory. For that, a local directory should be mounted into the platform controller and a variable should be set to inform the controller where it can find the files. In the following example, both is added to the `docker-compose.yml` file for a local directory named `meta`.
```yaml
  platform-controller:
    ...
    environment:
      ...
      LOCAL_METADATA_DIRECTORY: "/usr/src/app/metadata"
    volumes:
      ...
      - ./meta:/usr/src/app/metadata
```

After that, the platform will regularly load the metadata of all `*.ttl` files in this directory.

### ELK stack for log access

For Elasticsearch stack you need to configure `vm.max_map_count`:
```
$ sudo vim /etc/sysctl.conf
# add line:
vm.max_map_count=262144
$ sudo sysctl -p
```

If you have less than 8GB RAM, you need to tweak
the following settings in `config/elk/jvm.options`:
```
-Xms8g
-Xmx8g
```

Start ELK stack before starting HOBBIT platform:
```
docker-compose -f docker-compose-elk.yml up -d
```
