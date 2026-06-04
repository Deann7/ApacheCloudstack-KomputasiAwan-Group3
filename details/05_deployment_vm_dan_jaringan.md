# Deployment VM, Compute Offering, dan Jaringan Cloud

Dokumen ini menitikberatkan pada proses pembuatan Virtual Machine (VM) oleh pengguna *(user)* atau pengelola cloud *(admin)*, mencakup penyediaan ISO sistem operasi, pendaftaran varian spesifikasi (Compute Offering), hingga langkah pengaturan jaringan virtual agar VM terhubung ke luar (internet).

## 1. Mendaftarkan Citra (Register ISO)

Sebuah VM tidak akan berfungsi tanpa sistem operasi (OS). Pada CloudStack, admin perlu mengunduh citra file biner sistem OS (.ISO) yang tersimpan di dalam *Secondary Storage* agar dapat digunakan saat deployment VM.

1. Buka menu **Images → ISO → Register ISO**.
2. Masukkan URL ISO Langsung yang dapat diunduh (bukan halaman unduh web). Contoh untuk Ubuntu Server 22.04: `https://releases.ubuntu.com/jammy/ubuntu-22.04.5-live-server-amd64.iso`
3. Beri Nama (Contoh: `Ubuntu Server 22.04 LTS`) dan Deskripsi.
4. Klik OK. *Secondary Storage VM (SSVM)* akan bertugas mengunduh file ISO ini dari jaringan publik dan memindahkannya ke sistem repositori file *Secondary Storage* (NFS: `/export/secondary`).

## 2. Membuat Compute Offering (Kapasitas Varian VM)

Di layanan public cloud (seperti AWS atau GCP), Compute Offering dapat diumpamakan seperti tipe instance (misalnya `t2.micro` atau `e2-medium`). Ini mengatur batasan resource vCPU, Kecepatan CPU (Mhz), serta Memori (RAM).

1. Buka **Service Offerings → Compute Offering → Add Compute Offering**.
2. Parameter yang diberikan:
   - **Name:** `big`
   - **Compute Offering Type:** `Fixed Offering`
   - **CPU Cores:** `4`
   - **CPU (MHz):** `1000`
   - **Memory (MB):** `4096`
   - **Dynamic Scaling Enabled:** `On` (Memungkinkan alokasi CPU/RAM diubah dinamis tanpa henti)
   - **Public:** `Yes` (Semua domain pengguna dapat menggunakan offering ini)

Varian `big` ini memberikan sumber daya 4 vCPU dan 4 GB RAM. Penggunaan *offering* bawaan biasanya terlalu kecil sehingga OS terkadang bermasalah.

## 3. Eksekusi Instance (Launch VM)

Dengan resource ISO dan Compute Offering siap sedia, *Instance* VM dapat dibuat.
1. Navigasi ke **Compute → Instances → New Instance**.
2. **Infrastructure:** Pilih Zone, Pod, Cluster, dan Host.
3. **Template/ISO:** Beralih ke tab **ISO → My ISOs**, pilih *Ubuntu Server 22.04*.
4. **Network:** Pilih opsi *Isolated Network*. Jika belum ada, buat baru (`network-group3`). Tipe *Isolated* akan memicu pembuatan sebuah *Virtual Router* yang memegang kendali atas DHCP, DNS internal, Firewall, dan NAT untuk VM-VM di dalamnya.
5. Klik **Launch Instance**. Status progres akan berjalan sambil sistem mem-provisioning volume utama VM ke *Primary Storage* dan mengatur penjadwalan komputasi di Host KVM.

Ketika status menjadi *Running*, klik VM tersebut lalu pilih **View Console** untuk melihat tampilan booting dan menjalankan instalasi Ubuntu Server (seperti instalasi OS pada fisik umumnya). Pastikan untuk menge-klik *Detach ISO* setelah VM meminta me-reboot ulang pada akhir instalasi OS.

## 4. Konfigurasi Jaringan VM (Egress & Port Forwarding)

Sifat dari *Isolated Network* yang dikendalikan oleh *Virtual Router* adalah memiliki aturan "Deny-All" default untuk keamanan, yaitu menutup jalur lalu lintas masuk (*Ingress*) dan keluar (*Egress*). Agar VM dapat mengakses internet, dan dapat diakses masuk (SSH) dari komputer lokal kita, kebijakan Firewall harus dikonfigurasi.

### Mengizinkan Akses Keluar (Egress Rules)
1. Buka menu **Network → Guest Network**, klik `network-group3`.
2. Klik tab **Egress**.
3. Tambahkan profil:
   - **Source CIDR / Destination CIDR:** `0.0.0.0/0` (Untuk menjangkau semua IP di dunia).
   - **Protocol:** `All`
4. VM Anda sekarang akan bisa melakukan `ping 8.8.8.8` dan mengunduh paket dari internet melalui *Virtual Router* yang berperan sebagai NAT Gateway.

### Mengizinkan Akses Masuk lewat Port Forwarding (Ingress Rules)
Untuk meremote ke VM dari luar cloud:
1. Buka **Network → Public IP Addresses**. Anda akan melihat sebuah IP publik (*Source NAT IP*) yang telah diambil oleh layanan CloudStack (Contoh: `192.168.105.221`).
2. Masuk ke alamat IP ini, dan tuju tab **Firewall**. Buka jalan agar paket TCP pada port `22` sampai `23` dari sumber mana saja (`0.0.0.0/0`) dapat tembus ke Firewall *Virtual Router*.
3. Pindah ke tab **Port Forwarding**. Lakukan perutean port 22 ke dalam port 22 internal dari VM.
   - **Private / Public Start & End Port:** `22`
   - **Protocol:** `TCP`
   - Cari dan terapkan pada *Instance VM* Anda.

4. Buka Terminal lokal, Anda kini dapat langsung terhubung dengan SSH ke dalam instance tersebut dengan format:
   ```bash
   ssh <username_vm>@<source-nat-ip>
   ```
   (Contoh: `ssh ubuntu@192.168.105.221`)
