# Quiz 18.1 — Extend LVM Filesystem

## Informasi Quiz

**Topik:** Logical Volume Management (LVM)  
**Tujuan:** Menambahkan kapasitas 2 GB pada filesystem berbasis LVM yang ter-mount di `/lvm-extend`, tanpa melakukan unmount.  
**Hasil:** Berhasil — **Score 100/100**.

## Soal

**Context:** An LVM-backed filesystem is filling up.

Without unmounting the LVM-backed filesystem mounted at `/lvm-extend`, add 2 GB of disk space from the volume group `system`.

Jalankan perintah berikut untuk memeriksa hasil dan membersihkan lab:

```bash
dcomctl grading quiz-18.1
dcomctl reset-lab quiz-18.1
```

## 1. Persiapan Lab

```bash
dcomctl build-lab quiz-18.1
```

Perintah ini menjalankan skrip persiapan dan mengonfigurasi lingkungan lab.

## 2. Pemeriksaan Awal

Periksa kapasitas filesystem yang ter-mount di `/lvm-extend`:

```bash
df -h /lvm-extend
```

Hasil awal yang terlihat pada lab:

```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/pool--1-LVM--backend    974M  891M   17M  99% /lvm-extend
```

Periksa Logical Volume:

```bash
sudo lvs
```

Ringkasan hasil awal:

```text
LV           VG       Attr       LSize
LVM-backend  pool-1   -wi-ao----  1.00g
ubuntu-lv    ubuntu-vg -wi-ao--- <48.00g
```

Periksa Volume Group:

```bash
sudo vgs pool-1
```

Hasil yang terlihat:

```text
VG      #PV #LV #SN Attr   VSize    VFree
pool-1    1   1   0 wz--n- <10.00g <9.00g
```

**Catatan:** Pada lingkungan quiz ini, nama Volume Group yang benar adalah `pool-1`, bukan `system` seperti yang disebutkan dalam instruksi soal. Output aktual dari lab digunakan untuk mengidentifikasi LV dan VG yang benar.

## 3. Perintah Penyelesaian

Logical Volume yang digunakan oleh `/lvm-extend` adalah `/dev/pool-1/LVM-backend`.

Tambahkan 2 GB dan sesuaikan ukuran filesystem secara otomatis:

```bash
sudo lvextend -r -L +2G /dev/pool-1/LVM-backend
```

Arti opsi:

- `lvextend`: memperbesar Logical Volume.
- `-r`: memperbesar filesystem bersamaan dengan Logical Volume.
- `-L +2G`: menambahkan 2 GB ke ukuran LV saat ini; tanda `+` berarti menambah, bukan menetapkan ukuran total menjadi 2 GB.
- `/dev/pool-1/LVM-backend`: jalur LV yang benar berdasarkan hasil pemeriksaan lab.

Filesystem tetap ter-mount selama operasi ini, sesuai permintaan soal.

## 4. Verifikasi

Periksa ukuran LV:

```bash
sudo lvs
```

Hasil setelah diperbesar menunjukkan `LVM-backend` berukuran `3.00g`:

```text
LV           VG       Attr       LSize
LVM-backend  pool-1   -wi-ao----  3.00g
ubuntu-lv    ubuntu-vg -wi-ao--- <48.00g
```

Jalankan penilaian otomatis:

```bash
dcomctl grading quiz-18.1
```

Hasil:

```text
Running the assessment for quiz-18.1
Initiating the configuration check...
Checking Extend LVM Filesystem: The configuration is correct.
Your Score: 100
Grading completed successfully!
```

**Status: Lulus, skor 100/100.**

## 5. Ringkasan Konsep

- `df -h /lvm-extend` digunakan untuk memeriksa kapasitas filesystem dan titik mount.
- `sudo lvs` menampilkan Logical Volume, Volume Group, dan ukuran LV.
- `sudo vgs pool-1` menampilkan kapasitas Volume Group dan ruang kosongnya.
- `sudo lvextend -r -L +2G /dev/pool-1/LVM-backend` menambah kapasitas LV sebesar 2 GB dan memperbesar filesystem tanpa unmount.
- `dcomctl grading quiz-18.1` memvalidasi hasil konfigurasi.
- `dcomctl reset-lab quiz-18.1` mereset lingkungan lab; jalankan hanya setelah selesai jika memang ingin membersihkan atau mengulang lab.

## 6. Catatan Penting

Instruksi soal menyebut Volume Group `system`, tetapi hasil pemeriksaan aktual menunjukkan VG `pool-1` dan LV `LVM-backend`. Karena itu, perintah yang berhasil pada lingkungan ini menggunakan `/dev/pool-1/LVM-backend`.

Sebelum menjalankan perintah LVM di lingkungan lain, selalu periksa `df`, `lvs`, dan `vgs`, lalu pastikan target LV benar. Jangan menyalin nama LV/VG dari lab ini secara membabi buta.
