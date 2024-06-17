# DNPM:DIP - Deployment/Operation Instructions


**WORK IN PROGRESS**



## Pre-requisites

* Docker / Docker Compose

### Certificates

* DFN Server Certificate issued for your DNPM:DIP node's FQDN
* Client Certificate issued by DNPM CA (see [here](https://ibmi-ut.atlassian.net/wiki/spaces/DAM/pages/2590714/Zertifikat-Verwaltung#Zertifikat-Verwaltung-BeantragungeinesClient-Zertifikats) for instructions to request one)


## System Overview

TODO Overview Diagrams


## Set-up

### Docker Compose

A template [`docker-compose.yml`](https://github.com/KohlbacherLab/dnpm-dip-deployment/blob/master/docker-compose.yml) is available.

Basic configuration occurs via environment variables, for most of which default values are pre-defined.
The only mandatory variables that MUST be provided are 

| Variable | Use/Meaning |
| -------- | ------- |
| `BASE_URL` | The base URL to your local DNPM:DIP node, normally the (reverse) proxy behind which the components run (see below) |
| `BACKEND_LOCAL_SITE` | Your local DNPM Site, in format `{Site-ID}:{Site-name}`, e.g. `UKT:Tübingen` (see [here](https://ibmi-ut.atlassian.net/wiki/spaces/DAM/pages/2613900/DNPM+DIP+-+Broker-Verbindungen) for the overview list of Site IDs and names) |

These variables must be provided via a file `.env` (see also [here](https://docs.docker.com/compose/environment-variables/set-environment-variables/)). Here's a sample:

```bash
BACKEND_LOCAL_SITE=UKx:SiteName
BASE_URL=https://dnpm.uni-x.de
```

If desired, the following variables CAN be set in `.env` to override the default values interpolated in the `docker-compose.yml`:

| Variable | Use/Meaning |
| -------- | ------- |
| `AUTHUP_SECRET`           | Secret (password) of the Authup Admin user |
| `MYSQL_ROOT_PASSWORD`     | Password of the MySQL DB used in Authup |
| `BASE_URL`                | The Base URL to the reverse proxy exposing the DNPM:DIP node's frontend and backend |
| `BACKEND_RD_RANDOM_DATA`  | Set to a positive integer to activate in-memory generation of RD random data (for test purposes) |
| `BACKEND_MTB_RANDOM_DATA` | Set to a positive integer to activate in-memory generation of MTB random data (for test purposes) |
| `BACKEND_CONNECTOR_TYPE`  | Set to one of { broker, peer2peer } to specify the desired connector type (see below) | 
| `BACKEND_AUTHUP_URL`      | Base URL under which the Backend can reach Authup |  


### Backend

Various functions/components of the backend are configured via external configuration files.
Templates for these are available [here](https://github.com/KohlbacherLab/dnpm-dip-deployment/tree/master/backend-config).
They must be placed in the directory bound to docker volume `/dnpm_config` in `docker-compose.yml`.


#### Play HTTP Server

The Play HTTP Server the backend application runs in is configured via file [`production.conf`](https://github.com/KohlbacherLab/dnpm-dip-deployment/blob/master/backend-config/production.conf). The template provides defaults for all required settings.
Only in case the backend won't be addressed via a reverse proxy forwarding to 'localhost' (see below) but directly by IP and/or hostname, these "allowed hosts" must be configured explicitly:

```bash
  ...
  hosts {
    allowed = ["localhost","your.host.name"]
  }
```
See also the [Play Framework Documentation](https://www.playframework.com/documentation/3.0.x/AllowedHostsFilter).


#### Persistence

Data persistence by the backend (currently) uses the file system. Bind the directory meant for this purpose to the backend service's docker volume `/dnpm_data`. 
Depending on the permission set on this directory, you might have to explicitly set the system user ID for the docker process running the backend (see `backend.user` in `docker-compose.yml`).


#### Logging

Logging is based on [SLF4J](https://slf4j.org/). The SLF4J implementation used by the Play Framework is [Logback](https://logback.qos.ch/), which is configured via file [`logback.xml`](https://github.com/KohlbacherLab/dnpm-dip-deployment/blob/master/backend-config/logback.xml).
The default settings in the template define a daily rotating log file stored in sub-folder `/logs` of the docker volume bound for persistence (see above).
You might consider removing/deactivating the logging [appender to STDOUT](https://github.com/KohlbacherLab/dnpm-dip-deployment/blob/master/backend-config/logback.xml#L30)


#### Application Config

The Backend application itself is configured via [`config.xml`](https://github.com/KohlbacherLab/dnpm-dip-deployment/blob/master/backend-config/config.xml). The main configuration item is the type of Connector used for communication with external peer DNPM:DIP nodes.
This file's template shows example configurations for both possible connector types. Simply delete the one _not_ applicable to your case.

##### Broker Connector

The connector for the hub/spoke network topology used in DNPM, via a central broker.
The primary parameter to adapt is the base URL to your local Broker Proxy in `<Broker baseURL="..."/>`.
The connector performs "peer discovery" by fetching the list of external peers from the central broker.
If desired, you can also override the request time-out (seconds).
Also, in case you prefer the nector to periodically update its "peer list", instead of just once upon start-up, set the period (minutes).

##### Peer-to-peer Connector

The connector based on a peer-to-peer network topology, which requires each external peer's Site ID, Name, and BaseURL to be configured in a dedicated element, as shown in the example template.


### NGINX Proxy

TODO










