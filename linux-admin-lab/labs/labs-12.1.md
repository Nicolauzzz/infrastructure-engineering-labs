# Quiz 12.1 — Partitioning

## Tujuan

Membuat partisi pertama berukuran 4 GiB pada `/dev/sdb`, membuat filesystem ext3 dengan label `appstate`, memasangnya pada `/mydata`, dan mengatur agar filesystem otomatis di-mount saat sistem startup.

## 1. Menyiapkan lab

Jalankan perintah:

```bash
dcomctl build-lab quiz-12.1
```

Periksa disk target sebelum melakukan perubahan:

```bash
lsblk
sudo fdisk -l /dev/sdb
```

Pastikan `/dev/sdb` adalah disk yang dimaksud oleh soal.

## 2. Membuat partisi 4 GiB

Buka `fdisk`:

```bash
sudo fdisk /dev/sdb
```

Masukkan perintah berikut sesuai prompt yang muncul:

```text
n
p
1
[Enter]
+4G
p
w
```

Tekan Enter pada pilihan sektor awal agar menggunakan nilai default. Pada versi `fdisk` yang tidak menanyakan tipe partisi, ikuti prompt yang tersedia.

Verifikasi partisi:

```bash
lsblk /dev/sdb
```

Hasil yang diharapkan adalah `/dev/sdb1` dengan ukuran sekitar 4 GiB.

## 3. Membuat filesystem ext3 dan label

```bash
sudo mkfs.ext3 -L appstate /dev/sdb1
```

Periksa tipe filesystem, label, dan UUID:

```bash
sudo blkid /dev/sdb1
```

Pada pengerjaan ini, UUID yang ditampilkan adalah:

```text
e9dac665-0e6e-4783-972a-081df3b9ac7c
```

UUID dapat berbeda ketika lab dibuat ulang; selalu gunakan nilai aktual dari `blkid`.

## 4. Menyiapkan mount point dan memasang filesystem

Pada percobaan awal, perintah mount gagal karena direktori `/mydata` belum ada. Buat direktori tersebut:

```bash
sudo mkdir -p /mydata
```

Kemudian mount filesystem:

```bash
sudo mount /dev/sdb1 /mydata
```

Verifikasi:

```bash
findmnt /mydata
df -hT /mydata
```

Hasil menunjukkan `/dev/sdb1` bertipe `ext3` terpasang pada `/mydata`.

## 5. Mengatur mount otomatis melalui `/etc/fstab`

Buka file konfigurasi:

```bash
sudo nano /etc/fstab
```

Tambahkan satu baris untuk filesystem baru. Pada percobaan pertama, baris menggunakan UUID:

```text
UUID=e9dac665-0e6e-4783-972a-081df3b9ac7c /mydata ext3 defaults 0 2
```

`mount -a` dan `findmnt /mydata` berhasil, tetapi grading menyatakan pemeriksaan entri `/etc/fstab` belum benar. Karena itu, baris tersebut dicoba diganti menjadi:

```text
/dev/sdb1 /mydata ext3 defaults 0 2
```

**Catatan:** Penggunaan `/dev/sdb1` adalah percobaan perbaikan berdasarkan hasil grading, bukan hasil yang sudah terverifikasi memperoleh nilai penuh. Jika lab mensyaratkan UUID atau opsi tertentu, ikuti petunjuk spesifik dari grader.

Simpan di Nano dengan `Ctrl+O`, tekan `Enter`, lalu keluar dengan `Ctrl+X`.

Uji konfigurasi:

```bash
sudo umount /mydata
sudo mount -a
findmnt /mydata
```

Jika `mount -a` tidak menampilkan error dan `findmnt` menunjukkan `/dev/sdb1` pada `/mydata`, konfigurasi mount berhasil secara operasional.

## 6. Grading dan hasil

Jalankan:

```bash
dcomctl grading quiz-12.1
```

Hasil grading pada percobaan pertama:

- Ukuran partisi: benar.
- Tipe filesystem: benar.
- Mount point: benar.
- Entri `/etc/fstab`: belum benar.
- Nilai: **75/100**.

Setelah hasil tersebut, entri `/etc/fstab` diarahkan untuk diuji menggunakan `/dev/sdb1` sebagai sumber. Nilai akhir setelah perubahan ini belum dikonfirmasi dalam dokumentasi ini.

Jika sudah selesai dan ingin mengembalikan lab ke kondisi awal, gunakan:

```bash
dcomctl reset-lab quiz-12.1
```

Jalankan reset hanya setelah semua pemeriksaan dan bukti yang diperlukan selesai disimpan.

## Kendala dan penyelesaian

1. **Mount point tidak ditemukan.** Perintah `mount` gagal karena `/mydata` belum tersedia. Diselesaikan dengan `sudo mkdir -p /mydata`.
2. **UUID placeholder di `/etc/fstab`.** Awalnya entri memakai `UUID_PARTISI`, sehingga `mount -a` gagal. Placeholder diganti dengan UUID aktual hasil `blkid`.
3. **Grading entri `/etc/fstab` gagal.** Walaupun mount berhasil secara operasional, grader memberi nilai 75/100 dan menandai entri `fstab` belum sesuai. Baris kemudian dicoba menggunakan `/dev/sdb1`; hasil grading setelah perubahan perlu diperiksa lagi.

## Perintah penting yang dipelajari

| Perintah | Fungsi |
|---|---|
| `lsblk` | Melihat disk dan partisi dalam bentuk pohon |
| `fdisk` | Membuat dan mengelola tabel partisi |
| `mkfs.ext3 -L` | Membuat filesystem ext3 sekaligus memberi label |
| `blkid` | Melihat tipe filesystem, label, dan UUID |
| `mount` | Memasang filesystem ke direktori |
| `findmnt` | Memverifikasi mount yang aktif |
| `df -hT` | Melihat kapasitas dan tipe filesystem |
| `/etc/fstab` | Mengatur filesystem yang dipasang saat startup |
| `mount -a` | Menguji entri `fstab` dengan mencoba memasang filesystem |

## Catatan keamanan

- Pastikan perangkat target benar sebelum menjalankan `fdisk` atau `mkfs`.
- `mkfs` akan membuat filesystem baru dan dapat merusak data yang sebelumnya ada pada partisi.
- Jangan menyalin UUID dari dokumentasi ini jika lab dibuat ulang; ambil UUID terkini dari `sudo blkid /dev/sdb1`.
- Pastikan tidak ada entri duplikat untuk `/mydata` di `/etc/fstab`.
