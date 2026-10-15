# Quiz 19.1 — Working with RAID

## 1. Tujuan

Quiz ini melatih konfigurasi **Software RAID 1** di Linux menggunakan `mdadm`. Tugasnya:

1. Membuat empat partisi pada `/dev/sdd`.
2. Membuat RAID `/dev/md1` dengan RAID level 1 dan tiga anggota aktif.
3. Menambahkan `/dev/sdd4` sebagai *spare device*.
4. Menyimpan konfigurasi RAID ke `/etc/mdadm/mdadm.conf`.
5. Memeriksa konfigurasi dan status RAID.

**Hasil akhir: skor 100/100.**

## 2. Konsep Penting

- **RAID 1 (mirroring):** menyimpan salinan data pada beberapa anggota RAID untuk menyediakan redundansi.
- **RAID device (`/dev/md1`):** perangkat logis yang dikelola oleh Linux Software RAID.
- **Anggota aktif:** partisi yang sedang menjadi bagian utama array.
- **Spare device:** partisi cadangan yang dapat digunakan untuk menggantikan anggota aktif yang gagal.
- **`mdadm`:** utilitas untuk membuat, memantau, dan mengelola Software RAID.
- **`/etc/mdadm/mdadm.conf`:** file konfigurasi RAID pada lingkungan Ubuntu yang digunakan dalam quiz ini.

## 3. Alur Pengerjaan

### Langkah 1 — Menyiapkan lab

```bash
dcomctl build-lab quiz-19.1
```

Perintah ini menyiapkan lingkungan quiz. Pastikan disk target sesuai soal sebelum mengubah partisi.

### Langkah 2 — Membuat empat partisi pada `/dev/sdd`

```bash
sudo fdisk /dev/sdd
```

Di dalam `fdisk`, buat partisi nomor 1 hingga 4 menggunakan `n`. Ikuti ukuran yang ditentukan lab, lalu periksa tabel partisi dengan `p`.

Ubah tipe setiap partisi menjadi Linux RAID menggunakan kode `fd`:

```text
t
1
fd
t
2
fd
t
3
fd
t
4
fd
```

Jika `fdisk` meminta nomor partisi atau kode tipe, masukkan sesuai urutan. Periksa tabel partisi dengan `p`, lalu simpan menggunakan `w`.

Verifikasi dari terminal:

```bash
lsblk /dev/sdd
```

Pastikan `/dev/sdd1`, `/dev/sdd2`, `/dev/sdd3`, dan `/dev/sdd4` terdeteksi.

### Langkah 3 — Membuat RAID 1 dengan tiga anggota aktif

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=3 \
  /dev/sdd1 /dev/sdd2 /dev/sdd3
```

Penjelasan:

- `--create /dev/md1`: membuat perangkat RAID bernama `/dev/md1`.
- `--level=1`: memilih RAID 1.
- `--raid-devices=3`: menentukan tiga anggota aktif.
- `/dev/sdd1`, `/dev/sdd2`, `/dev/sdd3`: partisi yang menjadi anggota aktif.

Jika muncul pertanyaan konfirmasi, periksa kembali nama perangkat dan setujui hanya jika sesuai dengan lingkungan lab.

### Langkah 4 — Menambahkan spare device

```bash
sudo mdadm --add /dev/md1 /dev/sdd4
```

Perintah ini menambahkan `/dev/sdd4` ke array sebagai perangkat cadangan. Jangan gunakan partisi yang berisi data penting atau sedang digunakan filesystem lain.

### Langkah 5 — Menyimpan konfigurasi array

```bash
sudo mdadm --detail --scan
```

Perintah ini menampilkan baris konfigurasi array yang dikenali. Simpan konfigurasi untuk `/dev/md1` ke `/etc/mdadm/mdadm.conf`.

Buka file konfigurasi:

```bash
sudo nano /etc/mdadm/mdadm.conf
```

Tambahkan baris `ARRAY` yang benar dari hasil `mdadm --detail --scan` untuk `/dev/md1`. Jangan menyalin UUID contoh atau membuat entri duplikat jika baris array sudah tersedia.

Simpan di Nano dengan `Ctrl+O`, tekan `Enter`, lalu keluar menggunakan `Ctrl+X`.

### Langkah 6 — Memverifikasi status RAID

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md1
```

`/proc/mdstat` memberikan ringkasan status array. `mdadm --detail` menampilkan informasi lengkap seperti level RAID, jumlah anggota, status aktif, spare, dan UUID.

## 4. Hasil yang Diperoleh

Output `sudo mdadm --detail /dev/md1` menunjukkan:

| Informasi | Hasil |
|---|---|
| RAID Device | `/dev/md1` |
| RAID Level | `raid1` |
| RAID Devices | 3 |
| Total Devices | 4 |
| Active Devices | 3 |
| Working Devices | 4 |
| Failed Devices | 0 |
| Spare Devices | 1 |
| Active State | `clean` |

Anggota aktif:

- `/dev/sdd1` — `active sync`
- `/dev/sdd2` — `active sync`
- `/dev/sdd3` — `active sync`

Perangkat cadangan:

- `/dev/sdd4` — `spare`

Status `clean` menunjukkan array dalam keadaan bersih pada saat pemeriksaan. `active sync` menunjukkan partisi aktif sebagai anggota RAID. Spare tersedia sebagai cadangan, bukan anggota aktif.

## 5. Hasil Penilaian Quiz

Jalankan:

```bash
dcomctl grading quiz-19.1
```

Hasil yang diperoleh:

```text
Checking sdd partition: The configuration is correct.
Checking RAID configuration: The configuration is correct.
Checking RAID spare: The configuration is correct.
Checking /etc/mdadm/mdadm.conf: The configuration is correct.

Your Score: 100
Grading completed successfully!
```

**Kesimpulan:** seluruh pemeriksaan berhasil dan quiz mendapatkan nilai **100/100**.

Untuk mengulang lab:

```bash
dcomctl reset-lab quiz-19.1
```

Jalankan reset hanya jika ingin membersihkan atau mengulang lingkungan latihan.

## 6. Ringkasan Perintah

```bash
# Persiapan lab
dcomctl build-lab quiz-19.1

# Memeriksa partisi
lsblk /dev/sdd

# Membuat RAID 1 dengan tiga anggota aktif
sudo mdadm --create /dev/md1 --level=1 --raid-devices=3 \
  /dev/sdd1 /dev/sdd2 /dev/sdd3

# Menambahkan spare
sudo mdadm --add /dev/md1 /dev/sdd4

# Menampilkan konfigurasi array
sudo mdadm --detail --scan

# Memeriksa status array
cat /proc/mdstat
sudo mdadm --detail /dev/md1

# Penilaian
dcomctl grading quiz-19.1
```

## 7. Catatan Keselamatan

- Pastikan target disk benar sebelum mengubah partisi atau membuat array.
- Operasi partisi dan pembuatan RAID dapat mengubah metadata penyimpanan; jangan lakukan pada disk berisi data penting tanpa backup.
- RAID bukan pengganti backup.
- Konfigurasi `mdadm` dan lokasi file dapat berbeda antar-distribusi. Ikuti lokasi yang diminta oleh lab.
