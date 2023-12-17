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

1. Set the permissions for the keycloak's database:
```
make set-keycloak-permissions
```

1. At this point, the optional configurations described below can be applied if necessary. 
  * [Keycloak dependency can be removed](#deployment-without-keycloak).
  * The access to benchmarks and systems via [HOBBIT Gitlab credentials](#hobbit-gitlab-credentials) or [local files](enable-local-metadata-files) can be configured.
  * The [ELK stack](#elk-stack-for-log-access) could be configured and started.

1. Start the platform:
```
docker-compose up -d 
```

1. Initialize the Virtuoso storage (this needs to be done only when the platform is started for the first time):
```
./run-storage-init.sh
```

That's it!

After the platform startup, the following interfaces will be available for you:
* [localhost:8080](http://localhost:8080/)
(GUI, default credentials are: `challenge-organiser`:`hobbit`, `system-provider`:`hobbit` and `guest`:`hobbit`)
* [localhost:8081](http://localhost:8081/)
(RabbitMQ)
* [localhost:8890](http://localhost:8890/)
(Virtuoso, default credentials are: `HobbitPlatform`:`Password` and `dba`:`Password`)
* [localhost:8181](http://localhost:8181/)
(Keycloak, admin credentials are: `admin`:`H16obbit`)
* [localhost:5601](http://localhost:5601/)
(Kibana, available if deployed together with the ELK stack)

Now, when you've got a running platform,
you can [benchmark a system](/benchmarking.html).

## Optional Steps

### Deployment without Keycloak

Add the following environment variable for `gui` (`hobbitproject/hobbit-gui`) service in `docker-compose.yml`:
```diff
   gui:
     # ...
     environment:
+      - USE_UI_AUTH=false
```

Add the following mount for `gui` service:
```diff

   gui:
     # ...
     volumes:
+      - ./config/jetty/web-without-ui-auth.xml:/var/lib/jetty/webapps/ROOT/WEB-INF/web.xml
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
