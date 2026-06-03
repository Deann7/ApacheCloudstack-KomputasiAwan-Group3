# Single Node Apache CloudStack Private Cloud Installation Guide

## Program Studi Teknik Komputer, Departemen Teknik Elektro, Universitas Indonesia

![Universitas Indonesia](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)


## Table of Contents
- [Background](#background)
- [Objectives](#objectives)
- [Scope](#scope)
- [Contributors](#contributors)
- [Prerequisites](#prerequisites)
- [Installation Guide](#installation-guide)
  - [1. System Preparation](#1-system-preparation)
  - [2. Install Chrony (Time Synchronization)](#2-install-chrony-time-synchronization)
  - [3. Install Java 17 JRE](#3-install-java-17-jre)
  - [4. Install bridge-utils](#4-install-bridge-utils)
  - [5. Add CloudStack Repository](#5-add-cloudstack-repository)
  - [6. Add Repository Key](#6-add-repository-key)
  - [7. Install CloudStack Management Server](#7-install-cloudstack-management-server)
  - [8. Install and Configure MySQL Server](#8-install-and-configure-mysql-server)
  - [9. Set Up CloudStack Database](#9-set-up-cloudstack-database)
  - [10. Configure NFS for Storage](#10-configure-nfs-for-storage)
  - [11. Configure Firewall](#11-configure-firewall)
  - [12. Start CloudStack Services](#12-start-cloudstack-services)
  - [13. Install KVM Hypervisor](#13-install-kvm-hypervisor)
  - [14. Install CloudStack Agent & UI](#14-install-cloudstack-agent--ui)
  - [15. Start CloudStack Management](#15-start-cloudstack-management)
  - [16. Access the CloudStack UI](#16-access-the-cloudstack-ui)
  - [17. Add Zone (Advanced)](#17-add-zone-advanced)
- [Troubleshooting](#troubleshooting)
- [References](#references)


## Background
This document presents an installation guide for Apache CloudStack on a single node to build a private cloud environment. 

Apache CloudStack is an open-source cloud computing platform that enables the deployment and management of large networks of virtual machines, delivering Infrastructure-as-a-Service (IaaS) capabilities.

## Objectives
- Provide a step-by-step installation guide for Apache CloudStack in a single-node environment.
- Present a well-structured and easy-to-follow report.
- Document the complete installation process including troubleshooting encountered during setup.

## Scope
This guide covers:

- Environment preparation and software prerequisites.
- Apache CloudStack installation steps on a single node running Ubuntu 22.04.
- Basic configuration for private cloud access including KVM hypervisor setup, NFS storage configuration, and zone creation.

## Contributors
**Group 3:**

| Name | NIM |
|------|-----|
| Deandro Najwan Ahmad Syahbanna | 2302613174 |
| Muhammad Nadzhif Fikri | 2306210102 |
| Muhamad Rey Kafaka Fadlan | 2306250573 |
| Kharisma Aprilia | 2306223244 |
| Ekananda Zhafif Dean | 2306264420 |
| Dwigina Sitti Zahwa | 2306250724 |
| Azka Nabihan | 2306250541 |
| Dimas Dandossi W P | 2206059780 |
| Abednego Zebua | 2306161883 |


## Prerequisites
Before starting the installation, ensure the following requirements:

- **OS:** Ubuntu 22.04 LTS with hardware virtualization support (Intel VT-x or AMD-V enabled in BIOS)
- **Network:** A `/24` network with a static gateway (e.g., `***.***.***.1`) — avoid DHCP to prevent dynamic IP assignment conflicts for VMs
- **RAM:** Minimum 4 GB (8 GB+ recommended)
- **Storage:** Minimum 50 GB free disk space
- **Access:** Root or `sudo` privileges


## Installation Guide
### 1. System Preparation
Start by updating the package list and upgrading existing packages to ensure the system is up to date.

```bash
sudo apt update -y
```

![apt update](https://hackmd.io/_uploads/r1b5yutC-e.png)

Install OpenSSH Server to allow remote access to the machine:

```bash
sudo apt install openssh-server -y
```

![install openssh](https://hackmd.io/_uploads/HkUiNOKA-e.png)

Check the machine's IP address. Note it down — this will be used throughout the configuration:

```bash
ip a
```

![ip address](https://hackmd.io/_uploads/HkG0H_Y0bl.png)

> **Note:** IP `192.168.105.197/24`, replace this with your actual IP address in the steps below.

Verify hostname and network connectivity:

```bash
hostname --fqdn
ping 8.8.8.8
```


### 2. Install Chrony (Time Synchronization)
CloudStack is sensitive to time drift. Chrony is used to keep the system clock synchronized, which is critical for the management server and hypervisor agents to communicate properly.

```bash
sudo apt install chrony -y
```

![install chrony](https://hackmd.io/_uploads/H1DU2-kyfx.png)


### 3. Install Java 17 JRE
CloudStack requires Java 17 JRE to run the management server.

```bash
sudo apt install openjdk-17-jre -y
```

![install java](https://hackmd.io/_uploads/B1Rd2WJyzl.png)


### 4. Install bridge-utils
`bridge-utils` provides tools to configure network bridges, required by CloudStack for VM network connectivity.

```bash
sudo apt install bridge-utils -y
```

![install bridge-utils](https://hackmd.io/_uploads/S1FRNZk1Gl.png)


### 5. Add CloudStack Repository
Add the official Apache CloudStack 4.20 package repository for Ubuntu:

```bash
echo "deb https://download.cloudstack.org/ubuntu focal 4.20" | sudo tee /etc/apt/sources.list.d/cloudstack.list
```

![add repo](https://hackmd.io/_uploads/r1GxT-1yfe.png)

Update the package list after adding the repository:

```bash
sudo apt update
```


### 6. Add Repository Key
Import the CloudStack repository GPG key to verify package authenticity:

```bash
wget -O - https://download.cloudstack.org/release.asc | sudo tee /etc/apt/trusted.gpg.d/cloudstack.asc
```

![repo key](https://hackmd.io/_uploads/H1L86Zy1fl.png)

---

### 7. Install CloudStack Management Server
Install the CloudStack management server package:

```bash
sudo apt install cloudstack-management -y
```


### 8. Install and Configure MySQL Server
CloudStack uses MySQL as its database backend.

**Install MySQL Server:**

```bash
sudo apt install mysql-server -y
```

![mysql install](https://hackmd.io/_uploads/rkiOMMyJzl.png)

**Edit the MySQL configuration file** to tune it for CloudStack's requirements:

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

Add the following lines under the `[mysqld]` section:

```ini
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
```

![mysql config](https://hackmd.io/_uploads/H1XeAW1kMe.png)

**Restart MySQL** to apply the configuration changes:

```bash
sudo systemctl restart mysql
```

**Secure the MySQL installation** — answer `Y` to all prompts:

```bash
sudo mysql_secure_installation
```

![mysql secure](https://hackmd.io/_uploads/BJ-LJMyJMx.png)


### 9. Set Up CloudStack Database
Use the CloudStack setup script to initialize the database. Replace `<dbpassword>`, `<rootpassword>`, and `<management_server_ip>` with your actual values:

```bash
cloudstack-setup-databases cloud:<dbpassword>@localhost \
  --deploy-as=root:<rootpassword> \
  -e file \
  -m management_key \
  -k database_key \
  -i <management_server_ip>
```

> **Example:** If your management server IP is `192.168.105.197`, replace `<management_server_ip>` with `192.168.105.197`.

Alternatively, you can manually configure the database by logging into MySQL and running:

```sql
CREATE DATABASE cloud;
CREATE DATABASE cloud_usage;
CREATE USER 'cloud'@'localhost' IDENTIFIED BY '<password>';
CREATE USER 'cloud'@'%' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON cloud.* TO 'cloud'@'localhost';
GRANT ALL PRIVILEGES ON cloud.* TO 'cloud'@'%';
GRANT ALL PRIVILEGES ON cloud_usage.* TO 'cloud'@'localhost';
GRANT ALL PRIVILEGES ON cloud_usage.* TO 'cloud'@'%';
GRANT PROCESS ON *.* TO 'cloud'@'localhost';
GRANT PROCESS ON *.* TO 'cloud'@'%';
FLUSH PRIVILEGES;
```

![cloudstack db setup](https://hackmd.io/_uploads/Hkjazf1Jfx.png)

![cloudstack db setup 2](https://hackmd.io/_uploads/BJmhffyyfx.png)


### 10. Configure NFS for Storage
CloudStack uses NFS to provide shared storage for Primary Storage (VM disks) and Secondary Storage (templates, ISO images, and snapshots).

**Install the NFS kernel server:**

```bash
sudo apt install nfs-kernel-server -y
```

**Create the storage export directories:**

```bash
sudo mkdir -p /export/primary
sudo mkdir -p /export/secondary
```

**Edit the NFS exports file** to expose the directories to your network:

```bash
sudo vi /etc/exports
```

Add the following line (replace with your actual network subnet):

```
/export *(rw,async,no_root_squash,no_subtree_check)
```

**Apply the export changes:**

```bash
sudo exportfs -a
```

![nfs config](https://hackmd.io/_uploads/SJ9QXM1kzx.png)

![nfs config 2](https://hackmd.io/_uploads/ryCA7zJyMl.png)

![nfs config 3](https://hackmd.io/_uploads/H1-jXG1JMe.png)

![nfs config 4](https://hackmd.io/_uploads/SJGRgvi1fg.png)

**Edit the NFS kernel server defaults** to set fixed ports (required for firewall rules):

```bash
sudo vi /etc/default/nfs-kernel-server
```

Uncomment or add the following lines:

```ini
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892
RQUOTAD_PORT=875
STATD_PORT=662
STATD_OUTGOING_PORT=2020
```

![nfs kernel config](https://hackmd.io/_uploads/rJrHVf1kMx.png)

**Restart the NFS server** to apply changes:

```bash
sudo systemctl restart nfs-kernel-server
```


### 11. Configure Firewall
Open the required ports for CloudStack and NFS communication. Replace `192.168.105.0/24` with your actual network subnet:

```bash
# NFS and RPC ports
sudo ufw allow from 192.168.105.0/24 to any proto udp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 2049
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 32803
sudo ufw allow from 192.168.105.0/24 to any proto udp port 32769

# CloudStack Management UI port
sudo ufw allow 8080/tcp
```

![firewall rules](https://hackmd.io/_uploads/r1x-LDi1Gg.png)

![firewall rules 2](https://hackmd.io/_uploads/S1qvBwsyMl.png)


### 12. Start CloudStack Services
Enable and start the required background services:

```bash
sudo systemctl start rpcbind
sudo systemctl start nfs-kernel-server
sudo systemctl enable rpcbind
sudo systemctl enable nfs-kernel-server
```

Run the CloudStack management setup script:

```bash
sudo cloudstack-setup-management
```


### 13. Install KVM Hypervisor
CloudStack uses KVM as the hypervisor to run virtual machines. Install KVM and its dependencies:

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst
```

**Add your user to the `libvirt` and `kvm` groups** to allow managing VMs without root:

```bash
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG kvm $(whoami)
```

Apply the group changes in the current session:

```bash
newgrp libvirt
```

**Enable and start the `libvirtd` service:**

```bash
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
```

Verify that libvirtd is running:

```bash
systemctl status libvirtd
```

![libvirtd status](https://hackmd.io/_uploads/rJ9HRDoJze.png)

Verify KVM is working by listing all virtual machines (should return an empty list):

```bash
sudo virsh list --all
```


### 14. Install CloudStack Agent & UI
Install the CloudStack Agent — this runs on the hypervisor host and communicates with the management server:

```bash
sudo apt install cloudstack-agent -y
```

Install the CloudStack Web UI:

```bash
sudo apt install cloudstack-ui -y
```


### 15. Start CloudStack Management
Reboot the server to ensure all services and configurations are cleanly loaded:

```bash
sudo reboot
```

After rebooting, start the CloudStack management service:

```bash
sudo systemctl start cloudstack-management
```

Check that the service is running correctly:

```bash
systemctl status cloudstack-management
```

![management status](https://hackmd.io/_uploads/B1jwE10JGx.png)

> **Tip:** It may take a few minutes for CloudStack to fully initialize after the first start. Monitor the logs with:
> ```bash
> sudo tail -f /var/log/cloudstack/management/management-server.log
> ```


### 16. Access the CloudStack UI
Once the management service is running, open a browser and navigate to:

```
http://<Management-Server-IP>:8080/client
```

> **Example:** `http://192.168.105.197:8080/client` or `http://w1660:8080/client`

**Default login credentials:**

| Field | Value |
|-------|-------|
| Username | `admin` |
| Password | `password` |

![cloudstack login](https://hackmd.io/_uploads/B1f7210JMg.png)

![cloudstack dashboard](https://hackmd.io/_uploads/r1skT1AkMg.png)

![cloudstack dashboard 2](https://hackmd.io/_uploads/rktD6eAyzl.png)

> **Important:** Change the default admin password immediately after first login for security.


### 17. Add Zone (Advanced)
A Zone is the largest organizational unit in CloudStack. This setup uses the **Advanced** networking zone configuration for more granular network control.

> **Note:** A Basic Zone was initially attempted but failed to launch due to errors, so the Advanced Zone setup is used instead.

Navigate to **Infrastructure → Zones → Add Zone** and select **Advanced** as the network type.

![add zone](https://hackmd.io/_uploads/SySZ8-RyMe.png)

![zone config 1](https://hackmd.io/_uploads/S1UnL-Rkfe.png)

Fill in the zone details including the zone name, DNS addresses, and internal DNS:

![zone config 2](https://hackmd.io/_uploads/BkYMOZRkfe.png)

Configure the network settings for the zone:

![zone config 3](https://hackmd.io/_uploads/SJ87dZR1zx.png)

![zone config 4](https://hackmd.io/_uploads/B1kSO-0yzg.png)

![zone config 5](https://hackmd.io/_uploads/B1YCOZRJfl.png)

Configure the pod settings (pod name, reserved system gateway, netmask, and IP range):

![zone config 6](https://hackmd.io/_uploads/HyOEFbR1zx.png)


## Troubleshooting
### Agent Not Activating / Host Cannot Be Added
**Symptom:** The CloudStack agent on the host refuses to activate or the management server cannot connect to the host.

**Steps taken:**

1. Check time synchronization status — CloudStack requires the management server and agent clocks to be in sync:

```bash
chronyc tracking
```

![chrony tracking](https://hackmd.io/_uploads/HkS0a201Gg.png)

2. Verify the hostname resolves correctly by adding it to `/etc/hosts`:

```bash
sudo vi /etc/hosts
```

Add the following line (replace with your actual IP and hostname):

```
192.168.105.197 w1660
```

![etc hosts](https://hackmd.io/_uploads/SytAp2CyMg.png)

3. Check the CloudStack agent logs for errors:

```bash
sudo tail -f /var/log/cloudstack/agent/agent.log
```

4. Restart the agent service:

```bash
sudo systemctl restart cloudstack-agent
systemctl status cloudstack-agent
```

5. If the agent still fails, check for port conflicts and ensure `libvirtd` is running:

```bash
sudo systemctl status libvirtd
sudo netstat -tlnp | grep 16509
```

> **Known Issue:** In this setup, the agent was intermittently unresponsive despite correct configuration. This appears to be related to timing and host resolution. If the agent continues to fail after the steps above, restarting both `cloudstack-management` and `cloudstack-agent` services in sequence and waiting a few minutes often resolves the issue.


## References
- [Installing Apache CloudStack on Ubuntu 22 — Pratyukt (Hashnode)](https://pratyukt.hashnode.dev/installing-apache-cloudstack-on-ubuntu-22)
- [Apache CloudStack Official Documentation](https://docs.cloudstack.apache.org/)
- [Apache CloudStack GitHub](https://github.com/apache/cloudstack)
