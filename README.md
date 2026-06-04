# Single Node Apache CloudStack Private Cloud Installation Guide

## Program Studi Teknik Komputer, Departemen Teknik Elektro, Universitas Indonesia

![Universitas Indonesia](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)



## Table of Contents

- [Background](#background)
- [Objectives](#objectives)
- [Scope](#scope)
- [References](#references)
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
  - [10. Configure NFS for Primary and Secondary Storage](#10-configure-nfs-for-primary-and-secondary-storage)
  - [11. Configure Firewall](#11-configure-firewall)
  - [12. Start CloudStack Services](#12-start-cloudstack-services)
  - [13. Install KVM Hypervisor](#13-install-kvm-hypervisor)
  - [14. Install CloudStack Agent & UI](#14-install-cloudstack-agent--ui)
  - [15. Start CloudStack Management](#15-start-cloudstack-management)
  - [16. Access the CloudStack UI](#16-access-the-cloudstack-ui)
- [Zone Configuration](#zone-configuration)
  - [Add Zone (Basic) — First Attempt](#add-zone-basic--first-attempt)
  - [Add Zone (Advanced)](#add-zone-advanced)
  - [Troubleshooting](#troubleshooting)
- [Known Issues](#known-issues)



## Background

![cover](https://th.bing.com/th/id/R.ca5c6d30f86c5e0e2dbd2f819da0bb0b?rik=0dlTz8ogHC4rQQ&riu=http%3a%2f%2fdocs.cloudstack.apache.org%2fen%2flatest%2f_images%2facslogo.png&ehk=QPmWD4jFZkM2q4JSYGrx3rWXSJZMNCRMlES782jYfaU%3d&risl=&pid=ImgRaw&r=0)



Apache CloudStack is an open-source platform for managing large-scale cloud infrastructure. CloudStack provides a complete IaaS (Infrastructure as a Service) stack that supports multiple hypervisors such as KVM, VMware, and XenServer and exposes a web-based dashboard, a RESTful API, and a command-line interface for managing compute, network, and storage resources. It organizes infrastructure into a logical hierarchy: **Zones** represent physical locations or datacenters, **Pods** correspond to Layer 2 network segments within a zone, **Clusters** group hypervisor hosts of the same type, and **Hosts** are the physical machines that run virtual machine instances. This hierarchy allows CloudStack to intelligently schedule and place VM workloads across available resources.


In this guide, all components including management server, hypervisor, and storage run on a single physical machine. 


## Objectives

- Provide a step-by-step installation guide for Apache CloudStack in a single-node environment.
- Present a well-structured and easy-to-follow report.
- Document the complete installation process including troubleshooting encountered during setup.


## Scope

This guide covers:

- Environment preparation and software prerequisites.
- Apache CloudStack installation steps on a single node running Ubuntu 22.04.
- Basic configuration for private cloud access including KVM hypervisor setup, NFS storage configuration, and zone creation.
- Zone configuration (Basic and Advanced Networking).
- Troubleshooting performed during the installation process.



## References

- [Installing Apache CloudStack on Ubuntu 22 — Pratyukt (Hashnode)](https://pratyukt.hashnode.dev/installing-apache-cloudstack-on-ubuntu-22)
- [Apache CloudStack Official Documentation](https://docs.cloudstack.apache.org/)
- [Apache CloudStack GitHub](https://github.com/apache/cloudstack)



## Contributors

**Group 3:**

| Name | NPM |
|---|---|
| Deandro Najwan Ahmad Syahbanna | 2302613174 |
| Muhammad Nadzhif Fikri | 2306210102 |
| Dwigina Sitti Zahwa | 2306250724 |
| Muhamad Rey Kafaka Fadlan | 2306250573 |
| Kharisma Aprilia | 2306223244 |
| Azka Nabihan | 2306250541 |
| Ekananda Zhafif Dean | 2306264420 |
| Dimas Dandossi W P | 2206059780 |
| Abednego Zebua | 2306161883 |



## Prerequisites

Before starting the installation, ensure the following requirements are met:

| Requirement | Details |
|---|---|
| **OS** | Ubuntu 22.04 LTS with hardware virtualization support (Intel VT-x or AMD-V enabled in BIOS) |
| **Network** | A `/24` network with a static gateway  avoid DHCP to prevent dynamic IP conflicts for VMs |
| **RAM** | Minimum 4 GB (8 GB+ recommended) |
| **Storage** | Minimum 50 GB free disk space |
| **Access** | Root or `sudo` privileges |



## Installation Guide

### 1. System Preparation

Start by updating the package list to ensure the system is up to date:

```bash
sudo apt update -y
```

![apt update](https://hackmd.io/_uploads/r1b5yutC-e.png)

Install OpenSSH Server to allow remote access to the machine:

```bash
sudo apt install openssh-server -y
```

![install openssh](https://hackmd.io/_uploads/HkUiNOKA-e.png)

Check the machine's IP address and note it down — it will be used throughout the configuration:

```bash
ip a
```

![ip address](https://hackmd.io/_uploads/HkG0H_Y0bl.png)

> **Note:** In this setup, the machine IP is `192.168.105.197/24`. Replace this with your actual IP address in all subsequent steps.

Configure Netplan according to your network topology and ensure the network interface is connected to the correct subnet.

Verify hostname and network connectivity:

```bash
hostname --fqdn
ping 8.8.8.8
```



### 2. Install Chrony (Time Synchronization)

Chrony is used for NTP time synchronization. Accurate time synchronization is critical — clock drift between the management server and agents is a common source of CloudStack failures.

```bash
sudo apt install chrony -y
```

![install chrony](https://hackmd.io/_uploads/H1DU2-kyfx.png)



### 3. Install Java 17 JRE

Apache CloudStack requires Java Runtime Environment version 17 to run the management server.

```bash
sudo apt install openjdk-17-jre -y
```

![install java](https://hackmd.io/_uploads/B1Rd2WJyzl.png)



### 4. Install bridge-utils

`bridge-utils` provides tools for creating and managing network bridges on the hypervisor, which are required by CloudStack for VM network connectivity.

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



### 7. Install CloudStack Management Server

Install the CloudStack management server package:

```bash
sudo apt install cloudstack-management -y
```



### 8. Install and Configure MySQL Server

CloudStack uses MySQL as its database backend.

**Step 1 — Install MySQL Server:**

```bash
sudo apt install mysql-server -y
```

![mysql install](https://hackmd.io/_uploads/rkiOMMyJzl.png)

**Step 2 — Edit the MySQL configuration file** to tune the parameters required by CloudStack:

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

**Step 3 — Restart MySQL** to apply the configuration changes:

```bash
sudo systemctl restart mysql
```

**Step 4 — Secure the MySQL installation** — answer `Y` to all prompts:

```bash
sudo mysql_secure_installation
```

![mysql secure](https://hackmd.io/_uploads/BJ-LJMyJMx.png)



### 9. Set Up CloudStack Database

Use the built-in CloudStack setup script to initialize the database. Replace `<dbpassword>`, `<rootpassword>`, and `<management-server-ip>` with your actual values:

```bash
sudo cloudstack-setup-databases cloud:<dbpassword>@localhost \
  --deploy-as=root:<rootpassword> \
  -e file \
  -m management_key \
  -k database_key \
  -i <management-server-ip>
```

> **Example:** If your management server IP is `192.168.105.197`:
> ```bash
> sudo cloudstack-setup-databases cloud:cloud@localhost \
>   --deploy-as=root:<rootpassword> \
>   -e file -m management_key -k database_key \
>   -i 192.168.105.197
> ```

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

![cloudstack db setup 1](https://hackmd.io/_uploads/Hkjazf1Jfx.png)

![cloudstack db setup 2](https://hackmd.io/_uploads/BJmhffyyfx.png)



### 10. Configure NFS for Primary and Secondary Storage

NFS (Network File System) is used by CloudStack as Primary Storage (VM disks) and Secondary Storage (templates, ISO images, and snapshots).

**Step 1 — Install the NFS kernel server:**

```bash
sudo apt install nfs-kernel-server -y
```

**Step 2 — Create the storage export directories:**

```bash
sudo mkdir -p /export/primary
sudo mkdir -p /export/secondary
```

**Step 3 — Edit the NFS exports file** to expose the directories to your network:

```bash
sudo vi /etc/exports
```

Add the following line (replace with your actual network subnet):

```
/export *(rw,async,no_root_squash,no_subtree_check)
```

**Step 4 — Apply the export changes:**

```bash
sudo exportfs -a
```

![nfs config 1](https://hackmd.io/_uploads/SJ9QXM1kzx.png)

![nfs config 2](https://hackmd.io/_uploads/ryCA7zJyMl.png)

![nfs config 3](https://hackmd.io/_uploads/H1-jXG1JMe.png)

![nfs config 4](https://hackmd.io/_uploads/SJGRgvi1fg.png)

**Step 5 — Edit the NFS kernel server defaults** to set fixed ports (required for firewall rules):

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

**Step 6 — Restart the NFS server** to apply all changes:

```bash
sudo systemctl restart nfs-kernel-server
```



### 11. Configure Firewall

Open the required ports for CloudStack and NFS communication. The table below lists all ports that must be allowed:

| Port  | Protocol | Purpose                  |
|-|-|--|
| 111   | TCP/UDP  | RPC Portmapper           |
| 2049  | TCP      | NFS                      |
| 32803 | TCP      | NFS Lock Daemon (lockd)  |
| 32769 | UDP      | NFS Lock Daemon (lockd)  |
| 8080  | TCP      | CloudStack Management UI |

Run the following commands (replace `192.168.105.0/24` with your actual network subnet):

```bash
sudo ufw allow from 192.168.105.0/24 to any proto udp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 2049
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 32803
sudo ufw allow from 192.168.105.0/24 to any proto udp port 32769
sudo ufw allow 8080/tcp
```

![firewall rules 1](https://hackmd.io/_uploads/r1x-LDi1Gg.png)

![firewall rules 2](https://hackmd.io/_uploads/S1qvBwsyMl.png)



### 12. Start CloudStack Services

Enable and start the required background services (RPC and NFS), then run the CloudStack management setup:

```bash
sudo systemctl start rpcbind
sudo systemctl start nfs-kernel-server
sudo systemctl enable rpcbind
sudo systemctl enable nfs-kernel-server
sudo cloudstack-setup-management
```



### 13. Install KVM Hypervisor

KVM (Kernel-based Virtual Machine) is used as the hypervisor to run virtual machine instances on CloudStack.

**Install KVM and its dependencies:**

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst
```

**Add the active user to the `libvirt` and `kvm` groups** to allow VM management without root:

```bash
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG kvm $(whoami)
```

Apply the group changes in the current session:

```bash
newgrp libvirt
```

**Enable and start the `libvirtd` daemon:**

```bash
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
```

Verify that `libvirtd` is running:

```bash
systemctl status libvirtd
```

![libvirtd status](https://hackmd.io/_uploads/rJ9HRDoJze.png)

Verify KVM is working by listing all virtual machines (should return an empty list on a fresh setup):

```bash
sudo virsh list --all
```



### 14. Install CloudStack Agent & UI

Install the CloudStack Agent — this runs on the hypervisor host and handles communication between the management server and the hypervisor:

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

Verify the service is running correctly:

```bash
systemctl status cloudstack-management
```

![management status](https://hackmd.io/_uploads/B1jwE10JGx.png)

> **Tip:** CloudStack may take a few minutes to fully initialize after the first start. Monitor the logs with:
> ```bash
> sudo tail -f /var/log/cloudstack/management/management-server.log
> ```



### 16. Access the CloudStack UI

Once the management service is running, open a browser and navigate to:

```
http://<management-server-ip>:8080/client/
```

> **Example:** `http://192.168.105.197:8080/client/` or `http://w1660:8080/client/`

**Default login credentials:**

| Field    | Value      |
|---|---|
| Username | `admin`    |
| Password | `password` |

![cloudstack login](https://hackmd.io/_uploads/B1f7210JMg.png)

![cloudstack dashboard 1](https://hackmd.io/_uploads/r1skT1AkMg.png)

![cloudstack dashboard 2](https://hackmd.io/_uploads/rktD6eAyzl.png)

> **Important:** Change the default admin password immediately after first login.



## Zone Configuration

After the CloudStack management server is active, the next step is to configure a **Zone** — the largest organizational unit in CloudStack, representing a single datacenter or physical location.



### Add Zone (Basic) — First Attempt

The first attempt used a **Basic Networking** configuration. The following steps were carried out through the CloudStack Dashboard:

![basic zone 1](https://hackmd.io/_uploads/rJTDpeRkzg.png)

![basic zone 2](https://hackmd.io/_uploads/BJxOTxRJGx.png)

![basic zone 3](https://hackmd.io/_uploads/BJpA2lC1Ge.png)

![basic zone 4](https://hackmd.io/_uploads/Hy8p3eC1fx.png)

![basic zone 5](https://hackmd.io/_uploads/SkO--WRkfl.png)

![basic zone 6](https://hackmd.io/_uploads/SkVI1ZAyMg.png)

![basic zone 7](https://hackmd.io/_uploads/r1xWgZAyze.png)

![basic zone 8](https://hackmd.io/_uploads/rJBfxbCyfx.png)

> **Note:** The Basic Networking zone configuration failed at the _Launch Zone_ step due to an error. The zone was deleted and the setup was retried using the Advanced Networking configuration.



### Add Zone (Advanced)

The second attempt used **Advanced Networking**, which provides more complete network features such as VLAN support, Virtual Router, and Public IP management.

Navigate to **Infrastructure → Zones → Add Zone** and select **Advanced** as the network type.

![add zone advanced](https://hackmd.io/_uploads/SySZ8-RyMe.png)

![zone advanced 1](https://hackmd.io/_uploads/S1UnL-Rkfe.png)

Fill in the zone details including the zone name, DNS addresses, and internal DNS:

![zone advanced 2](https://hackmd.io/_uploads/BkYMOZRkfe.png)

Configure the network settings for the zone:

![zone advanced 3](https://hackmd.io/_uploads/SJ87dZR1zx.png)

![zone advanced 4](https://hackmd.io/_uploads/B1kSO-0yzg.png)

![zone advanced 5](https://hackmd.io/_uploads/B1YCOZRJfl.png)

Configure the pod settings (pod name, reserved system gateway, netmask, and IP range):

![zone advanced 6](https://hackmd.io/_uploads/HyOEFbR1zx.png)



### Troubleshooting

The following issues were encountered during the zone configuration process:

#### 1. Error During Launch Zone

An error occurred when attempting to launch the zone:

![launch zone error](https://hackmd.io/_uploads/r1Xp4fRkGe.png)

**Steps taken — Restart services and verify configuration:**

![troubleshoot 1](https://hackmd.io/_uploads/S182NfAJMx.png)

![troubleshoot 2](https://hackmd.io/_uploads/BkdWLGAJMl.png)



#### 2. Agent Not Activating / Host Cannot Be Added

**Symptom:** The CloudStack agent on the host refuses to activate, or the management server cannot connect to the host.

**Step 1 — Verify time synchronization status.**
CloudStack requires the management server and agent clocks to be in sync. Check Chrony tracking status:

```bash
chronyc tracking
```

![chrony tracking](https://hackmd.io/_uploads/HkS0a201Gg.png)

**Step 2 — Fix hostname resolution** by adding a mapping to `/etc/hosts`:

```bash
sudo vi /etc/hosts
```

Add the following line (replace with your actual IP and hostname):

```
192.168.105.197 w1660
```

![etc hosts](https://hackmd.io/_uploads/SytAp2CyMg.png)

**Step 3 — Check the CloudStack agent logs** for detailed error messages:

```bash
sudo tail -f /var/log/cloudstack/agent/agent.log
```

**Step 4 — Restart the agent service:**

```bash
sudo systemctl restart cloudstack-agent
systemctl status cloudstack-agent
```

**Step 5 — Check for port conflicts** and confirm `libvirtd` is running:

```bash
sudo systemctl status libvirtd
sudo netstat -tlnp | grep 16509
```

> **Known Issue:** In this setup, the agent was intermittently unresponsive despite correct configuration. This appears to be related to timing and host resolution. Restarting both `cloudstack-management` and `cloudstack-agent` in sequence and waiting a few minutes often resolves the issue.



## Known Issues

The following major issues were encountered by the team during installation and configuration:

1. **Basic Networking Zone failed to launch** — The _Launch Zone_ process produced an error, requiring the zone to be deleted and reconfigured using the Advanced Networking type.

2. **CloudStack Agent could not be activated consistently** — After switching to Advanced Networking, the _Add Host_ process continued to fail because the agent could not remain active in a stable state. Multiple troubleshooting attempts were carried out (service restarts, time synchronization verification, `/etc/hosts` configuration), but the issue was not fully resolved.

   ![agent error](https://hackmd.io/_uploads/H1-LwmAkfl.png)

   > This issue is likely caused by libvirt configuration, version compatibility, or network settings that require further investigation.
