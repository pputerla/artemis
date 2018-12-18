# ARTEMIS

Table of Contents
  * [General](#general)
  * [Features](#features)
  * [Architecture](#architecture)
  * [Getting Started](#getting-started)
  * [Min. technical requirements of testing server/VM](#min-technical-requirements-of-testing-servervm)
  * [How to install](#how-to-install)
  * [How to run](#how-to-run)
  * [Contributing](#contributing)
  * [Development Team and Contact](#development-team-and-contact)
  * [Versioning](#versioning)
  * [Authors and Contributors](#authors-and-contributors)
  * [License](#license)
  * [Acknowledgements and Funding Sources](#acknowledgements-and-funding-sources)

## General

ARTEMIS is a defense approach versus BGP prefix hijacking attacks
(a) based on accurate and fast detection operated by the AS itself,
leveraging the pervasiveness of publicly available BGP monitoring
services and their recent shift towards real-time streaming,
thus (b) enabling flexible and fast mitigation of hijacking events.
Compared to existing approaches/tools, ARTEMIS combines characteristics
desirable to network operators such as comprehensiveness, accuracy, speed,
privacy, and flexibility. With the ARTEMIS approach, prefix hijacking
can be neutralized within a minute!

You can read more about ARTEMIS (and check e.g., news, presentations and related publications)
on the INSPIRE Group ARTEMIS [webpage](http://www.inspire.edu.gr/artemis).

This repository contains the software of ARTEMIS as a tool.
ARTEMIS can be run on a testing server/VM as a modular (and extensible)
multi-container application. Up to today it has been tested at a major 
greek ISP, a dual-homed edge academic network (our home institutional network),
and a major R&E US backbone network.

## Features

For a detailed list of supported features please check the [CHANGELOG](CHANGELOG.md) file
(section: "Added"). On a high level, the following main features are supported:

* Real-time monitoring of the changes in the BGP routes of the network's prefixes.
* Real-time detection and notifications of BGP prefix hijacking attacks/events of the following types:
exact-prefix type-0/1, sub-prefix of any type, and squatting attacks.
* Automatic/custom tagging of detected BGP hijack events (ongoing, resolved, ignored, under mitigation, withdrawn and outdated).
* Manual or manually controlled mitigation of BGP prefix hijacking attacks.
* Comprehensive web interface.
* Configuration file editable by the operator (directly or via the web interface),
containing information about: prefixes, ASNs, monitors and ARTEMIS rules ("ASX originates prefix P and advertises it to ASY").
* Support for both IPv4 and IPv6 prefixes.
* Modularity/extensibility by design.

## System Architecture

![Architecture](docs/images/artemis_system_overview.png)

The philosophy behind the ARTEMIS architecture is the use of a message bus (MBUS) in its core,
used for routing messages (RPC, pub/sub, etc.) between different processes/services
and containers, using the kombu framework (https://github.com/celery/kombu) to interface
between rabbitmq and the message senders/receivers (e.g., consumers). The controller/supervisor
service is responsible for checking the status of the other backend services.
In a nutshell, there are 6 basic modules:
* Configuration
* Monitoring
* Detection
* Mitigation
* DB access/management
* Clock
* Observer
* User interface

The operator (i.e., the “user”) interfaces with the system by filling in a configuration file
and by interacting with the web application (UI) to control the various modules and
see useful information related to monitoring entries and detected hijacks (including their
current status). Configuration is imported in all modules since it is used for monitor
filtering, detection tuning, mitigation configuration and other functions. The feed from
the monitoring module (which can stem from multiple BGP monitoring sources around the world,
including local monitors) is validated and transmitted to the detection and db access modules.
The detection module reasons about whether what it sees is a hijack or not; if it is, it
generates a hijack entry which is in turn stored in the DB, together with the corresponding
monitoring entries. Finally, using a web application, the operator can instruct the mitigation
module to mitigate a hijack or mark it as resolved/ignored. All information is persistently
stored in the DB, which is accessed by the web application (user interface).
Clock and observer modules are auxiliary, and take care of periodic clock signaling and configuration
change notifications, respectively. For brevity we do not elaborate more on further auxiliary modules.

## Getting Started

ARTEMIS is built as a multi-container Docker application.
The following instructions will get you a containerized
copy of the ARTEMIS tool up and running on your local machine
for testing purposes. For instructions on how to set up ARTEMIS
in e.g., a Kubernetes environment, please contact the [ARTEMIS team](#development-team-and-contact).

## Min. technical requirements of testing server/VM

* CPU: 4 cores
* RAM: 4 GB
* HDD: 100 GB (less may suffice, depending on the use case)
* NETWORK: 1 public-facing network interface
* OS: Ubuntu Linux 16.04+
* SW PACKAGES: docker-ce and docker-compose should be pre-installed (see instructions later)
and docker should have sudo privileges, if only non-sudo user is allowed
* Other: SSH server

Moreover, one may optionally configure firewall rules related to the testing server/VM.
We recommend using [ufw](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04)
for this task. Please check the comments in the respective script we provide and
set the corresponding <> fields in the file before running:
```
sudo ./other/ufw_setup.sh
```

## How to install

Make sure that your Ubuntu package sources are up-to-date:
```
sudo apt-get update
```

If not already installed, follow the instructions
[here](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce)
to install the latest version of the docker tool for managing containers,
and [here](https://docs.docker.com/compose/install/#install-compose)
to install the docker-compose tool for supporting multi-container Docker applications.

If you would like to run docker without using sudo, please create
a docker group, if not existing:
```
sudo groupadd docker
```
and then add the user to the docker group:
```
sudo usermod -aG docker $USER
```
For more instructions and potential debugging on this please consult this
[webpage](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user).

Install ntp for time synchronization:
```
sudo apt-get install ntp
```

Install git for downloading ARTEMIS:
```
sudo apt-get install git
```
and then download ARTEMIS from github (if not already downloaded).

Note that while the backend and frontend code is available in the repository,
docker-compose is configured to pull the latest images that are built remotely
on [docker cloud](https://cloud.docker.com/)(TBD). 
In any case, you can build ARTEMIS locally by running:
```
docker-compose -f docker.compose.yaml -f docker_compose.<extra_service>.yaml build
```
after you have entered the root folder of the cloned ARTEMIS repo. Note that extra services are
currently the following (it is optional to use them):
* exabgp: local exaBGP monitor
* grafana: alternative UI (currently not in use)
* migrate: for migration of already existing DBs in production deployments
* pgadmin: UI for database management
* syslog: additional syslog container for collecting ARTEMIS logs
* test: test container (under development)

## How to configure and run

Please check our [wiki](https://github.com/FORTH-ICS-INSPIRE/artemis/wiki).

## Contributing
Please check [this file](CONTRIBUTING.md).

## Development Team and Contact
We follow a custom Agile approach for our development.
You can contact the ARTEMIS developers as follows:
* Dimitrios Mavrommatis (backend): mavromat_at_ics_dot_forth_dot_gr
* Petros Gigis (frontend): gkigkis_at_ics_dot_forth_dot_gr
* Vasileios Kotronis (coordinator): vkotronis_at_ics_dot_forth_dot_gr

## Versioning
Please check [this file](CHANGELOG.md).

## Authors and Contributors
Please check [this file](AUTHORS.md).

## License
The ARTEMIS software is open-sourced under the BSD-3 license.
Please check the [license file](LICENSE).

Note that all external dependencies are used in a way compatible with BSD-3
(that is, we obey the compatibility rules of each and every dependency);
the associated software packages and their respective licenses are documented
in detail in [this file](DEPENDENCIES-LICENSES.md), where we provide links
to their homepages and licenses.

## Acknowledgements and Funding Sources
Please check [this file](ACKS.md).
