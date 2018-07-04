---
title: storage-init.sh
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: parameters_storage_init.html
folder: docs
---

The [`storage-init.sh`](https://github.com/hobbit-project/platform/blob/master/config/db/storage-init.sh) script is used to initialize the Virtuoso storage. The following lines can be adapted to set the passwords of the `HobbitPlatform` and the `dba` user.
```
# Setup the HOBBIT Platform user
/opt/virtuoso-opensource/bin/isql 1111 dba dba exec="DB.DBA.USER_CREATE ('HobbitPlatform', 'Password'); GRANT SPARQL_UPDATE TO "HobbitPlatform";"

...

# Finally, change the 'dba' password
/opt/virtuoso-opensource/bin/isql 1111 dba dba exec="user_set_password ('dba', 'Password');"
```
Please note that the script can be executed only once with a newly created Virtuoso database because the last command changes the password of the `dba` user which is used to execute all the commands in the script.
