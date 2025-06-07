# Ansible Role: apigban.immich\_podman

This Ansible role deploys [Immich](https://immich.app/), a self-hosted photo and video backup solution, using Podman.

## Description

The role automates the setup of Immich, including creating necessary directories and mount points, setting up Podman networks and volumes, and deploying the required Immich containers (PostgreSQL, Redis, Immich Server and Immich Machine Learning). It supports using local volumes or NFS for storing Immich data, uploads, and external libraries.

## Requirements

### Ansible Version

  * min\_ansible\_version: 2.17.5

### Target Platforms

  * EL 9 (e.g., AlmaLinux 9.4)
  * Debian 12 (e.g., Ubuntu 22.04)

## Quickstart

```yaml
---
- hosts: your_immich_server
  roles:
    - role: apigban.immich_podman
  vars:
    # Override any default variables here if needed
    immich_db_password: "your_very_secure_postgres_password"
    immich_redis_password: "your_very_secure_redis_password"
    # Example: If you want to use local volumes instead of NFS
    # external_immich_upload_enabled: false
    # external_lib_enabled: false
```

## Role Variables

Variables are defined in `defaults/main.yml` and can be overridden in `vars/main.yml` or via inventory/playbook.

### NFS Configuration (`defaults/main.yml` and overridden in `vars/main.yml`)

**NFS Immich Upload:**

  * `external_immich_upload_enabled`: Set to `true` to use NFS for uploads (default in `defaults/main.yml` is `false`, overridden to `true` in `vars/main.yml`).
  * `external_immich_upload_src`: NFS server source (e.g., `"storage01.home.apigban.com"`).
  * `external_immich_upload_export_path`: Export path on the NFS server (e.g., `"/media/pool01/immich-upload"`).
  * `external_immich_upload_dst`: Mount point on the host (e.g., `"/mnt/immich-upload"`).

**NFS External Library:**

  * `external_lib_enabled`: Set to `true` to use an NFS external library (default in `defaults/main.yml` is `false`, overridden to `true` in `vars/main.yml`).
  * `external_lib_src`: NFS server source (e.g., `"storage01.home.apigban.com"`).
  * `external_lib_src_export_path`: Export path on the NFS server (e.g., `"/media/pool01/immich-external-libraries"`).
  * `external_lib_dst`: Mount point on the host (e.g., `"/mnt/immich-external-library"`).

## Tasks

The role performs the following main tasks:

1.  **Create Mountpoints (`tasks/01-create-mountpoints.yml`)**:

      * Ensures core Immich data directories exist with correct ownership and permissions.
      * If NFS is enabled for uploads (`external_immich_upload_enabled: true`), it creates the host mount point directory (if not already a mount) and mounts the NFS share.
      * If NFS is enabled for an external library (`external_lib_enabled: true`), it creates the host mount point directory (if not already a mount) and mounts the NFS share.

2.  **Create Podman Network and Volumes (`tasks/02-create-podman-network-and-volumes.yml`)**:

      * Creates the specified Podman network.
      * Creates Podman volumes for PostgreSQL data (`pgdata`), Redis data (`redisdata`), Immich generated files (`immich-generated`), and Immich upload data (`immich-upload` - only if `external_immich_upload_enabled` is false).
      * Creates a Podman volume for JSON log files (`immich-logs`).

3.  **Deploy Containers (`tasks/03-deploy-containers.yml`)**:

      * Deploys the PostgreSQL container, configuring it with environment variables for user, password, and database name, and mounts the `pgdata` volume.
      * Deploys the Redis container, configuring it with a password and mounts the `redisdata` volume.
      * Deploys the Immich Machine Learning container, setting environment variables for database and Redis connection, log level, timezone, and mounts the volume for generated data (model cache).
      * Deploys the Immich Server container, setting environment variables for database, Redis, machine learning URL, log level, timezone, and mounts volumes for uploads (either NFS or local `immich-upload` volume), generated files, and optionally the external library (read-only).

All containers are configured with restart\_policy "always" and appropriate healthchecks (for PostgreSQL and Redis). Log drivers are set to "json-file" with specified log paths and size limits.

## Molecule Tests

This role includes Molecule tests for validating its functionality. The default scenario uses an AlmaLinux 9.4 platform.

  * **Converge (`molecule/default/converge.yml`)**: Applies the role to the test instance and performs basic checks.
  * **Create (`molecule/default/create.yml`)**: Prepares the Molecule instance configuration.
  * **Destroy (`molecule/default/destroy.yml`)**: Clears the Molecule instance configuration.

## License

MIT

## Author Information

Alain Pinero Igban
[apigban.com](https://alain.apigban.com)