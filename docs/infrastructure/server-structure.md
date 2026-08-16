# Server Structure

## Overview

The Raspberry Pi uses `/srv` as the main location for server-related applications, persistent data, configuration, backups, Docker resources, and logs.

## Directory Structure

```text
/srv/
├── apps/
├── backups/
├── config/
├── data/
├── docker/
└── logs/
```

## Directory Purpose

### `/srv/apps`

Contains application source code and deployed server applications.

### `/srv/data`

Contains persistent application and service data.

### `/srv/backups`

Contains backups of important server data and configuration.

### `/srv/docker`

Contains Docker-related configuration, Compose files, and Docker service resources.

### `/srv/config`

Contains configuration files for services and applications.

### `/srv/logs`

Contains application and service logs.

## Design Goal

The directory structure separates applications, persistent data, configuration, backups, Docker resources, and logs.

This makes the server easier to maintain, back up, troubleshoot, and expand as more services are deployed.
