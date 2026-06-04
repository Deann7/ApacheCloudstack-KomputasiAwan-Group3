# Management Server dan Hypervisor (KVM)

Dokumen ini memberikan rincian untuk instalasi dan konfigurasi peran Management Server (sebagai orkestrator dari CloudStack) dan peran Hypervisor (tempat komputasi VM dieksekusi). Dalam topologi node tunggal (Single Node), kedua peran ini bekerja berdampingan di host yang sama, sehingga dibutuhkan pengaturan tambahan untuk keamanan jaringan (Firewall) agar komunikasi tidak terblokir.

## 1. Pengaturan Firewall (UFW)

Sebelum Management Server dan Hypervisor saling "berbicara" atau mengakses *Storage*, porta (port) jaringan spesifik harus terbuka. CloudStack secara intens menggunakan layanan RPC dan NFS, serta menggunakan port manajemen HTTP standar `8080`.

| Port  | Protokol | Fungsi Terkait |
| ----- | -------- | ------------------------ |
| 111   | TCP/UDP  | RPC Portmapper (Digunakan saat pemetaan lokasi layanan NFS) |
| 2049  | TCP      | NFS Protocol utama |
| 32803 | TCP      | NFS mountd |
| 32769 | UDP      | NFS lockd |
| 8080  | TCP      | CloudStack Management UI (Antarmuka Administrator) |

Menerapkan konfigurasi di Uncomplicated Firewall (UFW):

```bash
sudo ufw allow from 192.168.105.0/24 to any proto udp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 111
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 2049
sudo ufw allow from 192.168.105.0/24 to any proto tcp port 32803
sudo ufw allow from 192.168.105.0/24 to any proto udp port 32769
sudo ufw allow 8080/tcp
```

![UFW Config 1](https://hackmd.io/_uploads/r1x-LDi1Gg.png)
![UFW Config 2](https://hackmd.io/_uploads/S1qvBwsyMl.png)

## 2. Instalasi KVM dan Libvirt

KVM (Kernel-based Virtual Machine) merupakan hypervisor open-source andalan yang di-support penuh oleh Apache CloudStack. Instalasinya perlu dibarengi dengan `libvirt`, yaitu lapisan *middleware* yang mendikte hypervisor tentang bagaimana VM dibuat, diaturnya jaringan, dan dikelolanya penyimpanannya.

```bash
sudo apt update
sudo apt install -y qemu-kvm libvirt-clients libvirt-daemon-system bridge-utils virtinst
```

### Penambahan Hak Akses Pengguna

Untuk menghindari *permission denied* saat memanggil layanan hypervisor, pengguna yang sedang aktif wajib didaftarkan ke grup `libvirt` dan `kvm`.

```bash
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG kvm $(whoami)
newgrp libvirt
```

### Menjalankan Layanan Libvirt

Setelah paket berhasil dipasang, layanan libvirt harus diinisiasi dan di-enable agar berjalan otomatis saat *booting*.

```bash
sudo systemctl enable --now libvirtd
sudo systemctl start libvirtd
systemctl status libvirtd
```

![Libvirt Status](https://hackmd.io/_uploads/rJ9HRDoJze.png)

## 3. Pemasangan Management Server, Agent, dan UI

Pemasangan layanan inti Management dan interfacenya memerlukan paket-paket CloudStack. `cloudstack-agent` dibutuhkan sebagai "kaki tangan" Management Server yang terinstalasi di *Host* untuk menyampaikan perintah ke hypervisor KVM. UI dibutuhkan agar admin dapat melakukan setup secara *Graphical*.

```bash
sudo apt install cloudstack-agent -y
sudo apt install cloudstack-ui -y
```

### Inisialisasi CloudStack Management

Apabila sistem operasional, database, dan *storage* telah terkonfigurasi, kita dapat memicu setup awal untuk Management Server dan menghidupkan layanannya.

```bash
sudo cloudstack-setup-management
sudo systemctl start cloudstack-management
systemctl status cloudstack-management
```

Layanan ini berbasis Tomcat (di *background*) sehingga membutuhkan waktu sekitar 1-3 menit sampai antarmuka *Dashboard* dapat diakses dengan respons 200 (OK).

![Management Status](https://hackmd.io/_uploads/B1jwE10JGx.png)

## 4. Akses Dashboard

Kunjungi tautan berikut di peramban web: `http://<IP-SERVER>:8080/client/`

Kredensial login bawaan (default) saat pertama kali masuk:
- **Username:** `admin`
- **Password:** `password`

![CloudStack Login](https://hackmd.io/_uploads/B1f7210JMg.png)
![CloudStack Dashboard](https://hackmd.io/_uploads/r1skT1AkMg.png)
![CloudStack UI Home](https://hackmd.io/_uploads/rktD6eAyzl.png)

Langkah ini menutup persiapan dasar dan sistem telah siap memasuki tahap pembangunan topologi virtual, yaitu Zone Configuration.
