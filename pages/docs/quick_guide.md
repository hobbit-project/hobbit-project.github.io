---
title: Quick Guide
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: quick_guide.html
folder: docs
---

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

1. Start the platform:
```
docker-compose up -d 
```

1. Initialize the Virtuoso storage:
```
./run-storage-init.sh
```

That's it!

After the platform startup, the following interfaces will be available for you:
* [localhost:8080](http://localhost:8080/)
(GUI, default credentials are: `challenge-organiser`:`hobbit`,
`testuser`:`hobbit`, `system-provider`:`hobbit` and `guest`:`hobbit`)
* [localhost:5601](http://localhost:5601/)
(Kibana)
* [localhost:8181](http://localhost:8181/)
(Keycloak, admin credentials are: `admin`:`H16obbit`)
* [localhost:8081](http://localhost:8081/)
(RabbitMQ)
* [localhost:8890](http://localhost:8890/)
(Virtuoso)

## Optional Steps

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

### Elasticsearch stack

For Elasticsearch stack you need to configure `vm.max_map_count`:
```
$ sudo vim /etc/sysctl.conf
# add line:
vm.max_map_count=262144
$ sudo sysctl -p
```

Start ELK stack before starting HOBBIT platform:
```
docker-compose -f docker-compose-elk.yml up -d
```
