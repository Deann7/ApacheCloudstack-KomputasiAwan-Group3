# Troubleshooting dan Investigasi Masalah

Membuat lingkungan komputasi awan privat (Private Cloud) tidak lepas dari berbagai kendala konfigurasi tingkat lanjut. Dokumen ini mendokumentasikan serangkaian error *(kendala)* yang dihadapi selama implementasi Apache CloudStack serta upaya penyelesaian (Troubleshooting) yang dilakukan tim untuk memastikan layanan kembali stabil.

## 1. Kegagalan Peluncuran Konfigurasi Basic Zone

Percobaan pertama instalasi dilakukan dengan pendekatan perancangan jaringan *Basic Networking*, dimana mesin virtual mendapatkan *IP address* langsung dari jaringan datar yang sama dengan host (mirip dengan implementasi *bridge* sederhana). Namun, proses *Launch Zone* di akhir mengalami gagal total (*error*).

![Error Launch Zone](https://hackmd.io/_uploads/r1Xp4fRkGe.png)

Kegagalan ini utamanya dipicu oleh konfigurasi isolasi jaringan dan pembagian paket perutean yang kurang sesuai untuk eksekusi Single Node, dimana *System VM* tidak menemukan cara untuk mengelola fungsi *Gateway* secara alami. Berdasarkan kegagalan ini, Zone *Basic* diputuskan untuk dihapus secara penuh melalui *Database* maupun antarmuka UI, dan disubstitusikan dengan skenario percobaan kedua: **Advanced Networking Zone**.

## 2. Isu Instabilitas CloudStack Agent dan "Add Host"

Meskipun instalasi Advanced Networking jauh lebih lancar, muncul tantangan yang lebih kompleks: Host Agent KVM menolak untuk tetap berada pada status `Up`. Proses penambahan Host selalu menemui siklus gagal koneksi berulang dari `cloudstack-agent` kepada `cloudstack-management`.

![Kendala Agent Host](https://hackmd.io/_uploads/H1-LwmAkfl.png)

Penyelidikan mendalam membuktikan Agent CloudStack sangat sensitif terhadap sejumlah kriteria lingkungan. Tim melakukan pendekatan berlapis untuk mengatasi isu ketidakstabilan ini:

### A. Verifikasi Status Sinkronisasi Waktu (Chrony)
Apabila waktu server Management berbeda sedetik pun dengan waktu *Hypervisor / Agent*, sertifikat atau koneksi API SSL akan dianggap kadaluarsa atau tidak absah, memutus alur persetujuan Host. 

Pengecekan menggunakan command `chronyc tracking` dilakukan untuk melihat *System time* dan selisihnya:
```bash
chronyc tracking
```
![Status Chrony](https://hackmd.io/_uploads/HkS0a201Gg.png)
*(Terpantau waktu tersinkron dengan sangat akurat hingga sepersekian milidetik).*

### B. Resolusi DNS dan Modifikasi File Hosts (`/etc/hosts`)
Dalam infrastruktur *Single Node*, server seringkali berusaha mencari identitas namanya sendiri untuk mengirim paket layanan. Resolusi nama host (*hostname resolution*) yang cacat sering menyebabkan paket RPC terputus. Solusinya, file `/etc/hosts` ditambahkan dengan pemetaan manual antara IP Host dan *Hostname*.

```
192.168.105.197 w1660
```
![Konfigurasi Hosts](https://hackmd.io/_uploads/SytAp2CyMg.png)

### C. Restart dan Penyatuan Siklus Layanan (Service Management)
Layanan yang menggantung (hang) karena salah baca konfigurasi awal harus direstart sepenuhnya secara sekuensial (mulai dari RPC, NFS, Libvirt, Agent, dan terakhir Management). 

![Restart Services 1](https://hackmd.io/_uploads/S182NfAJMx.png)
![Restart Services 2](https://hackmd.io/_uploads/BkdWLGAJMl.png)

> **Catatan Tim Terkait Masalah Agent:**
> Terlepas dari semua tindakan mitigasi ini (verifikasi NTP, resolusi `/etc/hosts`, layanan KVM/Libvirt), masalah pada stabilitas agent untuk terhubung dengan server management berakar sangat dalam pada kompatibilitas versi *libvirt* atau limitasi implementasi arsitektur jaringan khusus pada sesi Node tunggal ini. Diperlukan investigasi level kernel log dan review file log `agent.log` lebih jauh di masa mendatang.
