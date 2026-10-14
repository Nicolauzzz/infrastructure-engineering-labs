# Linux Administration: Signals, Package Management, I/O, dan Filesystem

Dokumentasi ini melanjutkan materi `programs n process.md` di folder `materials`. Isinya merangkum materi Linux Administration yang telah dipelajari dan praktik quiz yang sudah dikerjakan.

## Daftar Isi

1. [Signals dan Pengelolaan Proses](#1-signals-dan-pengelolaan-proses)
2. [Software Package Management](#2-software-package-management)
3. [Monitoring Aktivitas I/O](#3-monitoring-aktivitas-io)
4. [I/O Scheduling](#4-io-scheduling)
5. [Filesystem Linux](#5-filesystem-linux)
6. [Praktik dan Quiz](#6-praktik-dan-quiz)
7. [Perintah Referensi](#7-perintah-referensi)

---

## 1. Signals dan Pengelolaan Proses

Signal adalah notifikasi asynchronous yang dikirimkan ke proses, misalnya oleh kernel atau proses lain. Signal dapat digunakan untuk meminta proses melakukan tindakan tertentu, menghentikan proses, atau memberi tahu proses tentang suatu kondisi.

### Signal penting

- `SIGTERM` (15): meminta proses berhenti secara normal. Proses dapat menangani atau mengabaikannya.
- `SIGKILL` (9): menghentikan proses secara paksa dan tidak dapat ditangkap atau diabaikan oleh proses.
- `SIGSTOP`: menghentikan sementara proses. Signal ini tidak dapat ditangkap atau diabaikan.
- Signal real-time dimulai dari `SIGRTMIN`; signal jenis ini dapat diantrikan.

### Perintah yang dipelajari

```bash
kill <PID>
kill -SIGTERM <PID>
kill -9 <PID>
killall <nama_proses>
pkill <kriteria>
ps aux
ps aux | grep "bash"
kill -l
```

`pkill` mendukung pemilihan proses berdasarkan kriteria, misalnya `-P` untuk parent PID, `-G` untuk group ID, dan `-U` untuk user ID. Gunakan `killall` atau `pkill` secara hati-hati karena beberapa proses dengan nama yang sama dapat ikut terpengaruh.

Referensi manual: `man 7 signal`, `man kill`, `man pkill`.

---

## 2. Software Package Management

Package manager membantu memasang, memperbarui, mengonfigurasi, dan menghapus software secara konsisten. Paket dapat menyertakan executable, library, dokumentasi, file konfigurasi, skrip instalasi, metadata versi, checksum, vendor, dan dependency.

Metadata paket disimpan dalam database lokal sehingga sistem dapat memeriksa status paket dan mengelola pembaruan serta dependensinya.

### Jenis paket

- **Binary package**: file siap dipasang, seperti executable dan library; umumnya bergantung pada arsitektur.
- **Source package**: kode sumber yang dapat digunakan untuk membangun binary package.
- **Architecture-independent package**: konten yang tidak bergantung pada arsitektur tertentu, seperti skrip, konfigurasi, atau dokumentasi.
- **Meta-package**: paket yang mengelompokkan paket lain untuk memenuhi suatu fungsi, misalnya desktop environment.

### Perintah APT yang dipelajari

```bash
sudo apt update
sudo apt install <nama-paket>
sudo apt remove <nama-paket>
apt show <nama-paket>
dpkg -l
```

`apt update` memperbarui indeks repositori, bukan langsung memperbarui semua aplikasi. `apt install` memasang paket beserta dependency yang dibutuhkan. Pastikan sumber repositori dan metadata paket valid jika terjadi masalah saat instalasi.

Dalam troubleshooting lab, sumber repositori yang bermasalah dapat menyebabkan kegagalan unduh atau ketidakcocokan hash. Periksa konfigurasi repositori dan jalankan pembaruan indeks kembali setelah sumber diperbaiki.

---

## 3. Monitoring Aktivitas I/O

Masalah performa disk dapat berkaitan dengan CPU, memori, atau jaringan. Sistem dapat disebut **I/O-bound** ketika CPU banyak menunggu operasi I/O selesai atau ketika transfer jaringan tertahan oleh proses I/O.

Monitoring dan tracing membantu menemukan bottleneck, tetapi satu metrik saja tidak selalu cukup untuk menentukan akar masalah.

### `iostat`

`iostat` menampilkan ringkasan penggunaan CPU dan statistik aktivitas perangkat I/O.

```bash
iostat
iostat -k
iostat -m
iostat -N
iostat -d
iostat -xk
iostat 2 5
```

- `-k`: menampilkan ukuran dalam kilobyte.
- `-m`: menampilkan ukuran dalam megabyte.
- `-N`: menampilkan nama device yang lebih informatif.
- `-d`: menampilkan statistik perangkat.
- `-x`: menampilkan statistik extended.
- `2 5`: mengambil laporan dengan interval 2 detik sebanyak 5 kali.

Pada output extended, perhatikan metrik seperti throughput, jumlah operasi, waktu tunggu, dan persentase utilisasi. Utilisasi yang mendekati 100% dapat mengindikasikan perangkat jenuh, tetapi tetap perlu dibaca bersama metrik lain dan karakteristik workload.

### `iotop`

`iotop` menampilkan penggunaan I/O proses secara berkala, mirip dengan `top`, dan biasanya membutuhkan hak akses root.

```bash
sudo iotop
sudo iotop -o
iotop --help
```

Opsi `-o` membantu menampilkan proses yang sedang melakukan aktivitas I/O agar output lebih ringkas.

### `ionice`

`ionice` mengatur kelas penjadwalan I/O dan prioritas suatu proses.

```bash
ionice
ionice -c 2 -n 3 -p <PID>
```

Kelas umum `ionice` adalah:

- `1`: Real Time
- `2`: Best Effort
- `3`: Idle

Untuk kelas Real Time dan Best Effort, nilai prioritas berada pada rentang 0–7; angka 0 berarti prioritas tertinggi. Efek pengaturan bergantung pada kernel, scheduler, dan perangkat yang digunakan. Dukungan scheduler lama seperti CFQ berbeda dari kernel modern.

---

## 4. I/O Scheduling

I/O scheduler berada di antara block layer Linux dan driver perangkat fisik. Scheduler mengatur urutan dan prioritas permintaan baca/tulis sebelum dikirim ke perangkat.

Tujuan I/O scheduling dapat saling bertentangan:

- Mengurangi waktu akses perangkat.
- Menggabungkan permintaan agar transfer lebih efisien.
- Mengurangi latency dan memenuhi deadline.
- Memberi respons yang baik untuk operasi baca.
- Membagi bandwidth secara adil atau berdasarkan prioritas.
- Menyesuaikan perilaku dengan jenis perangkat dan workload, termasuk SSD.

### Melihat scheduler yang tersedia

Contoh untuk perangkat `/dev/sda`:

```bash
cat /sys/block/sda/queue/scheduler
```

Nama scheduler yang tersedia bergantung pada kernel, perangkat, dan distribusi Linux. Contoh yang mungkin muncul pada kernel modern adalah `none`, `mq-deadline`, `bfq`, atau `kyber`.

### Mengubah scheduler

Perubahan dilakukan dengan hak akses administrator dan hanya jika scheduler yang dipilih tersedia:

```bash
echo bfq | sudo tee /sys/block/sda/queue/scheduler
cat /sys/block/sda/queue/scheduler
```

Perubahan melalui `/sys` umumnya berlaku saat runtime dan dapat hilang setelah reboot. Jangan menyalin contoh `/dev/sda` atau nama scheduler secara membabi buta; periksa nama perangkat dan opsi yang benar pada sistem terlebih dahulu.

---

## 5. Filesystem Linux

Filesystem mengatur bagaimana file dan direktori disimpan serta ditemukan pada perangkat penyimpanan. Aplikasi menggunakan nama dan path file, bukan alamat fisik disk secara langsung.

Linux memakai struktur direktori berbentuk pohon yang berakar pada `/`. Beberapa filesystem dapat digabungkan ke dalam satu pohon melalui proses mounting.

### Inode

Setiap objek file pada filesystem Linux memiliki inode yang menyimpan metadata seperti:

- Permission.
- Pemilik dan group.
- Ukuran.
- Timestamp akses, modifikasi, dan perubahan metadata.
- Referensi ke lokasi data file.

Nama file disimpan sebagai entri direktori yang menghubungkan nama dengan inode. Karena itu, nama file dan inode adalah dua hal yang berbeda.

### Hard link dan symbolic link

**Hard link**
- Merupakan entri nama tambahan yang menunjuk ke inode yang sama.
- Hanya dapat dibuat pada filesystem yang sama.
- Beberapa nama hard link merujuk pada data file yang sama.
- Menghapus salah satu nama tidak menghapus data selama masih ada hard link lain dan file masih terbuka oleh proses.

**Symbolic link (soft link)**
- Menyimpan path menuju target.
- Dapat menunjuk ke target di filesystem lain.
- Dapat menjadi dangling link jika target tidak ada atau tidak dapat diakses.

Perintah yang digunakan:

```bash
ln <target> <hard_link>
ln -s <target> <soft_link>
readlink <soft_link>
readlink -f <soft_link>
ls -li <file1> <file2>
stat <file>
find <direktori> -type f -links +1
```

`readlink -f` mencoba menyelesaikan path symbolic link menjadi path absolut. `ls -li` menampilkan inode dan jumlah hard link pada kolom metadata. Untuk menghitung jumlah link, perhatikan nilai jumlah link pada metadata file, bukan sekadar jumlah baris hasil `find`.

### VFS

Virtual Filesystem (VFS) adalah lapisan abstraksi yang memungkinkan aplikasi mengakses berbagai jenis filesystem melalui antarmuka yang seragam. VFS menerjemahkan operasi file menjadi operasi yang sesuai untuk filesystem terkait, termasuk filesystem jaringan seperti NFS.

### Jenis filesystem

Contoh filesystem yang umum dijumpai:

- `ext4`: filesystem Linux yang banyak digunakan.
- `XFS`: filesystem berperforma tinggi.
- `Btrfs`: mendukung fitur seperti snapshot dan pengelolaan volume.
- `VFAT`/`FAT`: umum untuk media yang kompatibel lintas sistem.
- `NTFS`: filesystem yang digunakan Windows.
- `NFS`: filesystem jaringan.

Daftar filesystem yang dikenali kernel dapat dilihat dengan:

```bash
cat /proc/filesystems
```

Entri dengan `nodev` biasanya menunjukkan pseudo-filesystem yang tidak menggunakan perangkat block sebagai penyimpanan biasa.

### Journaling filesystem

Journaling filesystem mencatat perubahan sebagai transaksi sehingga pemulihan setelah crash atau shutdown tidak normal dapat dilakukan lebih cepat dan mengurangi risiko inkonsistensi metadata. Contoh yang dikenal antara lain ext3, ext4, XFS, JFS, dan Btrfs. Fitur serta status dukungan masing-masing filesystem bergantung pada versi kernel dan distribusi.

### Special filesystem

Linux menyediakan pseudo-filesystem untuk mengekspos informasi kernel dan perangkat:

| Filesystem | Mount point umum | Kegunaan |
|---|---|---|
| `proc` | `/proc` | Informasi proses dan struktur kernel |
| `sysfs` | `/sys` | Informasi perangkat dan model objek kernel |
| `tmpfs` | Sesuai konfigurasi | Penyimpanan sementara berbasis memori, dapat menggunakan swap |
| `devpts` | `/dev/pts` | Pseudo-terminal |
| `debugfs` | `/sys/kernel/debug` | Informasi debugging kernel jika diaktifkan |
| `hugetlbfs` | Sesuai konfigurasi | Akses ke halaman memori besar |
| `sockfs` | Tidak selalu memiliki mount point pengguna | Fasilitas internal kernel untuk socket |

---

## 6. Praktik dan Quiz

### Quiz 8.1 — Log `instructor2`

Praktik ini melibatkan pencarian log yang berkaitan dengan akun `instructor2` dan penyimpanan hasil ke:

```text
/opt/quiz-8.1/instructor2.log
```

Contoh perintah untuk mencari informasi di log:

```bash
sudo grep -i "instructor2" /var/log/auth.log
sudo journalctl --since "2026-10-14 00:00:00" --no-pager
```

Setelah isi file log disiapkan, jalankan:

```bash
dcomctl grading quiz-8.1
```

Hasil grading yang diperoleh: **100/100**. File log dinyatakan benar oleh grader.

### Quiz 11.1 — Hard and Soft Link

Tugas quiz mencakup pembuatan symbolic link, penyelesaian path symbolic link, pembuatan hard link, dan pencatatan jumlah hard link.

Contoh perintah:

```bash
sudo ln -s /opt/quiz-11.1/resources/source_object.dat /opt/quiz-11.1/answers/softlink
readlink -f /opt/quiz-11.1/answers/softlink | sudo tee /opt/quiz-11.1/answers/dereference.txt
sudo ln /opt/quiz-11.1/resources/hard_target /opt/quiz-11.1/answers/hard_link
ls -li /opt/quiz-11.1/resources/hard_target /opt/quiz-11.1/answers/hard_link
dcomctl grading quiz-11.1
```

Jumlah hard link harus ditentukan dari file yang dimaksud pada filesystem `resources/hardlink/`, lalu ditulis ke `hard_link_count.txt`. Pada percobaan yang dilakukan, symbolic link, path hasil dereference, dan hard link dinyatakan benar, tetapi nilai jumlah hard link belum sesuai. Nilai akhir: **75/100**.

Untuk quiz lain, jalankan perintah grading yang sesuai dengan nama quiz sebelum reset lab.

---

## 7. Perintah Referensi

| Tujuan | Perintah |
|---|---|
| Melihat proses | `ps aux` |
| Mengirim signal | `kill -TERM <PID>` |
| Menghentikan paksa proses | `kill -KILL <PID>` |
| Melihat jenis signal | `kill -l` |
| Memperbarui indeks paket | `sudo apt update` |
| Memasang paket | `sudo apt install <paket>` |
| Memantau statistik disk | `iostat -xk` |
| Memantau I/O proses | `sudo iotop -o` |
| Melihat prioritas I/O | `ionice -p <PID>` |
| Melihat scheduler perangkat | `cat /sys/block/<device>/queue/scheduler` |
| Melihat filesystem kernel | `cat /proc/filesystems` |
| Melihat inode dan jumlah link | `ls -li <file>` |
| Menyelesaikan symbolic link | `readlink -f <link>` |
| Memeriksa metadata file | `stat <file>` |

## Catatan

- Jalankan perintah administratif dengan `sudo` hanya ketika diperlukan.
- Periksa path, nama perangkat, PID, dan nama quiz sebelum menjalankan perintah.
- Simpan screenshot evidence pada folder `evidence/` dan catat hasil grading yang benar-benar diperoleh.
