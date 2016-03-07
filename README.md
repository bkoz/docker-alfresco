docker-alfresco
===============

# Table of Contents

- [Introduction](#introduction)
- [Contributing](#contributing)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
  - [Datastore](#datastore)
  - [Database](#database)
  - [Options](#options)
- [Upgrading](#upgrading)
- [OpenShift](#openshift)
- [References](#references)


# Introduction
Dockerfile to build an Alfresco container image.


# Contributing
Here is how you can help:
- Send a Pull Request with your awesome new features and bug fixes
- Report [Issues](https://github.com/gui81/docker-alfresco/issues)


# Installation
Pull the image from the docker index.
```bash
docker pull gui81/alfresco:latest
```

or pull a particular version:
```bash
docker pull gui81/alfresco:5.0.d-1
```

Alternatively, you can build the image yourself:
```bash
git clone https://github.com/gui81/docker-alfresco.git
cd docker-alfresco
docker build --tag="$USER/alfresco" .
```


# Quick Start
Run the alfresco image with the name "alfresco".

```bash
docker run --name='alfresco' -it --rm -p 8080:8080 gui81/alfresco
```

**NOTE**: Please allow a few minutes for the application to start, especially if
populating the database for the first time.

Go to `http://localhost:8080` or point to the ip of your docker server.  On the
Mac, if you are running docker-machine, then you can go to the ip reported by:

```bash
docker-machine ip [name of Docker VM]
```

The default username and password are:
* username: **admin**
* password: **admin**

Alfresco should now be up and running.  The following is an example that would
mount the appropriate volume, connect to a remote PostgreSQL database, and use
an external LDAP server for authentication:
```bash
docker run --name='alfresco' -it --rm -p 445:445 -p 7070:7070 -p 8080:8080 \
    -v /host/alfresco_data:/alfresco/alf_data \
    -e 'CONTENT_STORE=/alfresco/alf_data' \
    -e 'LDAP_ENABLED=true' \
    -e 'LDAP_AUTH_USERNAMEFORMAT=uid=%s,cn=users,cn=accounts,dc=example,dc=com' \
    -e 'LDAP_URL=ldap://ipa.example.com:389' \
    -e 'LDAP_DEFAULT_ADMINS=admin' \
    -e 'LDAP_SECURITY_PRINCIPAL=uid=admin,cn=users,cn=accounts,dc=example,dc=com' \
    -e 'LDAP_SECURITY_CREDENTIALS=password' \
    -e 'LDAP_GROUP_SEARCHBASE=cn=groups,cn=accounts,dc=example,dc=com' \
    -e 'LDAP_USER_SEARCHBASE=cn=users,cn=accounts,dc=example,dc=com' \
    -e 'DB_KIND=postgresql' \
    -e 'DB_HOST=db_server.example.com' \
    -e 'DB_USERNAME=alfresco' \
    -e 'DB_PASSWORD=alfresco' \
    -e 'DB_NAME=alfresco' \
    gui81/alfresco
```

If you want to use this image in production, then please read on.


# Configuration
## Datastore
To persist data, you will want to make sure to specify and mount the
CONTENT_STORE, example:
* `/alfresco/alf_data`

Volumes can be mounted by passing the **'-v'** option to the docker run command.
The following is an example:
```bash
docker run --name alfresco -it --rm -v /host/alfresco_data:/alfresco/alf_data
```


## Database
If the `DB_HOST` environment variable is not set, or set to localhost, then the
image will use the internal PostgreSQL server.

PostgreSQL is the default, but MySQL/MariaDB is also supported.  If you are
using an existing database installation, then make sure to create the database
and a user:
```sql
CREATE ROLE alfresco WITH LOGIN PASSWORD 'alfresco';
CREATE DATABASE alfresco;
GRANT ALL PRIVILEGES ON DATABASE alfresco TO alfresco;
```


## Options
Below is the complete list of currently available parameters that can be set
using environment variables.
- **ALFRESCO_HOSTNAME**: hostname of the Alfresco server; default = `localhost`
- **CIFS_ENABLED**: whether or not to enable CIFS; default = `true`
- **CIFS_SERVER_NAME**: hostname of the CIFS server; default = `localhost`
- **CIFS_DOMAIN**: domain of the CIFS server; default = `WORKGROUP`
- **CONTENT_STORE**: where content is stored; default = `/alfresco/alf_data`
- **DB_HOST**: host of the database server; default = `localhost`
- **DB_KIND**: postgresql or mysql; default = `postgresql`
- **DB_NAME**: name of the database to connect to; default = `alfresco`
- **DB_PASSWORD**: password to use when connecting to the database; default = `admin`
- **DB_USERNAME**: username to use when connecting to the database; default = `alfresco`
- **FTP_PORT**: port of the database server; default = `5432`
- **LDAP_ENABLED**: whether or not to enable LDAP; default = `false`
- **LDAP_AUTH_USERNAMEFORMAT**: default = `uid=%s,cn=users,cn=accounts,dc=example,dc=com`
- **LDAP_URL**: URL of LDAP server; default = `ldap://ldap.example.com:389`
- **LDAP_DEFAULT_ADMINS**: comma separated list of admin names in ldap; default = `admin`
- **LDAP_SECURITY_PRINCIPAL**: default = `uid=admin,cn=users,cn=accounts,dc=example,dc=com`
- **LDAP_SECURITY_CREDENTIALS**: default = `password`
- **LDAP_GROUP_SEARCHBASE**: default = `cn=groups,cn=accounts,dc=example,dc=com`
- **LDAP_USER_SEARCHBASE**: default = `cn=users,cn=accounts,dc=example,dc=com`
- **MAIL_HOST**: hostname or IP where email should be sent; default = `localhost`
- **MAIL_PORT**: default = `25`
- **MAIL_USERNAME**: username to connect to the smtp server
- **MAIL_PASSWORD**: password to connect to the smtp server
- **MAIL_FROM_DEFAULT**: what is in the from field; default = `alfresco@alfresco.org`
- **MAIL_PROTOCOL**: smtp or smtps; default = `smtp`
- **MAIL_SMTP_AUTH**: is authentication required or not; default = `false`
- **MAIL_SMTP_STARTTLS_ENABLE**: use starttls or not; default = `false`
- **MAIL_SMTPS_AUTH**: is authentication required or not; default = `false`
- **MAIL_SMTPS_STARTTLS_ENABLE**: use starttls or not; default = `false`
- **NFS_ENABLED**: whether or not to enable NFS; default = `true`
- **SHARE_HOSTNAME**: hostname of the share server; default = `localhost`


# Upgrading
TODO: I might be able to add some options that aid in upgrading.  For now though,
backup, backup, backup, and then follow this guide:
* http://docs.alfresco.com/community/concepts/ch-upgrade.html

# OpenShift

Running the alfresco container on OpenShift requires the anyuid security context constraint (SCC).

As the cluster admin, run the following to grant the anyuid scc to authenticated users.
```
$ oadm policy add-scc-to-group anyuid system:authenticated
```
OR

```
oadm policy add-scc-to-user anyuid -z default -n PROJECT
```

Now login as an authenticated user.

Create the database pod and set a few important parameters.
```
$ oc new-app --template=mysql-persistent -p MYSQL_USER=alfresco,MYSQL_PASSWORD=admin,MYSQL_DATABASE=alfresco
```

Create the Alfresco pod from gui81's docker hub.
```
$ oc new-app docker.io/gui81/alfresco -e DB_KIND=mysql,DB_HOST=mysql,CONTENT_STORE=/content
```

A different option is to build the alfresco image from rsippl's Dockerfile on github.
```
$ oc new-app https://github.com/rsippl/docker-alfresco.git -e DB_KIND=mysql,DB_HOST=mysql,CONTENT_STORE=/content
```

Assuming that your OpenShift cluster has allocated persistent volumes, create a persistent volume claim for /content and add it to the docker-alfresco deployment config. This is most easily accomplished using the
OpenShift web console. Also create a route and change the port to 8080/tcp.

Example:

```
$ cat pvc.yaml << EOF 
apiVersion: "v1"
kind: "PersistentVolumeClaim"
metadata:
  name: "alfresco"
spec:
  accessModes:
    - "ReadWriteOnce"
  resources:
    requests:
      storage: "1Gi"
EOF
```
```
$ oc create -f pvc.yaml
```


# References
* http://www.alfresco.com/community
* http://docs.alfresco.com/community/concepts/welcome-infocenter_community.html
