# WPV1R: WP Local Docker v1 Revival

> This is an active fork of 10up/wp-local-docker, focused on linux hosts.
> 
> WP Local Docker is a Docker based local development environment for WordPress.

[![Support Level](https://img.shields.io/badge/support-tentative-blue.svg)](#support-level) [![MIT License](https://img.shields.io/github/license/10up/wp-local-docker.svg)](https://github.com/10up/wp-local-docker/blob/master/LICENSE.md)

## Foreword
### What is the difference between this project, WP Local Docker v1 and WP Local Docker v2?
- WPV1R is a docker-based project focused on allowing developers to easily create and modify containers to use when developing
  sites using WordPress. It is aimed at developers who want the most flexibility and the freedom to modify any piece of the
  server stack to fit their needs.
- [WP Local Docker v1](https://github.com/10up/wp-local-docker) is an archived project, no longer maintained by 10up and forked to become WPV1R
- [WP Local Docker v2](https://github.com/10up/wp-local-docker-v2) is an npm tool in active development, focused on the ability to run multiple
docker installs of WordPress, using the same main shared structure. It also features a neat installer and some pretty fancy tricks to ease and
automate the creation of new local environments.
  
### How do I choose a tool that fits my needs?
- If all you want is something that "just works" and you have no desire to tweak your containers unless absolutely necessary
  you will be more comfortable using [WP Local Docker v2](https://github.com/10up/wp-local-docker-v2). Even more so if you are doing your work on Windows or Mac hosts.

- If you are familiar with the command-line and want the most flexibility to set and configure all the containers in your stack
  at the expense of running a single environment at a time, choose WPV1R.
  
### Does it work on Mac or Windows?
I don't know :) It is very likely to work, because the foundation built originally was pretty great, but this project is not officially supported on platforms other than Linux.

## Table of Contents  
* [Overview](#overview)
* [Requirements](#requirements)
* [Setup](#setup)
* [Administrative Tools](#administrative-tools)
* [Docker Compose Overrides File](#docker-compose-overrides-file)
* [WP CLI](#wp-cli)
* [SSH Access](#ssh-access)
* [Useful Bash Aliases](#useful-bash-aliases)
* [MailCatcher](#mailcatcher)
* [WP Snapshots](#wp-snapshots)
* [Xdebug](#xdebug)
	* [Visual Studio Code](#visual-studio-code)
* [Updating WP Local Docker](#updating-wp-local-docker)
* [Credits](#credits)

## Overview

This project is based on [docker-compose](https://docs.docker.com/compose/). By default, the following containers are started: PHP-FPM, MySQL, Elasticsearch, Kibana, nginx, and Memcached. The `/wordpress` directory is the web root which is mapped to the nginx container.

You can directly edit PHP, nginx, and Elasticsearch configuration files from within the repo as they are mapped to the correct locations in containers.

The `/config/elasticsearch/plugins` folder is mapped to the plugins folder in the Elasticsearch container. You can drop Elasticsearch plugins in this folder to have them installed within the container.

## Requirements

* [Docker](https://www.docker.com/)
* [docker-compose](https://docs.docker.com/compose/)

## Setup

1. `git clone https://github.com/10up/wp-local-docker.git <my-project-name>`
1. `cd <my-project-name>`
1. `docker-compose up`
1. Run setup to download and install WordPress.
	1. On Linux / Unix / OSX, run `sh bin/setup.sh`.
	2. On Windows, run `.\bin\setup`.

If you want to use a domain other than `http://localhost`, you'll need to:
1. Add an entry to your hosts file. Ex: `127.0.0.1 docker.localhost`
1. Update _WordPress Address (URL)_ and _Site Address (URL)_.

Default MySQL connection information (from within PHP-FPM container):

```
Database: wordpress
Username: wordpress
Password: password
Host: mysql
```

Default WordPress admin credentials:

```
Username: admin
Password: password
```

Note: if you provided details different to the above during setup, use those instead.

Default Elasticsearch connection information (from within PHP-FPM container):

```
Host: http://elasticsearch:9200
```

The Elasticsearch container is configured for a maximum heap size of 750MB to prevent out of memory crashes when using the default 2GB memory limit enforced by Docker for Mac and Docker for Windows installations or for Linux installations limited to less than 2GB. If you require additional memory for Elasticsearch override the value in a `docker-compose.override.yml` file as described below.

## Administrative Tools

We've bundled a simple administrative override file to aid in local development where appropriate. This file introduces both [phpMyAdmin](https://www.phpmyadmin.net/) and [phpMemcachedAdmin](https://github.com/elijaa/phpmemcachedadmin) to the Docker network for local administration of the database and object cache, respectively.

You can run this atop a standard Docker installation by specifying _both_ the standard and the override configuration when initializing the service:

```
docker-compose -f docker-compose.yml -f admin-compose.yml up
```

The database tools can be accessed [on port 8092](http://localhost:8092).

The cache tools can be accessed [on port 8093](http://localhost:8093).

## Docker Compose Overrides File

Adding a `docker-compose.override.yml` file alongside the `docker-compose.yml` file, with contents similar to
the following, allows you to change the domain associated with the cluster while retaining the ability to pull in changes from the repo.

```
version: '3'
services:
  phpfpm:
    extra_hosts:
      - "dashboard.localhost:172.18.0.1"
  elasticsearch:
    environment:
      ES_JAVA_OPTS: "-Xms2g -Xmx2g"
```

## WP-CLI

Add this alias to `~/.bash_profile` to easily run WP-CLI command.

```
alias dcwp='docker-compose exec --user www-data phpfpm wp'
```

Instead of running a command like `wp plugin install` you instead run `dcwp plugin install` from anywhere inside the
`<my-project-name>` directory, and it runs the command inside of the php container.

There is also a script in the `/bin` directory that will allow you to execute WP CLI from the project directory directly: `./bin/wp plugin install`.

## SSH Access

You can easily access the WordPress/PHP container with `docker-compose exec:

```
docker-compose exec --user root phpfpm bash
```

Alternatively, there is a script in the `/bin` directory that allows you to SSH in to the environment from the project directory directly: `./bin/ssh`.

## Useful Bash Aliases

To increase efficiency with WP Local Docker, the following bash aliases can be added `~/.bashrc` or `~/.bash_profile`:

1. WP-CLI:
    ```bash
    alias dcwp='docker-compose exec --user www-data phpfpm wp'
    ```
2. SSH into container:
    ```bash
    alias dcbash='docker-compose exec --user root phpfpm bash'
    ```
3. Multiple instances cannot be run simultaneously. In order to switch projects, you'll need to kill all Docker containers first: 
    ```bash
    docker-stop() { docker stop $(docker ps -a -q); }
    ```
4. Combine the stop-all command with `docker-compose up` to easily start up an instance with one command: 
    ```bash
    alias dup="docker-stop && docker-compose up -d"
    ```

## MailCatcher

MailCatcher runs a simple local SMTP server which catches any message sent to it, and displays it in its built-in web interface. All emails sent by WordPress will be intercepted by MailCatcher. To view emails in the MailCatcher web interface, navigate to `http://localhost:1080` in your web browser of choice.

## WP Snapshots

[WP Snapshots](https://github.com/10up/wpsnapshots) is a project sharing tool for WordPress empowering developers to easily push snapshots of projects into the cloud for sharing with team members. Team members can pull snapshots such that everyhing "just works".  WP Local Docker comes bundled with WP Snapshots and comes with a bin script to proxy commands from the host to the docker containers.  To use WP Snapshots with WP Local Docker, follow the [configuration instructions](https://github.com/10up/wpsnapshots#configure), substituting `./bin/wpsnapshots.sh` for `wpsnapshots` in the CLI.

Example:

```sh
./bin/wpsnapshots.sh configure 10up
```

Once configured, you can use all of the WP Snapshots commands, again substituting `./bin/wpsnapshots.sh` for `wpsnapshots` in the CLI.

Examples:

```sh
./bin/wpsnapshots.sh push
./bin/wpsnapshots.sh pull <snapshot-id>
./bin/wpsnapshots.sh search <search-text>
```

## Xdebug

[Xdebug](https://xdebug.org/) is a PHP extension to assist with debugging and development.

In order to use remote Xdebugging follow the instructions below according to your favorite IDE.

### Visual Studio Code

1. Install the [PHP Debug](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug) extension.
2. In your project, go to the debugger, click the gear icon and choose PHP. A new launch configuration will be created for you.
3. Set the `pathMappings` settings in your launch.json. Example:
```json
"configurations": [
    {
        "name": "Listen for XDebug",
        "type": "php",
        "request": "launch",
        "port": 9000,
        "pathMappings": {
            "/var/www/html": "${workspaceRoot}/wordpress",
        }
    },
    //...
]
```

## Updating WP Local Docker

WP Local Docker is an ever-evolving tool, and it's important to keep your local install up-to-date. Don't forget to `git pull` the latest WP Local Docker code every once in a while to make sure you're running the latest version. We also recommend "watching" this repo on GitHub to stay on top of the latest development. You won’t need to grab every update, but you’ll be aware of bug fixes and enhancements that’ll keep your local development environments running smoothly.

It's especially important to `git pull` the latest code before you `docker pull` upgrades to your Docker images, either as a potential fix for an issue or just to make sure they’re running the latest versions of everything. This will make sure you have the latest WP Local Docker code first, including the `docker-compose.yml` file that defines what Docker images and versions the environment uses.

## Support Level

**Tentative:** 10up has stopped working on this version of WP Local Docker to focus their efforts on WP Local Docker V2. 
This is my own spin to it, as it gets used for my development tasks. It does run on Windows and Macs, but it will not be 
actively supported on platforms other than Linux.

## Credits

    This project is my own version
        - of 10up's own flavor
             - of an environment created by John Bloch.
