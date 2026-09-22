# Bitrix24 High-Availability, Shared Storage & Disaster Recovery Infrastructure

## Overview

Designed and implemented a high-availability Bitrix24 infrastructure with Primary/Secondary application servers, centralized MySQL/Percona database services, shared NFS storage, and a dedicated backup server.

The infrastructure provides application redundancy, centralized shared storage, database backup and restore capabilities, and disaster recovery procedures.

---

## Architecture

```text

                        +----------------------+
                        |     Bitrix Users     |
                        +----------+-----------+
                                   |
                                   v
                        +----------------------+
                        |   Application Layer  |
                        +----------+-----------+
                                   |
                   +---------------+---------------+
                   |                               |
                   v                               v
            +-------------+                +-------------+
            |   Bitrix    |                |   Bitrix    |
            |   Primary   |                |  Secondary  |
            | Application |                | Application |
            +------+------+                +------+------+
                   |                               |
                   +---------------+---------------+
                                   echo                                    |
                        +----------+-----------+
                        |     Shared NFS       |
                        |       Storage        |
                        |   Uploads / Data     |
                        +----------------------+

                        +----------------------+
                        |   MySQL / Percona    |
                        |    Database Layer    |
                        +----------+-----------+
                                   |
                                   v
                        +----------------------+
                        |   Backup / Restore   |
                        |       Server         |
                        +----------------------+

```

---

## Components

| Component | Role |
|---|---|
| **Bitrix Primary** | Primary application server |
| **Bitrix Secondary** | Secondary application server |
| **MySQL / Percona** | Centralized database service |
| **NFS Server** | Shared application storage |
| **Backup Server** | Database backup and restore |

---

## High Availability

The Bitrix environment uses Primary and Secondary application servers to provide application-level redundancy.

Both application servers access centralized shared NFS storage for Bitrix uploads and shared application data.

The database layer is centralized using MySQL / Percona, while database backups are stored separately to support recovery and disaster recovery scenarios.

---

## Shared NFS Storage

NFS is used to provide centralized access to Bitrix shared files and upload data.

The application servers mount the shared storage and access the same application data through the NFS server.

```text

Bitrix Primary  ----\
                     +----> NFS Server ----> Shared Uploads
Bitrix Secondary ---/

```

---

## Database Backup

Bash automation was implemented to create periodic MySQL backups using `mysqldump`.

Backups use date-based naming and gzip compression to produce `.sql.gz` archives.

Example backup workflow:

```text
MySQL / Percona
      |
      v
  mysqldump
      |
      v
  SQL Backup
      |
      v
    gzip
      |
      v
  .sql.gz Archive
      |
      v
Backup Storage
```

