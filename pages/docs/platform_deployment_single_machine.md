---
title: From `git clone` to Running Platform
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: platform_deployment_single_machine.html
folder: docs
---

For the quick start check out [From `git clone` to Running Platform](/tutorial_git_deploy.html) tutorial.

## Preparing

These steps have to be done only once before starting the platform the first time.

1. Clone this repository: 
    `git clone https://github.com/hobbit-project/platform && cd platform`
1. Create new hobbit networks:
    `make create-networks`
1. (optional) Build platform components by running:
    `make build`
1. Build and pull required docker containers by running:
    `docker-compose pull && docker-compose -f docker-compose-elk.yml pull`
1. Configure Virtuoso
   1. Change passwords (optional)
   1. Run initialization script (required)
      `make setup-virtuoso`
1. Configure Keycloak
1. Export your personal Gitlab token to env variables

### Configure Virtuoso

#### Change passwords (optional)
The Virtuoso store has two users - the Virtuoso super user `dba` and a second user that is used by the platform called `HobbitPlatform`. By default, both have the word `Password` as password. If you want to use different passwords, you can change them during the initialization of the Virtuoso. Note, that while you can run the initialization script only once, you still can change the Virtuoso user credentials using the Virtuoso GUI, later on.

* Generate two passwords for the Virtuoso super user `dba` and a second user that is used by the platform called `HobbitPlatform`
* Open `config/db/storage-init.sh` and put the passwords into the following two lines
```bash
# Setup the HOBBIT Platform user
/opt/virtuoso/bin/isql 1111 dba dba exec="DB.DBA.USER_CREATE ('HobbitPlatform', 'Password'); GRANT SPARQL_UPDATE TO "HobbitPlatform";"
...
# Finally, change the 'dba' password
/opt/virtuoso/bin/isql 1111 dba dba exec="user_set_password ('dba', 'Password');"
```

* Open `docker-compose.yml` in the platform directory and add the password for the `HobbitPlatform` user at
```yml
  # Storage service
  storage-service:
    ...
    environment:
      ...
      - SPARQL_ENDPOINT_USERNAME=HobbitPlatform
      - SPARQL_ENDPOINT_PASSWORD=Password
```

#### Run initialization script (required)

```bash
make setup-virtuoso
```

### Configure Keycloak

To be able to use the graphical user interface the Keycloak user management is needed. Since a user has to communicate with the Keycloak instance, the GUI needs to know the *public* address of the Keycloak instance, i.e., the address that Keycloak has when connecting to it using a browser.
* Determine the public address and put it in the `docker-compose.yml` file into the `KEYCLOAK_AUTH_URL` line of the GUI:
```yml
  # HOBBIT GUI
  gui:
    ...
    environment:
      - KEYCLOAK_AUTH_URL=http://<myipaddress>:8181/auth
```

* If you are accessing it locally on the machine the Docker is running, use localhost (or docker.local if you are on MacOS).
```yml
  # HOBBIT GUI
  gui:
    ...
    environment:
      - KEYCLOAK_AUTH_URL=http://localhost:8181/auth
```

Give write access to the keycloak database by performing
```bash
 chmod 777 config/keycloak
 chmod 666 config/keycloak/*
```
in the platforms project directory.

If the address of the GUI will be *different* from `http://<myipaddress>:8080` (e.g., because of the reason explained above) you have to configure this address in Keycloak
* Start Keycloak by running
```bash
docker-compose up -d keycloak
```
* Open the address of Keycloak in the browser and click on `Administration Console`. Login using the username `admin` and the password `H16obbit`.
* Make sure that the realm `Hobbit` is selected in the left upper corner below the Keycloak logo
* Click on `Clients` in the left menu and click on the `Hobbit-GUI` client.
* Add the address of the GUI to the list `Valid Redirect URIs` (with a trailing star, e.g., `http://192.168.99.100:8080/*` or `https://platform.example.com/*` in case of public deployment) as well as to the list `Web Origins` and click on `save` at the bottom of the page

#### Firewall adjustments (Linux)

The serverbackend of the hobbitgui container needs access to keycloak via the external IP address. Therefore you may need to adapt the firewall rules. First find the IP range of the hobbit network:
```bash
docker network inspect hobbit | grep Gateway
```

Assuming you get something like "Gateway": "172.19.0.1" you have to find the matching network device:
```bash
ip addr | grep -B 2 172.19.0
```

If you get something like
```bash
6: br-5c9d73b080ad: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:22:50:d4:8c brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/16 scope global br-5c9d73b080ad
```
the network device name is `br-5c9d73b080ad`. If you have iptables as firewall use:
```bash
iptables -A INPUT -i br-5c9d73b080ad -j ACCEPT
```

If you have firewalld (fedora/centos7) use:
```bash
firewall-cmd --permanent --zone=trusted --change-interface=br-5c9d73b080ad
firewall-cmd --reload
```

#### Details of the user management

To manage users, groups and/or roles:
* login to the Keycloak Administration Console (e.g., http://localhost:8181/auth/admin user: admin, default password: 'H16obbit')
* select the realm `Hobbit` (under KEYCLOAK logo, you should be able to access Master and Hobbit realm)

For new users do not forget to check/assign the role mappings. 
To assign role mappings go to Manage -> Users (sidebar on the left), then click "View all users".
Click "Edit" for any of the users and select "Role Mappings" tab. 
You can assign the roles on this screen.
The Hobbit-gui application uses following roles:
* `system-provider`, i.e., registered users
* `guest`
* `challenge-organiser` are registered users with the right to create challenges

The preconfigured Keycloak image has the following users (default password for all these users is `hobbit`):
* user `testuser` with the roles `system-provider`, `guest`
* user `system-provider` has role `system-provider`, `guest`
* user `guest` has role `guest`
* user `challenge-organiser` has role `challenge-organiser`

The admin used for the Keycloak configuration is not listed in the users of the `Hobbit` realm. You can change its password or add additional administrative users by choosing the `master` realm in the upper left corner.

### Using Benchmarks from the HOBBIT Git

For getting access to your Systems or Benchmarks that have been uploaded to the HOBBIT Git (http://git.project-hobbit.eu) the following steps are needed. Note that this is *the only way to introduce systems or benchmarks to the platform at the moment*.

Benchmarks should already be accessible if the project containing their meta data file is public. For accessing a public system, you need to define a user in your local Keycloak that has exactly the same user name as the user that owns the project in which the system meta data file is located. More information about user management can be found in the section [Details of the user management](https://github.com/hobbit-project/platform#details-of-the-user-management).

If a benchmark or system project is not public, you need to add your user name, your mail address and an access token of your user to the platforms compose file.
* Register and login to the [Hobbit git](http://git.project-hobbit.eu), open the `profile settings`, click on `Access Token` and create a personal access token for using the API. Write down your token, username and email
* Export username, email and token to the environment:
```bash
export GITLAB_USER=iermilov
export GITLAB_EMAIL=earthquakesan@gmail.com
export GITLAB_TOKEN=1234567890
```

When you execute the docker-compose commands these variables should be available in your terminal. You can quickly check it with (should show your username):
```bash
echo $GITLAB_USER
```

## Configure Elasticsearch

For Elasticsearch stack you need to configure `vm.max_map_count`:
```
$ sudo vim /etc/sysctl.conf
# add line:
vm.max_map_count=262144
$ sudo sysctl -p
```

## Running

You can run local version as follows: 
```bash
docker-compose -f docker-compose-elk.yml up -d 
docker-compose up -d
```

If you want to build and run the latest (e.g. dev) version of the platform you can do:
```
make build
make build-dev-images
docker-compose -f docker-compose-elk.yml up -d 
docker-compose -f docker-compose-dev.yml up -d
```


Available services
* http://localhost:8080/ Graphical user interface of the platform
* http://localhost:8081/ RabbitMQ GUI
* http://localhost:8181/ Keycloak GUI
* http://localhost:8890/ Virtuoso GUI (including a SPARQL endpoint
* http://localhost:5601/ Kibana GUI
* `5672` RabbitMQ communication port for adding additional components to the platform, e.g., for testing scenarios

## Shutting down the platform

To shut down the platform you can simply do:
```
docker-compose down
```

To stop and remove ALL containers on your host (other applications will be affected) you can do (use with caution):
```
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q)
```

## Troubleshooting

If you encounter problems setting up the platform, please have a look at our [FAQ](/faq.html).

### Regular cleaning of dangling Docker images
Depending on the way the platform is used, it can download many Docker images over time. Note that Docker itself won't delete but keep all versions of these images. To save disk space, a regular cleaning of dangling images is helpful.

#### Local machine

If the platform is deployed on a local machine, executing `docker rmi $(docker images -q -f "dangling=true")` manually from time to time is already sufficient.

#### Server

If the HOBBIT platform is deployed on a server a regular clean up of docker images is necessary. To perform clean up you need to install a cronjob as follows:
```
0 3 * * * docker rmi $(docker images -q -f "dangling=true") > /root/`date +\%Y\%m\%d\%H\%M\%S`-docker-rmi-dangling-cron.log
```

This will remove all dangling images at 3:00 A.M. (at night) every day.

To install crontab on several server you might want to use the following script:
```
#!/bin/bash

crontab -l > currentcron
echo "0 3 * * * docker rmi \$(docker images -q -f \"dangling=true\") > /root/\`date +\%Y\%m\%d\%H\%M\%S\`-docker-rmi-dangling-cron.log" >> currentcron
crontab currentcron
rm currentcron
```
The script can be shared across your cluster using e.g. NFS share and executed using following command:
```
for x in {1..7}; do ssh -K "myhobbitnodehostname$x" 'klsu -a /path/to/the/script.sh' ; done
```

# Related projects

There are some projects related to the platform

* [Core](https://github.com/hobbit-project/core) - Library containing core functionalities that ease the integration into the platform.
* [Evaluation storage](https://github.com/hobbit-project/evaluation-storage) - A default implementation of a benchmark component.
* [Platform](https://github.com/hobbit-project/platform) & The HOBBIT platform and a wiki containing tutorials.
* [Ontology](https://github.com/hobbit-project/ontology) & The HOBBIT ontology used to store data and described in D2.2.1 of the HOBBIT project.