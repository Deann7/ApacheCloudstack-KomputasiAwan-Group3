# Persiapan dan Prasyarat Instalasi Apache CloudStack

Dokumen ini menjabarkan secara detail langkah-langkah persiapan awal dan instalasi prasyarat perangkat lunak yang dibutuhkan sebelum melakukan instalasi komponen utama Apache CloudStack. Persiapan yang matang pada tahap ini akan mencegah kegagalan komunikasi antar layanan pada tahap-tahap berikutnya.

## 1. Pembaruan Sistem dan Akses Remote

Langkah pertama yang harus dilakukan adalah memastikan seluruh paket sistem operasi berada pada versi terbaru dan sistem dapat diakses secara remote. Akses remote sangat krusial karena server CloudStack biasanya berjalan di environment *headless* (tanpa monitor).

```bash
sudo apt update -y
```

![Update Sistem](https://hackmd.io/_uploads/r1b5yutC-e.png)

Selanjutnya, pasang OpenSSH Server agar mesin dapat dikelola jarak jauh melalui protokol SSH.

```bash
sudo apt install openssh-server -y
```

![Install SSH](https://hackmd.io/_uploads/HkUiNOKA-e.png)

Pastikan untuk memverifikasi alamat IP yang didapat oleh mesin host. IP ini akan menjadi IP Manajemen dan IP Host utama.

```bash
ip a
```

![Verifikasi IP](https://hackmd.io/_uploads/HkG0H_Y0bl.png)

*Contoh IP yang digunakan pada lingkungan ini adalah: `192.168.105.197/24`.* Konfigurasi statis sangat disarankan, yang dapat diatur melalui Netplan pada sistem operasi Ubuntu.

## 2. Instalasi dan Konfigurasi Chrony (NTP)

Sinkronisasi waktu antar komponen dalam arsitektur Cloud (Management Server, Hypervisor, Storage) adalah hal yang wajib. Jika waktu antar server berbeda (drift), sertifikat keamanan dapat ditolak dan proses sinkronisasi log serta eksekusi tugas (job) dapat gagal. Oleh karena itu, kita mengandalkan **Chrony** sebagai klien Network Time Protocol (NTP).

```bash
sudo apt install chrony -y
```

![Install Chrony](https://hackmd.io/_uploads/H1DU2-kyfx.png)

Pastikan layanan Chrony sudah aktif dan berjalan normal:
```bash
sudo systemctl enable chrony
sudo systemctl start chrony
```

## 3. Pemasangan Java 17 JRE

Layanan `cloudstack-management` (Management Server) dibangun di atas platform Java. Khusus untuk CloudStack versi 4.20 ke atas, dibutuhkan minimal Java Runtime Environment (JRE) versi 17. Penggunaan versi Java yang tidak sesuai akan membuat proses inisialisasi management server terhenti dengan pesan error `UnsupportedClassVersionError`.

```bash
sudo apt install openjdk-17-jre -y
```

![Install Java](https://hackmd.io/_uploads/B1Rd2WJyzl.png)

## 4. Pemasangan Bridge-utils

Sebagian besar node di cloud privat berbasis KVM akan membutuhkan virtual switch atau bridge untuk menghubungkan mesin virtual (VM) dengan jaringan fisik (NIC host). Paket `bridge-utils` menyediakan utilitas seperti `brctl` untuk membuat dan mengelola jembatan jaringan (network bridge) pada Linux.

```bash
sudo apt install bridge-utils -y
```

![Install Bridge Utils](https://hackmd.io/_uploads/S1FRNZk1Gl.png)

## 5. Penambahan Repositori Apache CloudStack

Paket instalasi CloudStack tidak tersedia pada repositori bawaan Ubuntu. Kita harus menambahkan repositori resmi Apache CloudStack (dalam panduan ini menggunakan versi 4.20) ke dalam daftar sumber paket (source list) APT.

```bash
echo "deb https://download.cloudstack.org/ubuntu focal 4.20" | sudo tee /etc/apt/sources.list.d/cloudstack.list
```

![Add CloudStack Repository](https://hackmd.io/_uploads/r1GxT-1yfe.png)

Untuk menjaga integritas paket dan memastikan paket yang diunduh benar-benar otentik dari rilis resmi, kita harus mengimpor kunci GPG dari repositori tersebut.

```bash
wget -O - https://download.cloudstack.org/release.asc | sudo tee /etc/apt/trusted.gpg.d/cloudstack.asc
```

![Add GPG Key](https://hackmd.io/_uploads/H1L86Zy1fl.png)

Setelah kunci diimpor, lakukan `sudo apt update -y` kembali untuk menyegarkan daftar paket. Dengan selesainya tahap persiapan ini, host telah siap untuk diinstal *Database*, *Storage*, dan *Hypervisor*.
