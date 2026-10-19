# Modul 19–21: RAID, Kernel Services, dan Virtualisasi
**Dokumentasi belajar Datacomm Linux Administration**  
Bahasa: Indonesia

> Catatan: Materi ini merangkum materi yang telah dibagikan, termasuk lab dan quiz yang dikerjakan. Jalankan perintah yang mengubah partisi, RAID, filesystem, kernel, atau konfigurasi boot hanya pada lab/VM yang sesuai. Perintah tersebut dapat mengakibatkan kehilangan data atau perubahan akses jaringan.

## Daftar Isi
1. [Modul 19 — RAID](#modul-19--raid)
2. [Modul 20 — Kernel Services and Configuration](#modul-20--kernel-services-and-configuration)
3. [Modul 21 — Virtualization Overview](#modul-21--virtualization-overview)
4. [Ringkasan alur praktik](#ringkasan-alur-praktik)

---

# Modul 19 — RAID

## 19.1 Pengenalan RAID

**RAID (Redundant Array of Independent Disks)** menggabungkan beberapa disk atau partisi menjadi satu susunan logis untuk meningkatkan ketersediaan data, performa, atau kapasitas—bergantung pada level RAID yang digunakan.

Manfaat RAID:
- **Redundansi:** data tetap tersedia ketika perangkat tertentu gagal, pada level yang mendukungnya.
- **Performa:** beberapa operasi baca/tulis dapat berlangsung secara paralel.
- **Kapasitas gabungan:** beberapa perangkat terlihat sebagai satu perangkat logis.

RAID bukan pengganti backup. Penghapusan file, kerusakan filesystem, malware, atau kesalahan konfigurasi tetap dapat memengaruhi data RAID.

## 19.2 Konsep RAID

RAID dapat diterapkan melalui:
- **Hardware RAID:** dikendalikan oleh controller RAID.
- **Software RAID:** dikelola oleh sistem operasi, pada Linux umumnya menggunakan `md`/`mdadm`.

Dalam software RAID, perangkat array seperti `/dev/md0` dibangun dari beberapa perangkat blok atau partisi. Filesystem kemudian dibuat di atas perangkat array tersebut.

Alur umumnya:

`Disk/partisi → mdadm RAID array (/dev/mdX) → filesystem → mount point`

Untuk latihan, dua partisi pada disk yang sama dapat digunakan. Namun, ini hanya simulasi: jika disk fisik tersebut gagal, semua anggota RAID pada disk itu ikut hilang. RAID sungguhan sebaiknya menggunakan perangkat fisik terpisah sesuai kebutuhan.

## 19.3 Level RAID

| Level | Cara kerja | Kelebihan | Catatan |
|---|---|---|---|
| RAID 0 | Striping; data dibagi ke beberapa disk | Performa dan kapasitas gabungan | Tidak memiliki redundansi; satu disk gagal dapat merusak seluruh array |
| RAID 1 | Mirroring; data disalin ke anggota array | Redundansi dan pemulihan dari kegagalan anggota | Kapasitas efektif umumnya setara satu anggota mirror |
| RAID 5 | Striping dengan parity terdistribusi | Efisien untuk kapasitas dan dapat menoleransi satu kegagalan disk | Minimal 3 perangkat; rebuild membebani sistem |
| RAID 6 | Striping dengan dua parity | Dapat menoleransi dua kegagalan disk | Minimal 4 perangkat |
| RAID 10 | Gabungan mirroring dan striping | Performa serta redundansi | Umumnya minimal 4 perangkat; kapasitas efektif sekitar setengah total |

**Penting:** RAID 1 biasanya dibuat dengan dua anggota aktif. Quiz 19.1 pada lab menggunakan tiga anggota aktif dan satu spare sesuai instruksi lab. `mdadm` mendukung konfigurasi tersebut, tetapi kapasitas efektif RAID 1 tetap mengikuti ukuran satu salinan mirror, bukan penjumlahan semua anggota.

## 19.4 Konfigurasi Software RAID

Tool utama adalah `mdadm`.

### Memeriksa perangkat

```bash
lsblk
sudo fdisk -l
```

Pastikan nama disk dan partisi benar sebelum melakukan perubahan. Jangan menyalin nama perangkat dari contoh tanpa memeriksa sistem sendiri.

### Membuat partisi untuk RAID

Contoh lab menggunakan `/dev/sdb`. Masuk ke `fdisk`:

```bash
sudo fdisk /dev/sdb
```

Perintah interaktif yang umum:
- `p`: menampilkan tabel partisi.
- `n`: membuat partisi baru.
- `t`: mengubah tipe partisi.
- `d`: menghapus partisi.
- `w`: menulis perubahan ke disk dan keluar.
- `q`: keluar tanpa menyimpan perubahan.

Pada materi lab lama, kode tipe `fd` digunakan untuk Linux RAID pada skema partisi MBR. Jenis kode yang tersedia dapat berbeda berdasarkan tabel partisi dan versi tool.

Contoh latihan membuat dua partisi berukuran sekitar 250 MiB, lalu membuat array:

```bash
sudo mdadm --create /dev/md0 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb3 /dev/sdb4
```

Saat diminta konfirmasi, periksa kembali perangkat yang ditampilkan sebelum menyetujui. Format partisi atau membuat array pada perangkat yang salah dapat merusak data.

### Membuat filesystem dan mount

```bash
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /myraid
sudo mount /dev/md0 /myraid
```

Verifikasi:

```bash
lsblk
findmnt /myraid
df -hT /myraid
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

### Membuat konfigurasi persisten

Agar filesystem dipasang saat boot, tambahkan entri yang sesuai ke `/etc/fstab`. Untuk penggunaan yang lebih andal, pertimbangkan menggunakan UUID filesystem:

```bash
sudo blkid /dev/md0
sudo nano /etc/fstab
```

Contoh entri yang digunakan dalam latihan:

```text
/dev/md0 /myraid ext4 defaults 0 0
```

Simpan konfigurasi array ke file konfigurasi mdadm. Lokasi dapat berbeda menurut distro; pada Ubuntu lab ini digunakan `/etc/mdadm/mdadm.conf`.

```bash
sudo mdadm --detail --scan
```

Setelah memastikan belum ada entri array yang sama, tambahkan hasil scan yang sesuai ke konfigurasi. Contoh:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

Periksa file setelah perubahan dan hindari menambahkan entri duplikat. Pada Ubuntu, perubahan konfigurasi mdadm dapat perlu diikuti pembaruan initramfs jika sistem mengandalkan konfigurasi itu saat boot:

```bash
sudo update-initramfs -u
```

### 19.5 Monitoring RAID

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
lsblk
```

Informasi penting:
- **Active devices:** anggota yang sedang aktif.
- **Working devices:** perangkat yang tersedia dan berfungsi.
- **Spare devices:** perangkat cadangan yang menunggu untuk menggantikan anggota gagal.
- **State:** status keseluruhan array.
- **Resync/recovery:** proses sinkronisasi atau pemulihan yang sedang berjalan.

Periksa status sebelum melakukan perubahan atau menghapus array.

### 19.6 RAID Hot Spares

**Spare** adalah perangkat cadangan yang tidak menjadi anggota aktif utama pada keadaan normal. Jika anggota aktif gagal, spare dapat digunakan untuk rebuild, tergantung konfigurasi dan kondisi array.

```bash
sudo mdadm --add /dev/md1 /dev/sdd4
sudo mdadm --detail /dev/md1
```

### Lab 19.1 — Creating a RAID Device

Tujuan lab:
1. Menyiapkan partisi anggota RAID.
2. Membuat array RAID 1 `/dev/md0`.
3. Membuat filesystem ext4.
4. Memasang filesystem pada `/myraid`.
5. Menyimpan konfigurasi mount dan RAID.
6. Memeriksa status array melalui `/proc/mdstat`.

Contoh perintah inti:

```bash
sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb3 /dev/sdb4
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /myraid
sudo mount /dev/md0 /myraid
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

### Quiz 19.1 — Working with RAID

**Tugas:** membuat empat partisi pada `/dev/sdd`, membuat `/dev/md1` dengan anggota `/dev/sdd1`, `/dev/sdd2`, dan `/dev/sdd3` pada level RAID 1, menambahkan `/dev/sdd4` sebagai spare, lalu menyimpan konfigurasi ke `/etc/mdadm/mdadm.conf`.

Contoh perintah inti setelah partisi dibuat dan diverifikasi:

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=3 \
  /dev/sdd1 /dev/sdd2 /dev/sdd3

sudo mdadm --add /dev/md1 /dev/sdd4
sudo mdadm --detail /dev/md1
cat /proc/mdstat
sudo mdadm --detail --scan
```

Simpan entri array yang benar ke `/etc/mdadm/mdadm.conf`, tanpa duplikasi. Pada quiz yang dikerjakan, `mdadm --detail /dev/md1` menunjukkan tiga active devices, empat working devices, dan satu spare device. Grading mengonfirmasi seluruh konfigurasi benar dengan **skor 100/100**.

### Membersihkan konfigurasi RAID

Hanya lakukan cleanup di lab yang boleh direset dan setelah memastikan tidak ada data yang diperlukan. Urutan konsep:
1. Hapus entri mount dari `/etc/fstab`.
2. Unmount filesystem yang terpasang.
3. Stop array dengan `mdadm --stop`.
4. Hapus array jika diperlukan.
5. Bersihkan metadata RAID dengan `mdadm --zero-superblock` pada anggota yang benar.
6. Hapus partisi hanya jika diminta oleh prosedur reset lab.

Contoh perintah berikut bersifat destruktif dan harus disesuaikan dengan perangkat yang sudah diverifikasi:

```bash
sudo umount /myraid
sudo mdadm --stop /dev/md0
sudo mdadm --remove /dev/md0
sudo mdadm --zero-superblock /dev/sdb3 /dev/sdb4
```

Jangan jalankan `umount` berulang kali pada perangkat dan mount point yang sama; gunakan `findmnt` untuk memeriksa mount yang aktif.

---

# Modul 20 — Kernel Services and Configuration

## 20.1 Tujuan pembelajaran

Setelah mempelajari modul ini, kamu diharapkan mampu:
- Menjelaskan tanggung jawab kernel.
- Memahami parameter pada kernel command line saat boot.
- Menemukan dokumentasi parameter kernel.
- Membaca dan mengubah parameter kernel menggunakan `sysctl`.
- Membuat perubahan kernel parameter bersifat sementara atau persisten.

## 20.2 Kernel Overview

Linux secara teknis adalah kernel, yaitu komponen inti yang menghubungkan perangkat keras dengan perangkat lunak. Distribusi Linux juga mencakup library, aplikasi, service, dan komponen lain.

Tanggung jawab utama kernel:
- Inisialisasi sistem dan proses boot.
- Penjadwalan proses dan alokasi waktu CPU.
- Manajemen memori.
- Pengendalian akses ke perangkat keras melalui driver.
- Penanganan I/O antara aplikasi dan perangkat penyimpanan.
- Dukungan filesystem lokal dan jaringan.
- Pengendalian keamanan dan permission.
- Pengelolaan jaringan.

Kernel menyediakan abstraksi agar aplikasi tidak perlu mengendalikan hardware secara langsung.

## 20.3 Kernel Command Line

Saat boot, bootloader seperti GRUB meneruskan parameter ke kernel. Parameter biasanya berada pada baris `linux` atau `linux16` di konfigurasi boot.

Periksa parameter yang digunakan pada boot saat ini:

```bash
cat /proc/cmdline
```

Parameter setelah path image kernel dapat berbentuk flag atau pasangan `nama=nilai`. Parameter yang tidak dipahami kernel dapat diteruskan kepada proses awal sistem, bergantung pada parameter dan mekanisme boot.

File konfigurasi GRUB dapat berada pada lokasi berbeda menurut distribusi. Jangan mengedit file hasil generate secara sembarang; gunakan mekanisme konfigurasi distro yang sesuai.

## 20.4 Kernel Boot Parameters

Dokumentasi parameter kernel tersedia melalui:
- `man bootparam`
- Dokumentasi kernel Linux, termasuk `kernel-parameters.txt`.
- Paket dokumentasi kernel seperti `linux-doc` atau `kernel-doc`, sesuai distribusi.

Contoh parameter:
- `root=`: lokasi root filesystem.
- `ro`: meminta root filesystem dipasang read-only pada tahap boot awal.
- `crashkernel=`: memesan memori untuk crash dump.
- `quiet`: mengurangi pesan boot yang ditampilkan.
- `LANG=`: pengaturan bahasa/locale yang dapat diteruskan dalam proses boot.
- `vconsole.keymap=`: layout keyboard konsol pada sistem yang mendukungnya.
- `rhgb`: opsi tampilan boot grafis yang digunakan pada distro tertentu.

Parameter boot memengaruhi proses startup. Perubahan permanen perlu dilakukan dengan hati-hati karena parameter yang keliru dapat menyebabkan sistem gagal boot atau berjalan tidak sesuai harapan.

## 20.5 Sysctl

`sysctl` membaca dan mengubah parameter kernel saat runtime. Parameter tersebut berhubungan dengan nilai di bawah `/proc/sys`; garis miring pada path diganti dengan titik pada nama `sysctl`.

Contoh dua cara yang ekuivalen untuk mengaktifkan IPv4 forwarding:

```bash
sudo sh -c 'echo 1 > /proc/sys/net/ipv4/ip_forward'
sudo sysctl net.ipv4.ip_forward=1
```

Cara kedua lebih praktis. Jangan menulis `sudo echo 1 > ...` tanpa shell root, karena pengalihan `>` dilakukan oleh shell pengguna saat ini.

### Membaca parameter

```bash
sysctl net.ipv4.ip_forward
sysctl -a
```

`sysctl -a` menampilkan banyak parameter kernel dan dapat menghasilkan output yang sangat panjang.

### Mengubah parameter sementara

```bash
sudo sysctl net.ipv4.ip_forward=1
```

Perubahan runtime biasanya tidak bertahan setelah reboot jika tidak disimpan ke konfigurasi persisten.

### Membuat perubahan persisten

File konfigurasi yang umum:
- `/etc/sysctl.conf`
- `/etc/sysctl.d/*.conf`

Contoh membuat forwarding persisten:

```bash
sudo nano /etc/sysctl.conf
```

Tambahkan:

```text
net.ipv4.ip_forward=1
```

Terapkan konfigurasi tanpa reboot:

```bash
sudo sysctl -p
```

Untuk file tertentu di `/etc/sysctl.d/`, gunakan `sysctl --system` untuk memuat konfigurasi sistem sesuai mekanisme distribusi.

**Catatan:** IPv4 forwarding memungkinkan kernel meneruskan paket antar-interface dan sering dibutuhkan pada router. Mengaktifkannya saja tidak otomatis membuat sistem menjadi router yang aman; konfigurasi routing dan firewall juga harus benar.

### Lab 20.1 — System Tunables with sysctl

Lab memperkenalkan parameter:

```text
net.ipv4.icmp_echo_ignore_all
```

- Nilai `0`: sistem merespons ICMP Echo Request (ping), jika aturan jaringan lain mengizinkannya.
- Nilai `1`: sistem mengabaikan ICMP Echo Request.

Baca nilai saat ini:

```bash
sysctl net.ipv4.icmp_echo_ignore_all
```

Ubah sementara:

```bash
sudo sysctl net.ipv4.icmp_echo_ignore_all=1
sudo sysctl net.ipv4.icmp_echo_ignore_all=0
```

Ubah melalui `/etc/sysctl.conf`, kemudian terapkan:

```bash
sudo nano /etc/sysctl.conf
sudo sysctl -p
```

Untuk lab, parameter dapat diset ke `1` lalu dikembalikan ke `0`. Jangan lupa menghapus atau mengubah baris persisten bila ingin mengembalikan perilaku awal.

`ping localhost` menguji jalur loopback, tetapi keberhasilan ping tetap bergantung pada parameter ICMP dan konfigurasi sistem.

### Quiz 20.1 — Configure Kernel Parameter

**Tugas:** mengaktifkan IPv4 forwarding secara persisten.

Tambahkan ke `/etc/sysctl.conf` atau file `.conf` yang sesuai di `/etc/sysctl.d/`:

```text
net.ipv4.ip_forward=1
```

Muat konfigurasi:

```bash
sudo sysctl -p
sysctl net.ipv4.ip_forward
```

Output yang diharapkan:

```text
net.ipv4.ip_forward = 1
```

Pada quiz yang dikerjakan, pemeriksaan konfigurasi IPv4 forwarding dinyatakan benar dengan **skor 100/100**.

---

# Modul 21 — Virtualization Overview

## 21.1 Tujuan pembelajaran

Setelah modul ini, kamu diharapkan mampu:
- Memahami konsep virtualisasi dan manfaatnya.
- Membedakan host dan guest.
- Membedakan emulasi dengan virtualisasi.
- Mengenali jenis hypervisor.
- Memahami peran `libvirt`, QEMU, dan KVM.
- Membuat dan mengelola VM menggunakan `virt-manager`, `virt-install`, dan `virsh`.

## 21.2 Apa itu Virtualisasi?

Virtualisasi membuat lingkungan mesin virtual melalui perangkat lunak. Sebuah **Virtual Machine (VM)** memiliki sistem operasi sendiri dan dapat menjalankan aplikasi seolah-olah ia komputer tersendiri.

Manfaat virtualisasi:
- Memanfaatkan perangkat keras secara lebih efisien.
- Menjalankan beberapa sistem operasi pada satu host.
- Mengisolasi workload.
- Membantu pengujian dan pengembangan.
- Memudahkan pembuatan, pemindahan, dan pengelolaan server.

Jenis virtualisasi lain:
- **Network virtualization:** mengabstraksi jaringan fisik, contohnya SDN.
- **Storage virtualization:** menyatukan atau mengabstraksi penyimpanan, contohnya NAS.
- **Application virtualization/container:** mengisolasi aplikasi tanpa harus menjalankan kernel OS terpisah untuk setiap container.

VM tetap memiliki overhead dan performa guest dipengaruhi oleh host, konfigurasi virtual hardware, dan beban kerja.

## 21.3 Sejarah singkat virtualisasi

Virtualisasi telah digunakan pada mainframe untuk meningkatkan pemanfaatan hardware dan mengisolasi workload. Pada PC, awalnya emulasi perangkat keras digunakan untuk menjalankan lingkungan berbeda. Dukungan virtualisasi pada CPU modern kemudian meningkatkan performa dan kemudahan menjalankan VM.

## 21.4 Host dan Guest

- **Host:** mesin fisik dan sistem operasi yang menyediakan sumber daya untuk VM.
- **Guest:** sistem operasi yang berjalan di dalam VM.

Host mengelola CPU, memori, storage, dan jaringan virtual. Guest menjalankan aplikasi dan service di lingkungannya sendiri.

Secara umum:
- Tuning hardware, kapasitas memori, CPU, dan throughput fisik dilakukan dengan memperhatikan host.
- Konfigurasi aplikasi, service, dan OS guest dilakukan di dalam guest.

## 21.5 Emulasi vs. Virtualisasi

**Emulasi** meniru arsitektur hardware melalui perangkat lunak. Ini memungkinkan menjalankan sistem yang arsitekturnya berbeda, tetapi biasanya lebih lambat.

**Virtualisasi berbantuan hardware** menjalankan guest pada arsitektur CPU yang sama dengan bantuan fitur virtualisasi CPU, sehingga performanya umumnya jauh lebih baik.

QEMU dapat digunakan untuk emulasi maupun virtualisasi. Ketika QEMU bekerja bersama KVM pada host yang didukung, VM dapat berjalan dengan akselerasi hardware.

## 21.6 Jenis virtualisasi dan hypervisor

Dua pendekatan umum:
- **Full/hardware virtualization:** guest dapat berjalan tanpa modifikasi khusus karena hypervisor menyajikan perangkat keras virtual.
- **Paravirtualization:** guest mengetahui bahwa ia berada di lingkungan virtual dan menggunakan interface yang dirancang untuk virtualisasi.

CPU Intel umumnya menampilkan flag `vmx`; CPU AMD umumnya menampilkan `svm` jika mendukung virtualisasi x86. Periksa dengan:

```bash
grep -E -m1 'vmx|svm' /proc/cpuinfo
```

Virtualisasi mungkin juga harus diaktifkan di BIOS/UEFI.

Hypervisor dapat berupa lapisan khusus yang berdiri sendiri atau terintegrasi dengan kernel OS. VMware ESXi adalah contoh hypervisor khusus; KVM menggunakan kernel Linux sebagai bagian utama implementasi hypervisor.

## 21.7 Dedicated Hypervisor

Dedicated hypervisor dirancang terutama untuk mengelola VM. Contohnya VMware ESXi. Model ini menempatkan lapisan virtualisasi di bawah VM dan menyediakan pengelolaan hardware serta guest secara terpusat.

## 21.8 Hypervisor di kernel: KVM

**KVM (Kernel-based Virtual Machine)** menambahkan kemampuan virtualisasi ke kernel Linux. KVM memanfaatkan fasilitas kernel untuk penjadwalan, memori, sinkronisasi, dan akses CPU virtualisasi.

Dalam penggunaan umum:
- KVM menyediakan akselerasi virtualisasi di kernel.
- QEMU menyediakan model perangkat keras virtual dan menjalankan proses VM.
- Linux tetap berfungsi sebagai OS host.

Karena VM dikelola sebagai proses Linux, sejumlah tool pengelolaan sumber daya Linux juga dapat membantu mengontrol workload VM.

## 21.9 Program yang menggunakan libvirt

**libvirt** menyediakan API dan tool untuk mengelola hypervisor serta domain/VM. Tool yang menggunakannya antara lain:
- `virsh`: pengelolaan VM dari command line.
- `virt-manager`: antarmuka grafis untuk membuat dan mengelola VM.
- `virt-install`: membuat VM melalui command line.

Ketersediaan fitur bergantung pada distribusi dan konfigurasi host.

## 21.10 Apa itu QEMU?

**QEMU (Quick Emulator)** adalah perangkat lunak untuk emulasi mesin dan virtualisasi. QEMU dapat meniru berbagai arsitektur CPU, termasuk x86, ARM, MIPS, dan lainnya.

Kegunaan QEMU:
- Menjalankan OS guest pada arsitektur yang sama atau berbeda.
- Menguji dan mengembangkan software untuk arsitektur yang belum tersedia secara fisik.
- Membuat, menjeda, menyimpan, dan memulihkan VM.
- Bekerja bersama KVM untuk akselerasi pada host yang mendukung.

QEMU tanpa akselerasi hardware biasanya lebih lambat. Pesan `KVM acceleration not available, using 'qemu'` menunjukkan VM berjalan melalui emulasi QEMU tanpa akselerasi KVM.

## 21.11 Integrasi hypervisor pihak ketiga

QEMU dapat diintegrasikan dengan:
- **KVM:** integrasi erat dengan Linux; akselerasi tinggi bila arsitektur host dan guest sesuai serta fitur hardware tersedia.
- **Xen:** hypervisor yang mendukung mode virtualisasi berbantuan hardware pada platform yang sesuai.
- **VirtualBox:** memiliki hubungan dengan ekosistem image dan tool virtualisasi; format image tertentu dapat dikonversi menggunakan `qemu-img`.

Pada kursus ini, `virt-manager` disarankan untuk GUI, sedangkan `virt-install` dan `virsh` digunakan untuk command line.

## 21.12 Format image disk

Image disk adalah file yang berperan sebagai disk virtual untuk VM.

Format yang umum:
- **raw:** sederhana, kompatibel luas; ukuran file dapat besar jika dialokasikan penuh, meski sparse file dapat menghemat ruang.
- **qcow2:** mendukung fitur seperti copy-on-write dan snapshot; sering digunakan dengan QEMU/KVM.
- **vdi:** terkait dengan VirtualBox.
- **vmdk:** terkait dengan VMware.

Lihat format yang didukung:

```bash
qemu-img --help | grep formats:
```

Konversi image, misalnya qcow2 ke vmdk:

```bash
qemu-img convert -O vmdk myvm.qcow2 myvm.vmdk
```

Periksa dokumentasi `man qemu-img` untuk opsi dan format yang tersedia. Pastikan VM tidak sedang menulis ke image saat operasi konversi atau manipulasi yang berisiko.

## 21.13 KVM dan Linux

Saat modul KVM aktif dan VM dijalankan, Linux berperan sebagai host sekaligus menyediakan fasilitas hypervisor. Fitur virtualisasi CPU seperti Intel VT-x/VMX atau AMD-V/SVM memungkinkan guest menjalankan instruksi secara efisien, sementara operasi tertentu ditangani hypervisor.

KVM dan QEMU bekerja bersama:
1. `libvirt` mengelola konfigurasi domain.
2. QEMU menjalankan proses VM dan menyediakan perangkat virtual.
3. KVM mempercepat eksekusi CPU guest menggunakan hardware virtualization jika tersedia.

## 21.14 Mengelola KVM

### Memeriksa service dan VM

```bash
systemctl is-active libvirtd
sudo virsh list --all
sudo virsh dominfo NAMA_VM
```

Pada sistem berbasis modular libvirt, nama service dapat berbeda, tetapi lab menggunakan `libvirtd`.

### Perintah `virsh` yang penting

```bash
sudo virsh list --all
sudo virsh dominfo cirros
sudo virsh start cirros
sudo virsh shutdown cirros
sudo virsh reboot cirros
sudo virsh suspend cirros
sudo virsh destroy cirros
sudo virsh autostart cirros
sudo virsh autostart --disable cirros
```

Makna perintah:
- `list --all`: menampilkan VM aktif maupun tidak aktif.
- `dominfo`: menampilkan detail domain.
- `start`: menyalakan VM.
- `shutdown`: meminta guest melakukan shutdown dengan normal.
- `reboot`: meminta guest reboot.
- `suspend`: menangguhkan eksekusi VM.
- `destroy`: menghentikan VM secara paksa; tidak sama dengan menghapus definisi VM.
- `autostart`: membuat VM otomatis berjalan saat host boot.
- `autostart --disable`: menonaktifkan autostart.

Gunakan `destroy` hanya bila diperlukan karena penghentian paksa dapat menyebabkan kehilangan data yang belum tersimpan.

### Lab 21.1 — Installing a VM from the Command Line

Lab menunjukkan cara menginstal tool virtualisasi dan membuat VM dengan image CirrOS.

Instalasi paket pada Ubuntu:

```bash
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager -y
sudo systemctl is-active libvirtd
```

Menambahkan pengguna ke grup yang sesuai:

```bash
sudo usermod -aG libvirt "$USER"
sudo usermod -aG kvm "$USER"
```

Perubahan grup umumnya baru efektif setelah login ulang.

### Apa itu CirrOS?

**CirrOS** adalah image Linux yang sangat kecil, terutama digunakan untuk pengujian cloud dan lingkungan virtualisasi. Ia berguna untuk memastikan VM dapat dibuat, dinyalakan, dan terhubung ke jaringan tanpa memakai image OS besar.

CirrOS tetap merupakan guest VM tersendiri. Jika server lab berjalan di VirtualBox, lalu server lab itu menjalankan CirrOS melalui QEMU/KVM, maka secara konsep terjadi **nested virtualization** atau virtualisasi bertingkat. Ini memerlukan dukungan CPU, konfigurasi hypervisor, dan izin yang sesuai. Tanpa akselerasi KVM, QEMU mungkin menjalankan VM melalui emulasi yang lebih lambat.

### Mengunduh image CirrOS

```bash
wget https://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img
sudo mv cirros-0.6.2-x86_64-disk.img /var/lib/libvirt/images/
```

Pastikan nama file yang diunduh sama dengan nama file pada perintah berikutnya. Contoh pada materi pernah menyebut versi image yang berbeda; selalu verifikasi dengan `ls -lh`.

### Membuat VM dengan `virt-install`

```bash
sudo virt-install \
  --name cirros \
  --vcpus 1 \
  --ram 1024 \
  --disk path=/var/lib/libvirt/images/cirros-0.6.2-x86_64-disk.img,format=qcow2 \
  --import \
  --osinfo detect=on,require=off \
  --autostart \
  --network bridge=virbr0 \
  --noautoconsole
```

Penjelasan opsi:
- `--name cirros`: nama domain/VM.
- `--vcpus 1`: satu vCPU.
- `--ram 1024`: RAM 1024 MB.
- `--disk`: image disk yang digunakan.
- `--import`: memakai image OS yang sudah tersedia, bukan instalasi dari media installer.
- `--osinfo`: memberi petunjuk jenis OS atau menggunakan deteksi jika didukung.
- `--autostart`: menyalakan VM otomatis saat host boot.
- `--network bridge=virbr0`: menghubungkan VM ke bridge jaringan virtual yang tersedia.
- `--noautoconsole`: tidak membuka console otomatis; proses pembuatan dapat selesai tanpa interaksi console.

Jika muncul error “Size must be specified for non existent volume”, pastikan path image benar-benar ada dan gunakan opsi `--disk` dengan sintaks yang sesuai. Jika muncul error `--os-variant/--osinfo OS name is required`, tentukan OS info yang didukung atau gunakan `--osinfo detect=on,require=off` jika versi `virt-install` mendukungnya.

Jika `libvirtd` berstatus `inactive`, periksa service dan log sebelum membuat VM:

```bash
sudo systemctl status libvirtd
sudo systemctl start libvirtd
```

Pesan `KVM acceleration not available, using 'qemu'` tidak selalu menggagalkan pembuatan VM, tetapi performa dapat lebih lambat.

### Quiz 21.1 — Configure Domain

**Tugas:** mengatur domain `webserver` agar otomatis start saat host boot, lalu menyalakan VM.

```bash
sudo virsh autostart webserver
sudo virsh start webserver
sudo virsh list --all
sudo virsh dominfo webserver
```

Pada pengerjaan, daftar VM menunjukkan `webserver` dan `cirros` berjalan. Grading menyatakan autostart dan status running benar dengan **skor 100/100**.

**Apakah webserver adalah CirrOS?** Tidak otomatis. `webserver` dan `cirros` adalah nama domain yang berbeda. Jenis OS dalam domain `webserver` ditentukan oleh image dan konfigurasi VM yang disediakan lab. Quiz 21.1 meminta mengelola VM yang sudah dibuat oleh sistem; bukan membuat ulang `webserver` dari image CirrOS.

### Quiz 21.2 — Create Virtual Machine

**Tugas:** membuat VM bernama `jammy` dari image Ubuntu Server cloud image dengan RAM 1024 MB, menggunakan `virt-install` dan `--noautoconsole`.

Image yang disediakan kursus:

```text
https://s3-jkt1.dcloud.co.id/lms-datacomm-asset-qre62/quiz-21.1/jammy-server-cloudimg-amd64.img
```

#### Metode A — Download langsung dari terminal dengan `wget`

```bash
wget -O jammy-server-cloudimg-amd64.img \
  "https://s3-jkt1.dcloud.co.id/lms-datacomm-asset-qre62/quiz-21.1/jammy-server-cloudimg-amd64.img"
```

Lalu pindahkan ke direktori image libvirt:

```bash
sudo mv jammy-server-cloudimg-amd64.img /var/lib/libvirt/images/
```

Jika unduhan lambat, terputus, atau memakan waktu lama, gunakan metode B.

#### Metode B — Download di Windows lalu akses melalui VirtualBox Shared Folder

Metode ini digunakan saat `wget` dari VM terlalu lambat.

1. Unduh image melalui browser Windows.
2. Simpan file ke folder lokal, misalnya `C:\\labs-21.2`.
3. Pada pengaturan VM server di VirtualBox, tambahkan folder tersebut sebagai **Shared Folder**, dengan path yang benar dan opsi Auto-mount jika tersedia.
4. Di dalam guest Linux, pastikan Guest Additions tersedia dan aktif:

```bash
sudo modprobe vboxsf
lsmod | grep vboxsf
```

5. Buat mount point dan mount shared folder. Contoh berikut mengasumsikan nama shared folder di VirtualBox adalah `labs-21.2`:

```bash
sudo mkdir -p /mnt/shared
sudo mount -t vboxsf labs-21.2 /mnt/shared
ls -lh /mnt/shared
```

6. Salin image ke lokasi yang dapat diakses libvirt:

```bash
sudo cp /mnt/shared/jammy-server-cloudimg-amd64.img /var/lib/libvirt/images/
sudo ls -lh /var/lib/libvirt/images/jammy-server-cloudimg-amd64.img
```

**Catatan:** nama share pada perintah `mount -t vboxsf` harus sama persis dengan nama yang terdaftar di pengaturan VirtualBox. Folder share Windows tidak otomatis muncul di `/media`; bila perlu, mount secara manual seperti contoh di atas.

#### Membuat VM `jammy`

Setelah image tersedia, jalankan:

```bash
sudo virt-install \
  --name jammy \
  --memory 1024 \
  --vcpus 1 \
  --disk path=/var/lib/libvirt/images/jammy-server-cloudimg-amd64.img,format=qcow2 \
  --import \
  --osinfo detect=on,require=off \
  --noautoconsole
```

Image cloud dapat memiliki karakteristik berbeda dari image installer biasa. Ikuti spesifikasi lab dan sesuaikan opsi bila versi `virt-install` pada sistem berbeda.

Verifikasi:

```bash
sudo virsh list --all
sudo virsh dominfo jammy
```

Pada pengerjaan quiz, image berhasil diakses melalui Shared Folder VirtualBox dan VM `jammy` berhasil dibuat. Grading menyatakan konfigurasi VM benar dengan **skor 100/100**.

---

# Ringkasan alur praktik

## RAID
1. Identifikasi disk dan partisi dengan `lsblk`.
2. Siapkan partisi yang benar.
3. Buat array dengan `mdadm`.
4. Tambahkan spare jika diminta.
5. Periksa `/proc/mdstat` dan `mdadm --detail`.
6. Simpan konfigurasi array tanpa entri duplikat.
7. Buat filesystem dan mount bila dibutuhkan.
8. Lakukan cleanup hanya setelah memastikan data tidak diperlukan.

## Kernel dan sysctl
1. Baca parameter dengan `sysctl`.
2. Ubah nilai sementara menggunakan `sudo sysctl nama.parameter=nilai`.
3. Untuk persistensi, simpan di `/etc/sysctl.conf` atau `/etc/sysctl.d/*.conf`.
4. Terapkan menggunakan `sudo sysctl -p` atau `sudo sysctl --system`.
5. Verifikasi nilai aktual setelah perubahan.

## Virtualisasi
1. Periksa service libvirt, dukungan CPU, dan image.
2. Buat VM dengan `virt-install` atau `virt-manager`.
3. Kelola domain dengan `virsh`.
4. Periksa status dengan `virsh list --all` dan `virsh dominfo`.
5. Gunakan `autostart` hanya untuk VM yang memang harus menyala saat host boot.
6. Jika image terlalu besar atau `wget` lambat, gunakan Shared Folder VirtualBox untuk memindahkan file dari host Windows ke guest Linux.

## Glosarium singkat

| Istilah | Arti |
|---|---|
| RAID | Gabungan beberapa perangkat penyimpanan untuk redundansi, performa, atau kapasitas |
| `mdadm` | Tool Linux untuk mengelola software RAID |
| Spare | Perangkat cadangan untuk membantu menggantikan anggota RAID yang gagal |
| Kernel | Inti OS yang mengelola hardware dan sumber daya |
| `sysctl` | Interface untuk membaca dan mengubah parameter kernel |
| IPv4 forwarding | Kemampuan kernel meneruskan paket IPv4 antar-interface |
| VM | Mesin virtual yang menjalankan guest OS |
| Host | Mesin dan OS yang menjalankan hypervisor/VM |
| Guest | OS yang berjalan di dalam VM |
| QEMU | Tool emulasi dan virtualisasi mesin |
| KVM | Fasilitas virtualisasi di kernel Linux |
| libvirt | API dan kumpulan tool untuk mengelola hypervisor/domain |
| `virt-install` | Tool command-line untuk membuat VM |
| `virsh` | Tool command-line untuk mengelola domain/VM |
| `qcow2` | Format image disk QEMU dengan dukungan fitur seperti copy-on-write |
