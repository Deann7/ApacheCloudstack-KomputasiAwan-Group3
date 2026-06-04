# Database dan Storage Node CloudStack

Dokumen ini menjelaskan tahapan konfigurasi arsitektur back-end penting dari Apache CloudStack, yakni sistem basis data (database) yang mengelola status state seluruh cloud, serta sistem penyimpanan network (Network File System) untuk keperluan penyimpanan volume instance dan aset data center.

## 1. Pemasangan MySQL Server

CloudStack sangat bergantung pada MySQL sebagai basis data utama manajemen state (metadata cloud, data pengguna, data jaringan, dll). Kegagalan pada database ini dapat menghentikan seluruh layanan manajemen CloudStack.

```bash
sudo apt install mysql-server -y
```

![Instalasi MySQL](https://hackmd.io/_uploads/rkiOMMyJzl.png)

Setelah instalasi selesai, beberapa pengaturan parameter spesifik CloudStack harus ditambahkan pada file konfigurasi `mysqld.cnf`. Parameter ini sangat penting agar CloudStack tidak mengalami isu batasan koneksi dan isu performa pada database.

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
Pastikan Anda mengubah parameter `bind-address` menjadi `0.0.0.0` jika Anda akan mengakses database dari server manajemen eksternal, atau biarkan default/`localhost` jika dalam mode Single Node. Tambahkan parameter *innodb*, *max_connections*, dan lain-lain sesuai spesifikasi CloudStack.

![Edit Konfigurasi MySQL](https://hackmd.io/_uploads/H1XeAW1kMe.png)

### Pengamanan Database
Gunakan *built-in script* `mysql_secure_installation` untuk mengubah tingkat keamanan instalasi default, termasuk mendefinisikan *root password*, mematikan *remote root login*, menghapus *test databases*, dan mencabut izin anonimus.

```bash
sudo mysql_secure_installation
```

![MySQL Secure Installation](https://hackmd.io/_uploads/BJ-LJMyJMx.png)

## 2. Inisialisasi Database CloudStack

Apache CloudStack menyediakan alat CLI `cloudstack-setup-databases` untuk mempermudah inisialisasi awal. Perintah ini akan masuk sebagai *root MySQL*, membuat user bernama `cloud` dengan password `cloud`, dan menanam schema database dasar di host lokal.

```bash
sudo cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:<password_mysql> -i 192.168.105.197
```

![Setup DB 1](https://hackmd.io/_uploads/Hkjazf1Jfx.png)
![Setup DB 2](https://hackmd.io/_uploads/BJmhffyyfx.png)

Jika sukses, layanan CloudStack sudah memiliki kerangka database dan dapat menyimpan segala jenis operasi.

## 3. Konfigurasi NFS (Network File System)

Dalam arsitektur CloudStack, memisahkan *Primary Storage* (untuk VM yang berjalan) dan *Secondary Storage* (untuk ISO, Snapshot, Template) merupakan praktik terbaik, meskipun secara fisik masih di mesin yang sama dalam implementasi Single Node ini.

### Pembuatan Direktori Penyimpanan

```bash
sudo mkdir -p /export/primary
sudo mkdir -p /export/secondary
```

### Konfigurasi Izin Ekspor (/etc/exports)

Kita perlu memberikan hak akses *(share)* pada folder-folder tadi agar dapat dipanggil (mount) oleh layanan Hypervisor secara jarak jauh. Hal ini diatur di `/etc/exports`. Parameter yang digunakan biasanya mencakup `rw` (read-write), `async` (untuk performa disk lebih tinggi, namun ada risiko data jika listrik padam tiba-tiba), `no_root_squash` (agar KVM/CloudStack dapat memodifikasi file sebagai root), dan `no_subtree_check`.

![Konfigurasi NFS Exports 1](https://hackmd.io/_uploads/SJ9QXM1kzx.png)
![Konfigurasi NFS Exports 2](https://hackmd.io/_uploads/ryCA7zJyMl.png)
![Konfigurasi NFS Exports 3](https://hackmd.io/_uploads/H1-jXG1JMe.png)
![Konfigurasi NFS Exports 4](https://hackmd.io/_uploads/SJGRgvi1fg.png)

### Penyesuaian NFS Kernel Server

Terdapat juga opsi file `/etc/default/nfs-kernel-server` yang dapat diatur untuk mengatur seberapa banyak *thread/worker* NFS yang berjalan di *background*.

```bash
sudo vi /etc/default/nfs-kernel-server
```

![NFS Kernel Server](https://hackmd.io/_uploads/rJrHVf1kMx.png)

Layanan `rpcbind` dan `nfs-kernel-server` kemudian dapat dimulai dan dijadikan autostart, sehingga seluruh sumber daya penyimpanan utama dapat disajikan dengan stabil.

```bash
sudo systemctl start rpcbind
sudo systemctl start nfs-kernel-server
sudo systemctl enable rpcbind
sudo systemctl enable nfs-kernel-server
```
