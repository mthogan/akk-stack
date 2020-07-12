# AkkStack | Containerized EverQuest Emulator Server Environment

<p align="center"><img width="600" src="https://user-images.githubusercontent.com/3319450/87238998-55010c00-c3cf-11ea-8db5-3be25a868ac8.png" alt="AkkStack"></p>

# What is AkkStack ?

AkkStack is a simple Docker Compose environment that is augmented with developer and operator focused tooling for running EverQuest Emulator servers

This is what I've used in production, battle-tested, for almost 2 years. I've worked through a lot of issues to give you the final stable product. It's what I've also used for development for around the same time frame and you will see why shortly

# Requirements

Linux Host or VM with Docker Installed along with Docker Compose

# What's Included

## Containerized Services

| **Service** | **Description**  |
|---|---|
| eqemu-server |  Runs the Emulator server and all services  |
| mariadb | MySQL service |
| phpmyadmin | (Optional) PhpMyAdmin which is automatically configured behind a password proxy |
| peq-editor | (Optional) PEQ Editor which is automatically configured  |
| ftp-quests | (Optional) An FTP instance fully ready to be used to remotely edit quests |
| backup-cron | (Optional) A container built to automatically backup (Dropbox API) the entire deployment and perform database and quest snapshots for with different retention schedules defined in `.env` |

# Features

## Very easy to use CLI menus

Embedded server management CLI (What is used a majority of the time)

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240603-7c140980-c3e0-11ea-9e92-ce18edcfad29.gif"></p>

A `make` menu to manage the in-container environment

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240694-779c2080-c3e1-11ea-8330-26d8add10e5f.gif"></p>

A `make` menu to manage the host-level container environment

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240726-bfbb4300-c3e1-11ea-80ac-e53bfa3386f4.gif"></p>

## SSH

Automatically configured SSH to the `eqemu-server` with automatically generated 30+ character password, persistent keys through reboot
  
## MariaDB

Configurable INNODB_BUFFER_POOL_MEMORY (Default: 256MB) (Must set before make install or rebuild mariadb)

## PEQ Editor

Automatically configured with pre-set admin password; listens on port 8081 by default

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240902-3dcc1980-c3e3-11ea-9d1e-746e217b4459.png"></p>

## PhpMyAdmin

Automatically configured PhpMyAdmin instance with pre-set admin password (Behind a password protected proxy); listens on port 8082 by default

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240916-63f1b980-c3e3-11ea-8dd8-93bca87f54ec.png"></p>

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240919-6f44e500-c3e3-11ea-8c56-6fe0e5ecef89.png"></p>

## Occulus

Automatically installed server admin panel [Occulus repository](https://github.com/Akkadius/eqemu-web-admin); listens on port 3000 by default

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87236540-8c13f500-c3b0-11ea-87f6-756e60fa61ed.png"></p>

## Symlinked resources
  * Server binaries - Never need to copy binaries after a compile
  * Patch files
  * Quests
  * Plugins 
  * LUA Modules

## File Structure

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240837-ba122d00-c3e2-11ea-811f-5ed92f2c79f0.gif"></p>

## Automated Backups

Automated cron-based backups that upload to Dropbox using Dropbox API

Follow instructions below to get an API key to enter into the `.env`

```
This is the first time you run this script, please follow the instructions:

1) Open the following URL in your Browser, and log in using your account: https://www.dropbox.com/developers/apps
2) Click on "Create App", then select "Dropbox API app"
3) Now go on with the configuration, choosing the app permissions and access restrictions to your DropBox folder
4) Enter the "App Name" that you prefer (e.g. MyUploader984915521299)

Now, click on the "Create App" button.
```

Backup retention configurable in `.env`

```
# DEPLOYMENT_NAME=peq-production (used in backup names)
# DROPBOX_OAUTH_ACCESS_TOKEN=
# BACKUP_RETENTION_DAYS_DB_SNAPSHOTS=10
# BACKUP_RETENTION_DAYS_DEPLOYMENT=35
# BACKUP_RETENTION_DAYS_QUEST_SNAPSHOTS=7
```

Crons defined in `backup/crontab.cron`

Crons are configured to run on a variance so that not all deployments fire backups at the same time

| **Backup Type** | **Description** | **Schedule** |
|---|---|---|
| Deployment | Deployment consists of the entire akk-stack folder (server, database etc.). If you ever experienced catastrophic failure or needed to restore the entire setup, simply restoring the deployment folder will get you back up and running | Once a week at 1AM on a random variance of 1800 seconds |
| Quests | A simple snapshot of the quests folder | Once a day at 1M on a random variance of 1800 seconds |
| Database | A simple snapshot of the database | Once a day at 1M on a random variance of 1800 seconds |

## High CPU Process Watchdog

If a zone process goes into an infinite loop; the watchdog will kill the process and log it in the home directory

```
eqemu@f8905f80723c:~$ cat process-kill.log
Sat Jul 11 20:52:47 CDT 2020 [process-watcher] Killed process [21143] [./bin/zone] for taking too much CPU time [43.50]
```

# Installation

First clone the repository somewhere on your server, in this case I'm going to clone it to an `/opt/eqemu-servers` folder in a Debian Linux host with Docker installed

```
root@host:/opt/eqemu-servers# git clone https://github.com/Akkadius/akk-stack.git peq-test-server
Cloning into 'peq-test-server'...
remote: Enumerating objects: 57, done.
remote: Counting objects: 100% (57/57), done.
remote: Compressing objects: 100% (42/42), done.
remote: Total 782 (delta 14), reused 52 (delta 11), pack-reused 725
Receiving objects: 100% (782/782), 101.94 KiB | 7.28 MiB/s, done.
Resolving deltas: 100% (437/437), done.
```

Change into the new directory that represents your server

```
root@host:/opt/eqemu-servers# cd peq-test-server/
```

## Initialize the Environment

There are a ton of configuration variables available in the `.env` file that is produced from running the next command, we will get into that later. The key thing here is that it creates the base `.env` and scrambles all of the password fields in the environment

```
root@host:/opt/eqemu-servers# make init-reset-env
make env-transplant
Wrote updated config to [.env]
make env-scramble-secrets
Wrote updated config to [.env]
```

## Initialize Network Parameters

The next command is going to initialize two large key things in our setup

1) The ip address we're going to use
2) The zone port range we're going to use

Make sure that you only open as many ports as you need on the zone end, because `docker-proxy` will NAT all ports individually in its own docker userland which does take some time when starting and shutting off containers. The more ports you nail up, the longer it takes to start / stop. Since this is a test server, I'm only going to use 30 ports. This `make` command also drives the `eqemu_config.json` port and address parameters as well automatically for you

```
root@host:/opt/eqemu-servers# make set-vars port-range-high=7030 ip-address=66.70.153.122
Wrote [IP_ADDRESS] = [66.70.153.122] to [.env]
Wrote [PORT_RANGE_HIGH] = [7030] to [.env]
```

# Install

From this point you're ready to run the fully automated install with a simple `make install`

An example of what this output looks like below (Sped up)

<p align="center"><img src="https://user-images.githubusercontent.com/3319450/87240353-7289a200-c3de-11ea-8afe-1b0a5ad8400e.gif"></p>
  
# Post-Install

Now that you're installed we need to look at how we interact with the environment

To gain a bash into the emulator server we have two options, we can come through a docker exec entry or we can SSH into the container

## Direct Bash

![make-bash](https://user-images.githubusercontent.com/3319450/87241544-e8473b00-c3e9-11ea-8232-33fa3da9d40b.gif)

## SSH

![make-ssh](https://user-images.githubusercontent.com/3319450/87241545-ea10fe80-c3e9-11ea-9a7f-c97ba54e93fa.gif)

## MySQL Console 

You can hop into MySQL shell from either docker exec `make mc` or from the `eqemu-server` embeded shell alias `mc`

![mysql-shell](https://user-images.githubusercontent.com/3319450/87241546-ec735880-c3e9-11ea-9a8e-412ca4d99736.gif)


  
