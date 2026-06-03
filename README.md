# Single Node Apache CloudStack Private Cloud Installation Guide

## Program Studi Teknik Komputer, Departemen Teknik Elektro, Universitas Indonesia

![image](https://github.com/user-attachments/assets/7f2482b6-7a3c-49ac-912c-8d22d042740b)

## Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Tujuan](#tujuan)
- [Ruang Lingkup](#ruang-lingkup)
- [Referensi](#referensi)
- [Kontributor](#kontributor)
- [Panduan Instalasi](#panduan-instalasi)
  - [Persiapan Awal](#persiapan-awal)
  - [Install Chrony](#install-chrony)
  - [Install Java 17 JRE](#install-java-17-jre)
  - [Install bridge-utils](#install-bridge-utils)
  - [Add CloudStack Repository](#add-cloudstack-repository)
  - [Repository Key](#repository-key)
  - [Install MySQL Server](#install-mysql-server)
  - [Set Up CloudStack Database](#set-up-cloudstack-database)
  - [Konfigurasi NFS untuk Primary dan Secondary Storage](#konfigurasi-nfs-untuk-primary-dan-secondary-storage)
  - [Pengaturan Firewall](#pengaturan-firewall)
  - [Menjalankan Service CloudStack](#menjalankan-service-cloudstack)
  - [Instalasi KVM](#instalasi-kvm)
  - [Tambahkan Pengguna ke Grup libvirt dan kvm](#tambahkan-pengguna-ke-grup-libvirt-dan-kvm)
  - [Enable dan Start Libvirt](#enable-dan-start-libvirt)
  - [Install CloudStack Agent & UI](#install-cloudstack-agent--ui)
  - [Start CloudStack Management](#start-cloudstack-management)
  - [Akses CloudStack Dashboard](#akses-cloudstack-dashboard)
- [Membuat Zone Baru](#membuat-zone-baru)
  - [Pemilihan Tipe Zone](#pemilihan-tipe-zone)
  - [Pemilihan Tipe Jaringan](#pemilihan-tipe-jaringan)
  - [Mengisi Detail Zone](#mengisi-detail-zone)
  - [Konfigurasi Jaringan](#konfigurasi-jaringan)
  - [Menambahkan Resource](#menambahkan-resource)
  - [Meluncurkan Zone](#meluncurkan-zone)
- [Konfigurasi dan Instalasi VM](#konfigurasi-dan-instalasi-vm)
  - [Mendaftarkan ISO](#mendaftarkan-iso)
  - [Membuat Compute Offering](#membuat-compute-offering)
  - [Membuat Instance Baru](#membuat-instance-baru)
  - [Instalasi Ubuntu Server](#instalasi-ubuntu-server)
- [Konfigurasi Jaringan CloudStack](#konfigurasi-jaringan-cloudstack)
  - [Konfigurasi Egress](#konfigurasi-egress)
  - [Konfigurasi Port Forwarding](#konfigurasi-port-forwarding)

## Latar Belakang

Dokumen ini menyajikan panduan instalasi Apache CloudStack pada satu node untuk penyusunan cloud privat. Apache CloudStack adalah platform open-source untuk mengelola infrastruktur cloud berskala besar. Dalam panduan ini, seluruh komponen (management server, hypervisor, dan storage) dijalankan pada satu mesin fisik. Materi ini disusun oleh tim yang berasal dari Program Studi Teknik Komputer, Departemen Teknik Elektro, Universitas Indonesia.

## Tujuan

- Menyediakan panduan langkah-demi-langkah instalasi Apache CloudStack dalam lingkungan satu node.
- Menyajikan struktur laporan yang rapi dan mudah diikuti.
- Mendokumentasikan proses pembuatan zone, deployment VM, dan konfigurasi jaringan pada CloudStack.

## Ruang Lingkup

Panduan ini mencakup:

- Persiapan lingkungan dan prasyarat perangkat lunak.
- Langkah instalasi Apache CloudStack di satu node.
- Konfigurasi dasar untuk akses cloud privat.
- Pembuatan zone dengan Advanced Networking.
- Deployment virtual machine dan konfigurasi jaringan.

## Referensi

- [Installing Apache CloudStack on Ubuntu 22 — Pratyukt (Hashnode)](https://pratyukt.hashnode.dev/installing-apache-cloudstack-on-ubuntu-22)

## Kontributor

Tim penyusun Group 3:

| Nama                           | NPM        |
| ------------------------------ | ---------- |
| Deandro Najwan Ahmad Syahbanna | 2306213174 |
| Muhammad Nadzhif Fikri         | 2306210102 |
| Dwigina Sitti Zahwa            | 2306250724 |
| Muhamad Rey Kafaka Fadlan      | 2306250573 |
| Kharisma Aprilia               | 2306223244 |
| Azka Nabihan                   | 2306250541 |
| Ekananda Zhafif Dean           | 2306264420 |
| Dimas Dandossi W P             | 2206059780 |
| Abednego Zebua                 | 2306161883 |

---

## Panduan Instalasi

### Persiapan Awal

1. Perbarui daftar paket pada sistem:

```bash
sudo apt update -y
```

![image](https://hackmd.io/_uploads/r1b5yutC-e.png)

2. Pasang OpenSSH Server agar mesin dapat diakses secara remote:

```bash
sudo apt install openssh-server -y
```

![image](https://hackmd.io/_uploads/HkUiNOKA-e.png)

3. Verifikasi alamat IP pada mesin:

```bash
ip a
```

![image](https://hackmd.io/_uploads/HkG0H_Y0bl.png)

- Contoh IP yang diperoleh: `192.168.105.197/24`

4. Konfigurasi Netplan sesuai dengan topologi jaringan yang digunakan. Pastikan interface jaringan terhubung ke subnet yang sesuai.

### Install Chrony

Chrony digunakan untuk sinkronisasi waktu (NTP). Sinkronisasi waktu yang akurat penting agar komponen CloudStack dapat berkomunikasi dengan benar.

```bash
sudo apt install chrony -y
```

![image](https://hackmd.io/_uploads/H1DU2-kyfx.png)

### Install Java 17 JRE

Apache CloudStack membutuhkan Java Runtime Environment versi 17 untuk menjalankan management server.

```bash
sudo apt install openjdk-17-jre -y
```

![image](https://hackmd.io/_uploads/B1Rd2WJyzl.png)

### Install bridge-utils

Paket `bridge-utils` diperlukan untuk membuat dan mengelola network bridge pada hypervisor.

```bash
sudo apt install bridge-utils -y
```

![image](https://hackmd.io/_uploads/S1FRNZk1Gl.png)

### Add CloudStack Repository

Tambahkan repository resmi Apache CloudStack versi 4.20 ke daftar sumber paket:

```bash
echo "deb https://download.cloudstack.org/ubuntu focal 4.20" | sudo tee /etc/apt/sources.list.d/cloudstack.list
```

![image](https://hackmd.io/_uploads/r1GxT-1yfe.png)

### Repository Key

Tambahkan kunci GPG repository untuk verifikasi keaslian paket:

```bash
wget -O - https://download.cloudstack.org/release.asc | sudo tee /etc/apt/trusted.gpg.d/cloudstack.asc
```

![image](https://hackmd.io/_uploads/H1L86Zy1fl.png)

### Install MySQL Server

CloudStack menggunakan MySQL sebagai backend database. Lakukan instalasi dan konfigurasi awal:

1. Instal MySQL Server:

```bash
sudo apt install mysql-server -y
```

![image](https://hackmd.io/_uploads/rkiOMMyJzl.png)

2. Edit file konfigurasi MySQL untuk menyesuaikan parameter yang dibutuhkan CloudStack:

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

![image](https://hackmd.io/_uploads/H1XeAW1kMe.png)

3. Amankan instalasi MySQL dengan menjalankan script keamanan bawaan:

```bash
sudo mysql_secure_installation
```

![image](https://hackmd.io/_uploads/BJ-LJMyJMx.png)

### Set Up CloudStack Database

Setelah MySQL terpasang, lakukan setup database CloudStack menggunakan tools bawaan `cloudstack-setup-databases`:

```bash
sudo cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:<password> -i <management-server-ip>
```

![image](https://hackmd.io/_uploads/Hkjazf1Jfx.png)

![image](https://hackmd.io/_uploads/BJmhffyyfx.png)

### Konfigurasi NFS untuk Primary dan Secondary Storage

NFS (Network File System) digunakan sebagai primary storage dan secondary storage oleh CloudStack. Buat direktori yang diperlukan dan konfigurasi ekspor NFS:

1. Buat direktori untuk primary dan secondary storage:

```bash
sudo mkdir -p /export/primary
sudo mkdir -p /export/secondary
```

2. Konfigurasi file `/etc/exports` untuk mengizinkan akses NFS:

![image](https://hackmd.io/_uploads/SJ9QXM1kzx.png)

![image](https://hackmd.io/_uploads/ryCA7zJyMl.png)

![image](https://hackmd.io/_uploads/H1-jXG1JMe.png)

![image](https://hackmd.io/_uploads/SJGRgvi1fg.png)

3. Edit konfigurasi NFS kernel server:

```bash
sudo vi /etc/default/nfs-kernel-server
```

![image](https://hackmd.io/_uploads/rJrHVf1kMx.png)

### Pengaturan Firewall

Izinkan akses pada port-port yang diperlukan oleh CloudStack melalui UFW. Port-port berikut digunakan untuk komunikasi NFS dan akses management server:

| Port  | Protokol | Fungsi                   |
| ----- | -------- | ------------------------ |
| 111   | TCP/UDP  | RPC Portmapper           |
| 2049  | TCP      | NFS                      |
| 32803 | TCP      | NFS mountd               |
| 32769 | UDP      | NFS lockd                |
| 8080  | TCP      | CloudStack Management UI |

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

### Menjalankan Service CloudStack

Aktifkan dan jalankan layanan pendukung (RPC dan NFS), kemudian jalankan setup management CloudStack:

```bash
sudo systemctl start rpcbind
sudo systemctl start nfs-kernel-server
sudo systemctl enable rpcbind
sudo systemctl enable nfs-kernel-server
sudo cloudstack-setup-management
```

### Instalasi KVM

KVM (Kernel-based Virtual Machine) digunakan sebagai hypervisor untuk menjalankan instance virtual machine pada CloudStack.

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst
```

### Tambahkan Pengguna ke Grup libvirt dan kvm

Agar pengguna aktif memiliki akses untuk mengelola virtual machine, tambahkan ke grup `libvirt` dan `kvm`:

```bash
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG kvm $(whoami)
newgrp libvirt
```

### Enable dan Start Libvirt

Aktifkan dan jalankan daemon libvirt, kemudian verifikasi statusnya:

```bash
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
systemctl status libvirtd
```

![image](https://hackmd.io/_uploads/rJ9HRDoJze.png)

### Install CloudStack Agent & UI

Instal agent (untuk komunikasi antara management server dan hypervisor) serta UI (antarmuka web):

```bash
sudo apt install cloudstack-agent -y
sudo apt install cloudstack-ui -y
```

### Start CloudStack Management

Jalankan service CloudStack management dan verifikasi statusnya:

```bash
sudo systemctl start cloudstack-management
```

Untuk memastikan service berjalan dengan benar:

```bash
systemctl status cloudstack-management
```

![image](https://hackmd.io/_uploads/B1jwE10JGx.png)

### Akses CloudStack Dashboard

Setelah service berjalan, CloudStack Dashboard dapat diakses melalui browser pada alamat berikut:

```
http://<management-server-ip>:8080/client/
```

Gunakan kredensial default untuk login pertama kali:

- **Username:** `admin`
- **Password:** `password`

![image](https://hackmd.io/_uploads/B1f7210JMg.png)

![image](https://hackmd.io/_uploads/r1skT1AkMg.png)

![image](https://hackmd.io/_uploads/rktD6eAyzl.png)

---

## Membuat Zone Baru

Setelah management server aktif dan dashboard dapat diakses, langkah berikutnya adalah membuat **Zone**. Zone merupakan unit deployment terbesar dalam arsitektur CloudStack, merepresentasikan satu lokasi data center.

### Pemilihan Tipe Zone

Pada saat pembuatan zone, terdapat dua pilihan tipe:

- **Core**: Zone standar untuk menjalankan workload komputasi utama. Cocok untuk lingkungan produksi dengan fitur lengkap.
- **Edge**: Zone ringan yang umumnya digunakan untuk edge computing (misalnya IoT, CDN, atau lokasi remote) dengan resource dan layanan yang lebih terbatas.

Untuk keperluan panduan ini, pilih **Core** agar seluruh fitur CloudStack tersedia.

### Pemilihan Tipe Jaringan

Selanjutnya, tentukan tipe jaringan yang akan digunakan:

- **Basic**: Jaringan flat tanpa VLAN. Setiap VM mendapatkan IP secara langsung. Konfigurasi lebih sederhana.
- **Advanced**: Mendukung VLAN, virtual router, dan multiple guest network. Memberikan fleksibilitas dan isolasi jaringan yang lebih baik.

### Mengisi Detail Zone

Isi informasi dasar zone pada form yang tersedia:

| Field          | Contoh Nilai      | Keterangan                                    |
| -------------- | ----------------- | --------------------------------------------- |
| Name           | `Zone-Group3`     | Nama deskriptif untuk zone                    |
| IPv4 DNS 1     | `8.8.8.8`         | DNS publik untuk resolusi nama domain         |
| Internal DNS 1 | `192.168.105.197` | IP mesin host, digunakan sebagai DNS internal |
| Hypervisor     | `KVM`             | Tipe hypervisor yang digunakan                |

### Konfigurasi Jaringan

#### Physical Network

Gunakan konfigurasi default untuk physical network, lalu lanjutkan ke langkah berikutnya.

#### Public Traffic

Konfigurasi rentang IP untuk akses publik VM:

| Field    | Contoh Nilai      | Keterangan                          |
| -------- | ----------------- | ----------------------------------- |
| Gateway  | `192.168.105.1`   | Gateway jaringan                    |
| Netmask  | `255.255.255.0`   | Subnet mask                         |
| Start IP | `192.168.105.221` | IP awal (pastikan belum digunakan)  |
| End IP   | `192.168.105.225` | IP akhir (pastikan belum digunakan) |

IP pada rentang ini akan dialokasikan untuk akses publik ke virtual machine.

#### Pod

Setiap zone memerlukan minimal satu **Pod**, yaitu unit yang mengelompokkan cluster dan host.

| Field    | Contoh Nilai      | Keterangan                   |
| -------- | ----------------- | ---------------------------- |
| Name     | `Pod-Group3`      | Nama deskriptif untuk pod    |
| Gateway  | `192.168.105.1`   | Gateway jaringan             |
| Netmask  | `255.255.255.0`   | Subnet mask                  |
| Start IP | `192.168.105.226` | IP awal untuk manajemen pod  |
| End IP   | `192.168.105.230` | IP akhir untuk manajemen pod |

#### Guest Traffic

Tentukan rentang VLAN/VNI untuk isolasi traffic jaringan guest:

- **VLAN/VNI Range:** `3300 - 3339`

Pastikan VLAN yang dipilih telah dikonfigurasi pada switch atau router jaringan.

### Menambahkan Resource

#### Cluster

Cluster berfungsi untuk mengelompokkan host hypervisor yang berbagi konfigurasi storage dan jaringan.

- **Cluster Name:** `Cluster-Group3`

#### Host

Masukkan informasi mesin host yang akan menjalankan VM:

| Field    | Nilai             | Keterangan               |
| -------- | ----------------- | ------------------------ |
| Hostname | `192.168.105.197` | IP mesin host            |
| Username | `root`            | Username akses host      |
| Password | `******`          | Password root mesin host |

#### Primary Storage

Primary storage menyimpan disk volume dari VM yang sedang berjalan.

| Field    | Nilai             | Keterangan              |
| -------- | ----------------- | ----------------------- |
| Name     | `PrimStor-Group3` | Nama primary storage    |
| Scope    | `Zone`            | Cakupan storage         |
| Protocol | `NFS`             | Protokol yang digunakan |
| Server   | `192.168.105.197` | IP NFS server           |
| Path     | `/export/primary` | Direktori NFS primary   |
| Provider | `DefaultPrimary`  | Provider storage        |

#### Secondary Storage

Secondary storage digunakan untuk menyimpan template, ISO, dan snapshot.

| Field    | Nilai               | Keterangan              |
| -------- | ------------------- | ----------------------- |
| Provider | `NFS`               | Provider storage        |
| Name     | `SecStor-Group3`    | Nama secondary storage  |
| Server   | `192.168.105.197`   | IP NFS server           |
| Path     | `/export/secondary` | Direktori NFS secondary |

### Meluncurkan Zone

Klik **Launch Zone** untuk memulai proses pembuatan zone dengan seluruh konfigurasi yang telah diisi. Proses ini memerlukan beberapa waktu.

Jika zone berhasil dibuat, akan muncul notifikasi konfirmasi. Apabila terjadi error, gunakan tombol **Fix Issues** untuk diarahkan ke halaman konfigurasi yang perlu diperbaiki.

> **Tips:** Error yang umum terjadi biasanya terkait konflik IP address rentang IP yang sudah digunakan oleh perangkat lain dalam jaringan. Coba ganti dengan rentang IP yang belum terpakai dan ulangi proses.

Untuk Contoh Konfigurasi yang kami lakukan dapat dilihat dalam gambar di bawah.

![image](https://hackmd.io/_uploads/rJTDpeRkzg.png)

![image](https://hackmd.io/_uploads/BJxOTxRJGx.png)

![image](https://hackmd.io/_uploads/BJpA2lC1Ge.png)

![image](https://hackmd.io/_uploads/Hy8p3eC1fx.png)

![image](https://hackmd.io/_uploads/SkO--WRkfl.png)

![image](https://hackmd.io/_uploads/SkVI1ZAyMg.png)

![image](https://hackmd.io/_uploads/r1xWgZAyze.png)

![image](https://hackmd.io/_uploads/rJBfxbCyfx.png)

---

## Konfigurasi dan Instalasi VM

Setelah zone aktif, langkah selanjutnya adalah menyiapkan resource dan membuat virtual machine.

### Mendaftarkan ISO

Untuk menginstal sistem operasi pada VM, diperlukan file ISO. Cari direct link ISO dari internet (umumnya berakhiran `.iso`). Contoh untuk Ubuntu Server 22.04:

```
https://releases.ubuntu.com/jammy/ubuntu-22.04.5-live-server-amd64.iso
```

Langkah pendaftaran ISO:

1. Dari sidebar, navigasi ke **Images → ISO → Register ISO**.
2. Isi detail yang diperlukan:

| Field       | Contoh Nilai                                                             |
| ----------- | ------------------------------------------------------------------------ |
| URL         | `https://releases.ubuntu.com/jammy/ubuntu-22.04.5-live-server-amd64.iso` |
| Name        | `Ubuntu Server 22.04`                                                    |
| Description | `Ubuntu Server 22.04 LTS`                                                |

3. Biarkan pengaturan lainnya pada nilai default.

Proses unduh ISO akan berjalan di background dan membutuhkan waktu tergantung kecepatan koneksi internet. Status dapat dipantau melalui halaman detail ISO pada tab **Zone**.

### Membuat Compute Offering

Compute Offering mendefinisikan alokasi CPU, memori, dan resource komputasi lainnya untuk VM.

1. Dari sidebar, navigasi ke **Service Offerings → Compute Offering → Add Compute Offering**.
2. Isi konfigurasi berikut:

| Field                   | Nilai            |
| ----------------------- | ---------------- |
| Name                    | `big`            |
| Description             | `big`            |
| Compute Offering Type   | `Fixed Offering` |
| CPU Cores               | `4`              |
| CPU (MHz)               | `1000`           |
| Memory (MB)             | `4096`           |
| Dynamic Scaling Enabled | `On`             |
| GPU                     | `None`           |
| Public                  | `Yes`            |

Konfigurasi ini menghasilkan offering dengan 4 core CPU dan 4 GB RAM. Disarankan untuk membuat offering baru karena offering default hanya memiliki 1 core CPU yang dapat menyebabkan performa VM kurang optimal.

### Membuat Instance Baru

1. Dari sidebar, navigasi ke **Compute → Instances → New Instance**.

2. **Pilih Deployment Infrastructure:**
   - Zone, Pod, Cluster, dan Host sesuai yang telah dibuat sebelumnya.

3. **Pilih Template/ISO:**
   - Klik tab **ISO**, pilih **My ISOs**, lalu pilih ISO yang sudah didaftarkan.

4. **Konfigurasi Network:**

   Jika belum memiliki jaringan, buat **Isolated Network** baru:

   | Field | Nilai                  |
   | ----- | ---------------------- |
   | Name  | `network-group3`       |
   | Zone  | Zone yang telah dibuat |

   Isolated network menyediakan jaringan guest yang terisolasi dengan akses internet outbound.

5. **Detail Instance:**

   | Field             | Nilai                  |
   | ----------------- | ---------------------- |
   | Name              | Nama instance VM       |
   | Keyboard Language | `Standard US Keyboard` |

6. Klik **Launch Instance** untuk memulai provisioning VM.

### Instalasi Ubuntu Server

Setelah instance diluncurkan, akses console VM melalui dashboard untuk melakukan instalasi Ubuntu Server. Ikuti wizard instalasi yang tampil di layar.

Setelah instalasi selesai dan diminta untuk reboot, **detach ISO** dari halaman Instance agar VM boot dari disk yang sudah terinstal.

> Setelah instalasi selesai, VM dapat diakses melalui console. Namun, koneksi internet belum tersedia karena konfigurasi jaringan belum dilakukan.

---

## Konfigurasi Jaringan CloudStack

Jika menggunakan **Isolated Network**, konfigurasi tambahan diperlukan agar VM dapat mengakses internet dan dapat diakses melalui SSH dari luar.

### Konfigurasi Egress

Agar VM dapat mengakses internet (outbound traffic), lakukan langkah berikut:

1. Navigasi ke **Network → Guest Network**.
2. Klik nama network yang digunakan (misalnya `network-group3`).
3. Buka tab **Egress**.
4. Tambahkan rule dengan parameter:
   - **Source CIDR:** `0.0.0.0/0`
   - **Destination CIDR:** `0.0.0.0/0`
   - **Protocol:** `All`
5. Klik **Add**.

Setelah rule diterapkan, perubahan akan terlihat secara langsung pada console VM. Verifikasi koneksi dengan perintah:

```bash
ping 8.8.8.8
```

> **Alternatif akses:** Selain port forwarding, dapat juga menginstal VPN seperti Tailscale pada VM untuk akses SSH dari mana saja.

### Konfigurasi Port Forwarding

Untuk mengakses VM melalui SSH dari luar, diperlukan konfigurasi firewall dan port forwarding pada Source NAT IP.

#### 1. Pengaturan Firewall

1. Navigasi ke **Network → Public IP Addresses**.
2. Klik pada alamat **Source NAT**.
3. Buka tab **Firewall**.
4. Tambahkan rule:
   - **Source CIDR:** `0.0.0.0/0`
   - **Start Port:** `22`
   - **End Port:** `23`
   - **Protocol:** `TCP`

#### 2. Pengaturan Port Forwarding

1. Buka tab **Port Forwarding**.
2. Tambahkan rule dengan parameter:
   - **Private Start Port:** `22`
   - **Private End Port:** `23`
   - **Public Start Port:** `22`
   - **Public End Port:** `23`
   - **Protocol:** `TCP`
3. Klik **Add**, lalu pilih VM tujuan dan konfirmasi.

Setelah konfigurasi selesai, VM dapat diakses melalui SSH menggunakan Source NAT IP address dari komputer lain yang berada dalam jaringan yang sama:

```bash
ssh <username>@<source-nat-ip>
```
