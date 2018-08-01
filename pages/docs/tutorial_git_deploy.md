---
title: From `git clone` to Running Platform
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: tutorial_git_deploy.html
folder: docs
---

Make sure you've got all [requirements](/requirements.html) installed.

Clone the repository:
```
git clone https://github.com/hobbit-project/platform && cd platform
```

Pull the required HOBBIT platform and ELK docker images (around 7.5GB):
```
docker-compose pull
docker-compose -f docker-compose-elk.yml pull
```

Register at HOBBIT gitlab instance at `git.project-hobbit.eu`. Write down your username and email. On the HOBBIT gitlab instance go to User Settings (click on your userpic in the upper right corner --> Settings) -> Access Token and generate a personal access token for your HOBBIT instance. Write down your token. Export username, email and token to the environment:
```
export GITLAB_USER=iermilov
export GITLAB_EMAIL=earthquakesan@gmail.com
export GITLAB_TOKEN=1234567890
```

When you execute the docker-compose commands these variables should be available in your terminal. You can quickly check it with (should show your username):
```
echo $GITLAB_USER
```

Initialize Docker Swarm and create the necessary docker networks (the used subnets are 172.16.100.0/24, 172.16.101.0/24 and 172.16.102.0/24):
```
docker swarm init
make create-networks
```

For Elasticsearch stack you need to configure `vm.max_map_count`:
```
$ sudo vim /etc/sysctl.conf
# add line:
vm.max_map_count=262144
$ sudo sysctl -p
```

Start ELK stack:
```
docker-compose -f docker-compose-elk.yml up -d 
```

Start the platform:
```
docker-compose up -d 
```

Initialize the Virtuoso storage:
```
./run-storage-init.sh
```

After the platform startup, the following interfaces will be available for you:
* localhost:8080 (GUI) - 

Default credentials are: challenge-organiser/hobbit, testuser/hobbit, system-provider/hobbit and guest/hobbit

* localhost:5601 (Kibana)
* localhost:8181 (Keycloak) 

Admin credentials are: `admin`:`H16obbit`

* localhost:8081 (RabbitMQ)
* localhost:8890 (Virtuoso)