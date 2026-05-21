# Single Node Apache Cloudstack Private Cloud Installation Guide

## Program Studi Teknik Komputer, Departemen Teknik Elektro, Universitas Indonesia

![image](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)

## Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Tujuan](#tujuan)
- [Ruang Lingkup](#ruang-lingkup)
- [Kontributor](#kontributor)
- [Panduan Instalasi](#panduan-instalasi)
  - [Persiapan Awal](#persiapan-awal)
  - [Repositori dan Kunci](#repositori-dan-kunci)
  - [Instalasi Database](#instalasi-database)
  - [Konfigurasi NFS](#konfigurasi-nfs)
  - [Pengaturan Firewall](#pengaturan-firewall)
  - [Menjalankan Service CloudStack](#menjalankan-service-cloudstack)
  - [Instalasi KVM](#instalasi-kvm)
  - [Install Agent & UI CloudStack](#install-agent--ui-cloudstack)


## Latar Belakang

Dokumen ini menyajikan panduan instalasi Apache Cloudstack pada satu node untuk penyusunan cloud privat. Materi ini disusun oleh tim yang berasal dari Program Studi Teknik Komputer, Departemen Teknik Elektro, Universitas Indonesia.

## Tujuan

- Menyediakan panduan langkah-demi-langkah instalasi Apache Cloudstack dalam lingkungan satu node.
- Menyajikan struktur laporan yang rapi dan mudah diikuti.
- Mengidentifikasi anggota tim yang berkontribusi dalam pekerjaan ini.

## Ruang Lingkup

Panduan ini mencakup:

- Persiapan lingkungan dan prasyarat perangkat lunak.
- Langkah instalasi Apache Cloudstack di satu node.
- Konfigurasi dasar untuk akses cloud privat.

## Kontributor

Tim penyusun Group 3:

- **Deandro Najwan Ahmad Syahbanna (2306213174)**
- **Muhammad Nadzhif Fikri (2306210102)** 
- **Dwigina Sitti Zahwa (2306250724)** 
- **Muhamad Rey Kafaka Fadlan (2306250573)** 
- **Kharisma Aprilia (2306223244)** 
- **Azka Nabihan (2306250541)** 
- **Ekananda Zhafif Dean (2306264420)** 
- **Dimas Dandossi W P (2206059780)**
- **Abednego Zebua (2306161883)**

## Panduan Instalasi

## Persiapan Awal

1. Perbarui paket terlebih dahulu:

```bash
sudo apt update -y
```

![image](https://hackmd.io/_uploads/r1b5yutC-e.png)

2. Pasang OpenSSH Server:

```bash
sudo apt install openssh-server -y
```

![image](https://hackmd.io/_uploads/HkUiNOKA-e.png)

3. Cek alamat IP pada mesin:

```bash
ip a
```

![image](https://hackmd.io/_uploads/HkG0H_Y0bl.png)

- IP PC digi: `192.168.105.195/24`

4. Konfigurasi Netplan sesuai jaringan Anda.

## Install Chrony

```bash
sudo apt install chrony -y
```

![image](https://hackmd.io/_uploads/H1DU2-kyfx.png)

## Install Java 17 JRE

```bash
sudo apt install openjdk-17-jre -y
```

![image](https://hackmd.io/_uploads/B1Rd2WJyzl.png)

## Install bridge-utils

```bash
sudo apt install bridge-utils -y
```

![image](https://hackmd.io/_uploads/S1FRNZk1Gl.png)

## Add CloudStack Repository

Tambahkan repository CloudStack:

```bash
echo "deb https://download.cloudstack.org/ubuntu focal 4.20" | sudo tee /etc/apt/sources.list.d/cloudstack.list
```

![image](https://hackmd.io/_uploads/r1GxT-1yfe.png)

## Repository Key

Tambahkan kunci repository:

```bash
wget -O - https://download.cloudstack.org/release.asc | sudo tee /etc/apt/trusted.gpg.d/cloudstack.asc
```

![image](https://hackmd.io/_uploads/H1L86Zy1fl.png)

## Install MySQL Server

Instalasi MySQL server dan konfigurasi awal:

```bash
sudo apt install mysql-server -y
```

![image](https://hackmd.io/_uploads/rkiOMMyJzl.png)

Edit konfigurasi MySQL:

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

![image](https://hackmd.io/_uploads/H1XeAW1kMe.png)

Amankan instalasi MySQL:

```bash
sudo mysql_secure_installation
```

![image](https://hackmd.io/_uploads/BJ-LJMyJMx.png)

## Set Up Cloudstack Database

Lanjutkan dengan pembuatan dan konfigurasi database CloudStack.

![image](https://hackmd.io/_uploads/Hkjazf1Jfx.png)

![image](https://hackmd.io/_uploads/BJmhffyyfx.png)

## Konfigurasi NFS untuk Primary dan Secondary Storage

Konfigurasi NFS pada server untuk storage CloudStack.

![image](https://hackmd.io/_uploads/SJ9QXM1kzx.png)

![image](https://hackmd.io/_uploads/ryCA7zJyMl.png)

![image](https://hackmd.io/_uploads/H1-jXG1JMe.png)

![image](https://hackmd.io/_uploads/SJGRgvi1fg.png)

Edit konfigurasi NFS:

```bash
sudo vi /etc/default/nfs-kernel-server
```

![image](https://hackmd.io/_uploads/rJrHVf1kMx.png)

## Pengaturan Firewall

Izinkan akses pada port-port yang diperlukan CloudStack:

```bash
sudo ufw allow from 192.168.105.0/24 to any proto udp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 2049
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 32803
sudo ufw allow from 192.168.105.0/24 to any proto udp port 32769
sudo ufw allow 8080/tcp
```

![image](https://hackmd.io/_uploads/r1x-LDi1Gg.png)

![image](https://hackmd.io/_uploads/S1qvBwsyMl.png)

## Menjalankan Service CloudStack

Mulai layanan pendukung dan setup manajemen CloudStack:

```bash
sudo systemctl start rpcbind
sudo systemctl start nfs-kernel-server
sudo systemctl enable rpcbind
sudo systemctl enable nfs-kernel-server
sudo cloudstack-setup-management
```

## Instalasi KVM

Pasang KVM dan dependensi untuk hypervisor:

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst
```

## Tambahkan Pengguna ke Grup libvirt dan kvm

```bash
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG kvm $(whoami)
newgrp libvirt
```

## Enable dan Start Libvirt

```bash
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
systemctl status libvirtd
```

![image](https://hackmd.io/_uploads/rJ9HRDoJze.png)

## Install CloudStack Agent & UI

```bash
sudo apt install cloudstack-agent -y
sudo apt install cloudstack-ui -y
```

## Start CloudStack Management

```bash
sudo systemctl start cloudstack-management
```

