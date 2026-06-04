# Konfigurasi Zone dan Jaringan

Dokumen ini menyajikan penjabaran terkait konfigurasi infrastruktur virtual CloudStack. Setelah dasbor aktif, "Zone" adalah hierarki terbesar yang harus diinisiasi pertama kali, mengacu pada satu buah pusat data (data center) secara penuh.

## 1. Konsep Hierarki: Zone, Pod, Cluster, dan Host

CloudStack memiliki terminologi spesifik yang memisahkan infrastruktur menjadi empat lapisan logis:
- **Zone:** Level tertinggi, dapat disamakan dengan Data Center fisik. Sebuah Zone memiliki akses ke Secondary Storage dan berisi beberapa Pod.
- **Pod:** Grup dari jaringan fisik (L2 Network), yang menaungi satu atau beberapa Cluster.
- **Cluster:** Kumpulan dari beberapa server fisik (Host) yang memiliki tipe Hypervisor yang sama (misal, semua KVM atau VMware) dan berbagi *Primary Storage* yang serupa.
- **Host:** Mesin virtualisasi tunggal (Hypervisor Node) yang mengeksekusi Guest VM.

## 2. Pilihan Tipe Zone dan Tipe Jaringan

Saat memulai pembuatan Zone, akan disuguhkan opsi Tipe Zone dan Tipe Jaringan:
- **Core vs Edge**: *Core* memberikan seluruh kapabilitas komputasi data center, sedangkan *Edge* digunakan untuk footprint kecil dengan layanan terbatas. Kita menggunakan **Core**.
- **Basic vs Advanced Networking**:
  - *Basic Networking*: Model sederhana dimana VM mendapat IP satu subnet rata tanpa VLAN atau perutean *Virtual Router*.
  - *Advanced Networking*: Memberikan kapabilitas jaringan virtual lanjutan (VLAN, VXLAN, isolasi jaringan per-tenant, Firewall, Load Balancer, Port Forwarding) dengan membangun sistem Virtual Router secara dinamis.

Pada laporan ini, tim mencatat *Basic Networking* gagal, sehingga percobaan beralih ke fitur andalan CloudStack yaitu **Advanced Networking**.

## 3. Konfigurasi Zone Lanjutan (Advanced)

Langkah demi langkah di dasbor CloudStack untuk konfigurasi *Advanced Zone*:

### A. Konfigurasi Awal Zone
Mendefinisikan identitas Zone serta parameter DNS utama.
- **Name:** `Zone-Group3`
- **IPv4 DNS 1:** `8.8.8.8` (untuk resolusi nama domain eksternal)
- **Internal DNS 1:** `192.168.105.197` (IP Host Management)
- **Hypervisor:** KVM

![Zone Name Config](https://hackmd.io/_uploads/SySZ8-RyMe.png)

### B. Konfigurasi Physical dan Public Network
Langkah selanjutnya mengatur porsi *Physical Network* dan *Public Traffic*. IP pada ruang *Public Traffic* adalah rentang IP yang dikendalikan oleh CloudStack dan diperuntukkan bagi *Source NAT*, akses web (Port Forwarding), maupun *Static NAT* di dalam Cloud.

- **Gateway:** `192.168.105.1`
- **Netmask:** `255.255.255.0`
- **Start IP - End IP:** `192.168.105.221` hingga `192.168.105.225`

![Public Network Config](https://hackmd.io/_uploads/S1UnL-Rkfe.png)

### C. Alokasi Pod
Setiap Pod membutuhkan IP manajemen sendiri untuk mengontrol VM tingkat sistem (*System VM*) yang bertugas menjaga layanan jaringan (Virtual Router, Secondary Storage VM, Console Proxy VM).

- **Name:** `Pod-Group3`
- **Gateway:** `192.168.105.1`
- **Netmask:** `255.255.255.0`
- **Start IP - End IP:** `192.168.105.226` hingga `192.168.105.230`

![Pod Config](https://hackmd.io/_uploads/BkYMOZRkfe.png)

### D. Guest Traffic (VLAN Range)
Pada arsitektur *Advanced*, isolasi penyewa (tenant/guest isolation) difasilitasi oleh alokasi VLAN. Rentang VLAN disetel di angka `3300 - 3339`. Ini artinya, CloudStack bisa membuat *Isolated Network* mandiri menggunakan ID VLAN tersebut.

![Guest Traffic](https://hackmd.io/_uploads/SJ87dZR1zx.png)

### E. Konfigurasi Cluster dan Penambahan Host
Kita mendefinisikan *Cluster-Group3* dan melampirkan server Host ke dalam orkestrasi.
- **Hostname:** `192.168.105.197` (IP Host Utama)
- **Username:** `root`
- **Password:** *Kredensial root pada Host*

Agent di sisi host akan terhubung secara otomatis untuk mengirim sinyal telemetri.

![Host Config](https://hackmd.io/_uploads/B1kSO-0yzg.png)

### F. Pemasangan Storage Primary dan Secondary
Parameter *Primary Storage* (`PrimStor-Group3`) dan *Secondary Storage* (`SecStor-Group3`) didaftarkan, menunjuk pada `/export/primary` dan `/export/secondary` dengan protokol *NFS*. 

![Storage Primary](https://hackmd.io/_uploads/B1YCOZRJfl.png)
![Storage Secondary](https://hackmd.io/_uploads/HyOEFbR1zx.png)

Jika seluruh langkah di atas disetujui, klik **Launch Zone**. CloudStack akan memulai pembuatan struktur Zone di database, mendeploy dua System VM (CPVM - *Console Proxy VM* dan SSVM - *Secondary Storage VM*), dan menginisiasi agent-agent yang relevan.
