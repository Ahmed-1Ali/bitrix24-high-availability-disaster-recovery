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
                                                                      \|
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


--- 

## Failover and Application Redundancy

The Primary and Secondary Bitrix application servers provide application-level redundancy.

Both application servers use centralized NFS storage for shared Bitrix uploads and application data.

--- 

## NFS Configuration

### Install NFS Client
```bash
dnf install -y nfs-utils
```

### Check NFS Exports
```bash
showmount -e <NFS_SERVER_IP>
```

### Mount Shared Storage
```bash
mkdir -p /home/bitrix/www/upload
mount -t nfs <NFS_SERVER_IP>:/srv/nfs/upload /home/bitrix/www/upload
mount | grep nfs
df -h /home/bitrix/www/upload
```

--- 

## Database Restore

```bash
gunzip -c /backup/<DATABASE_NAME>_<DATE>.sql.gz | mysql -u <DB_USER> -p <DATABASE_NAME>
mysql -u <DB_USER> -p -e "SHOW DATABASES;"
```

--- 

## Linux Service Management

```bash
systemctl status httpd
systemctl status php-fpm
systemctl status mysqld
journalctl -u httpd -n 100 --no-pager
journalctl -u php-fpm -n 100 --no-pager
journalctl -u mysqld -n 100 --no-pager
```

--- 

## Firewall and NFS Troubleshooting

```bash
firewall-cmd --list-all
firewall-cmd --list-services
systemctl status rpcbind
systemctl status nfs-server
rpcinfo -p <NFS_SERVER_IP>
showmount -e <NFS_SERVER_IP>
```

--- 

## SELinux Troubleshooting

```bash
getenforce
ausearch -m AVC -ts recent
ls -Z /home/bitrix/www
restorecon -Rv /home/bitrix/www
```

--- 

## Disaster Recovery

The disaster recovery process was validated through database backup verification, database restore testing, NFS connectivity checks, and application service recovery.

```text
Database Failure
      |
      v
Identify Latest Backup
      |
      v
Restore MySQL Database
      |
      v
Verify Database
      |
      v
Verify NFS Storage
      |
      v
Restart Application Services
      |
      v
Validate Bitrix Application
```

--- 

## Technologies Used

- Rocky Linux
- Bitrix24
- Apache
- PHP / PHP-FPM
- MySQL / Percona
- NFS
- Bash
- systemd
- SELinux
- firewalld
- RPC / RPCBind

--- 

## Key Skills Demonstrated

- Linux system administration
- High-availability application architecture
- Shared storage using NFS
- MySQL database administration
- Automated database backup and restore
- Disaster recovery procedures
- Systemd and service troubleshooting
- Network and firewall troubleshooting
- SELinux and NFS troubleshooting
- Bash scripting and automation

--- 

## Project Outcome

The project demonstrates a production-style Bitrix24 infrastructure focused on application availability, centralized shared storage, database protection, backup automation, and disaster recovery.

--- 

## Failover and Application Redundancy

The environment uses Primary and Secondary Bitrix application servers to provide application-level redundancy.

Both application servers access centralized NFS storage so shared Bitrix uploads and application data remain available across the application layer.

--- 

## NFS Configuration

### Install NFS Client
```bash
dnf install -y nfs-utils
```

### Check NFS Exports
```bash
showmount -e <NFS_SERVER_IP>
exportfs -v
```

### Create Mount Point
```bash
mkdir -p /home/bitrix/www/upload
```

### Mount Shared Storage
```bash
mount -t nfs <NFS_SERVER_IP>:/srv/nfs/upload /home/bitrix/www/upload
```

### Verify NFS Mount
```bash
mount | grep nfs
findmnt -t nfs
df -h /home/bitrix/www/upload
```

### NFS Server Services
```bash
systemctl status nfs-server
systemctl status rpcbind
showmount -e <NFS_SERVER_IP>
```

--- 

## Automated MySQL Backup

Backups are created using `mysqldump`, date-based naming, and gzip compression.

### Create Backup Directory
```bash
mkdir -p /home/bitrix/db_bak
```

### Create MySQL Backup
```bash
mysqldump -u <DB_USER> -p <DATABASE_NAME> > /home/bitrix/db_bak/<DATABASE_NAME>_$(date +%%Y-%%m-%%d).sql
```

### Compress Backup
```bash
gzip /home/bitrix/db_bak/<DATABASE_NAME>_$(date +%%Y-%%m-%%d).sql
```

### Direct Compressed Backup
```bash
mysqldump -u <DB_USER> -p <DATABASE_NAME> | gzip > /home/bitrix/db_bak/<DATABASE_NAME>_$(date +%%Y-%%m-%%d).sql.gz
```

### Verify Backups
```bash
ls -lh /home/bitrix/db_bak/
```

--- 

## Database Restore

Database restoration was used to validate backup integrity and disaster recovery procedures.

### Restore Compressed Backup
```bash
gunzip -c /home/bitrix/db_bak/<DATABASE_NAME>_<DATE>.sql.gz | mysql -u <DB_USER> -p <DATABASE_NAME>
```

### Verify Database
```bash
mysql -u <DB_USER> -p -e "SHOW DATABASES;"
mysql -u <DB_USER> -p <DATABASE_NAME> -e "SHOW TABLES;"
```

--- 

## Linux Service Management

### Check Services
```bash
systemctl status httpd
systemctl status php-fpm
systemctl status mysqld
```

### Restart Services
```bash
systemctl restart httpd
systemctl restart php-fpm
systemctl restart mysqld
```

### Check Logs
```bash
journalctl -u httpd -n 100 --no-pager
journalctl -u php-fpm -n 100 --no-pager
journalctl -u mysqld -n 100 --no-pager
```

--- 

## Firewall Troubleshooting

### Check Firewall
```bash
systemctl status firewalld
firewall-cmd --list-all
firewall-cmd --list-services
firewall-cmd --list-ports
```

### NFS Firewall Services
```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload
```

--- 

## RPC and NFS Troubleshooting

```bash
systemctl status rpcbind
systemctl status nfs-server
rpcinfo -p <NFS_SERVER_IP>
showmount -e <NFS_SERVER_IP>
mount | grep nfs
```

--- 

## SELinux Troubleshooting

### Check SELinux
```bash
getenforce
```

### Check AVC Denials
```bash
ausearch -m AVC -ts recent
```

### Check File Contexts
```bash
ls -Z /home/bitrix/www
```

### Restore File Contexts
```bash
restorecon -Rv /home/bitrix/www
```

--- 

## Disaster Recovery Procedure

The disaster recovery workflow validates database restoration, shared storage availability, application services, and Bitrix application functionality.

```text
Database Failure
      |
      v
Identify Latest Backup
      |
      v
Restore MySQL Database
      |
      v
Verify Database
      |
      v
Verify NFS Storage
      |
      v
Restart Application Services
      |
      v
Validate Bitrix Application
```

--- 

## Useful Troubleshooting Commands

### System Resources
```bash
df -h
free -h
lsblk
```

### Network Diagnostics
```bash
ip addr
ip route
ss -lntp
ping <SERVER_IP>
```

### File Permissions
```bash
ls -lah /home/bitrix/www
stat /home/bitrix/www
```

### Service Logs
```bash
journalctl -xe
journalctl -u <SERVICE_NAME> -n 100 --no-pager
```

--- 

## Technologies Used

- Rocky Linux
- Bitrix24
- Apache
- PHP / PHP-FPM
- MySQL / Percona
- NFS
- Bash
- systemd
- SELinux
- firewalld
- RPC / RPCBind

--- 

## Key Skills Demonstrated

- Linux system administration
- High-availability application architecture
- Shared storage using NFS
- MySQL database administration
- Automated database backup and restore
- Disaster recovery procedures
- Systemd and service troubleshooting
- Network and firewall troubleshooting
- SELinux and NFS troubleshooting
- Bash scripting and automation

--- 

## Project Outcome

The project demonstrates a production-style Bitrix24 infrastructure focused on application availability, centralized shared storage, database protection, backup automation, and disaster recovery.

The environment was also used to troubleshoot infrastructure issues involving MySQL, NFS, firewall rules, RPC/RPCBind, SELinux, permissions, and systemd services.


