# hub.docker.com/r/tiredofit/openldap

[![Build Status](https://img.shields.io/docker/build/tiredofit/openldap.svg)](https://hub.docker.com/r/tiredofit/openldap)
[![Docker Pulls](https://img.shields.io/docker/pulls/tiredofit/openldap.svg)](https://hub.docker.com/r/tiredofit/openldap)
[![Docker Stars](https://img.shields.io/docker/stars/tiredofit/openldap.svg)](https://hub.docker.com/r/tiredofit/openldap)
[![Docker Layers](https://images.microbadger.com/badges/image/tiredofit/openldap.svg)](https://microbadger.com/images/tiredofit/openldap)

# Introduction

This as a Dockerfile to build a [OpenLDAP](https://openldap.org) server for maintaining a directory.
Upon starting this image it will give you a ready to run server with many configurable options.

* Tracks latest release
* Compiles from source
* Multiple backends (bdb, hdb, mdb, sql
* All overlays compiled
* Supports TLS encryption
* Supports Replication
* Optional Web Server included to take advantage of Let's Encrypt certificates
* Scheduled Backups of Data
* Ability to choose NIS or rfc2307bis Schema
* Zabbix Monitoring templates included

* This Container uses a [customized Alpine Linux base](https://hub.docker.com/r/tiredofit/alpine) which includes [s6 overlay](https://github.com/just-containers/s6-overlay) enabled for PID 1 Init capabilities, [zabbix-agent](https://zabbix.org) based on 3.4 Packages for individual container monitoring, Cron also installed along with other tools (bash,curl, less, logrotate, mariadb-client, nano, vim) for easier management. It also supports sending to external SMTP servers..


[Changelog](CHANGELOG.md)

# Authors

- [Dave Conroy](dave@tiredofit.ca)

# Table of Contents

- [Introduction](#introduction)
	- [Changelog](CHANGELOG.md)
- [Prerequisites](#prerequisites)
- [Dependencies](#dependendcies)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
	- [Data Volumes](#data-volumes)
	- [Database](#database)
	- [Environment Variables](#environmentvariables)   
	- [Networking](#networking)
- [Maintenance](#maintenance)
	- [Shell Access](#shell-access)
- [References](#references)


# Prerequisites

This image has the capability to take advantage of getting TLS certificates autogenerated via the 
[jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy) and the [Let's Encrypt Proxy Companion @ 
https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion). 
However, it will run just fine on it's own without it.

# Dependencies

None.

# Installation

Automated builds of the image are available on [Registry](https://registry.selfdesign.org/docker/openldap) and is the recommended method of installation.

```bash
docker pull registry.selfdesign.org/docker/openldap
```

# Quick Start

* The quickest way to get started is using [docker-compose](https://docs.docker.com/compose/). See the examples folder for a working [docker-compose.yml](examples/docker-compose.yml) that can be modified for development or production use.

* Set various [environment variables](#environment-variables) to understand the capabilities of this image.
* Map [persistent storage](#data-volumes) for access to configuration and data files for backup.
* Map [Network Ports](#networking) to allow external access.

Start openldap using:

```bash
docker-compose up
```
__NOTE__: Please allow up to 2 minutes for the application to start for the first time if you are generating TLS certificates.

## Data-Volumes


The following directories are used for configuration and can be mapped for persistent storage.

| Directory | Description |
|-----------|-------------|
| `/var/lib/openldap` | Data Directory |
| `/etc/openldap/slapd.d` | Configuration Directory |
| `/assets/custom-scripts/` | If you'd like to execute a script during the initialization process drop it here (Useful for using this image as a base)
| `/assets/slapd/certs/` | Drop TLS Certificates here |
| `/data/backup` | Backup Directory |   
| `/www/html` | If you want to put a landing page if using Nginx for LetsEncrypt SSL Place it here |      

## Environment Variables
Along with the Environment Variables from the [Base image](https://hub.docker.com/r/tiredofit/alpine), below is the complete list of 
available options that can be used to customize your installation.

Required and used for new ldap server only:

| Variable | Description |
|-----------|-------------|
| `DOMAIN` | LDAP domain. Default  `example.org` |
| `BASE_DN` | LDAP base DN. If empty automatically set from `DOMAIN` value. Default  (empty) |
| `ADMIN_PASS` | Ldap Admin password. Default  `admin` |
| `CONFIG_PASS` | Ldap Config password. Default  `config` |
| `ENABLE_READONLY_USER` | Add a read only user. Default`false` |
| `READONLY_USER_USER` | Read only user username. Default `readonly |
| `READONLY_USER_PASS` | Read only user password. Default `readonly` |
| `SCHEMA_TYPE` | Use `nis` or `rfc2307bis` core schema. Default `nis` |


| Variable | Description |
|-----------|-------------|
| `BACKEND` | Ldap backend. `bdb` `hdb` `mdb` and others. Default `mdb` |
| `LOG_LEVEL` | Set LDAP Log Level - Default `256`
    

Backup Options:

| Variable | Description |
|-----------|-------------|
| `BACKUP_CONFIG_CRON_PERIOD` | Cron expression to schedule OpenLDAP config backup. Defaults `0 4 * * *` Every day at 4am. |
| `BACKUP_DATA_CRON_PERIOD` | Cron expression to schedule OpenLDAP data backup. Defaults `0 4 * * *`  Every day at 4am. |
| `BACKUP_TTL ` | Automatically cleanup backup after how many days. Default `15` |

TLS options:

| Variable | Description |
|-----------|-------------|
| `ENABLE_TLS` | Add TLS capabilities. Can't be removed once set to `true`. Defaults `true` |
| `TLS_CRT_FILENAME` | Ldap ssl certificate filename. Default `cert.pem` |
| `TLS_KEY_FILENAME` | Ldap ssl certificate private key filename. Default `key.pem` |
| `TLS_CA_CRT_FILENAME` | Ldap ssl CA certificate filename. Default `ca.pem` |
| `TLS_ENFORCE` | Enforce TLS. Can't be disabled once set to `true`. Defaults `false` |
| `TLS_CIPHER_SUITE` | TLS cipher suite. Default `ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:RSA+AESGCM:RSA+AES:-DHE-DSS:-RSA:!aNULL:!MD5:!DSS:!SHA` |
| `TLS_VERIFY_CLIENT` | TLS verify client. Default  `try`

    Help: http://www.openldap.org/doc/admin24/tls.html

Replication options:

| Variable | Description |
|-----------|-------------|
| `ENABLE_REPLICATION` | Add replication capabilities. Multimaster only at present. Default `false`
| `REPLICATION_CONFIG_SYNCPROV` |  olcSyncRepl options used for the config database. Without rid and provider which are automatically added based on `REPLICATION_HOSTS`. Default  `binddn="cn=admin,cn=config" bindmethod=simple credentials=$LDAP_CONFIG_PASSWORD searchbase="cn=config" type=refreshAndPersist retry="60 +" timeout=1 starttls=critical` |
| `REPLICATION_DB_SYNCPROV` | olcSyncRepl options used for the database. Without rid and provider which are automatically added based on `REPLICATION_HOSTS`. Default  `binddn="cn=admin,$LDAP_BASE_DN" bindmethod=simple credentials=$LDAP_ADMIN_PASSWORD searchbase="$LDAP_BASE_DN" type=refreshAndPersist interval=00:00:00:10 retry="60 +" timeout=1 starttls=critical` |
| `REPLICATION_HOSTS`  | list of replication hosts seperated by a space, must contain the current container hostname set by --hostname on docker run command. If replicating all hosts must be set in the same order. Example - `ldap://ldap1.example.com ldap://ldap2.example.com ldap://ldap3.example.com`


 Other environment variables:

| Variable | Description |
|-----------|-------------|
| `ENABLE_NGINX` | If you want to use automatic LetsEncrypt certificates for your server, set this to `true`
| `REMOVE_CONFIG_AFTER_SETUP` | Delete config folder after setup. Default  `true` |
| `SSL_HELPER_PREFIX` | Ssl-helper environment variables prefix. Default  `ldap`, ssl-helper first search config from `SSL_HELPER_*` variables, before `SSL_HELPER_*` variables. |


## Networking

The following ports are exposed and available to public interfaces

| Port | Description |
|-----------|-------------|
| `80` | Nginx - For Automatic LetsEncrypt Certficates |
| `389` | Unecrypted LDAP |
| `636` | TLS Encrypted LDAP |

## Maintenance
#### Shell Access

For debugging and maintenance purposes you may want access the containers shell. 

```bash
docker exec -it openldap bash
```

# References

* https://openldap.org
