# Linux Disk Management dan Partitioning

Materi Linux Administration — Module 12

## 12.1 Learning Objectives

Setelah mempelajari materi ini, kita diharapkan mampu:

- Membedakan jenis disk dan data bus yang umum digunakan.
- Menjelaskan geometri disk serta konsep partisi.
- Memahami penamaan perangkat disk Linux dan device node.
- Memilih strategi partisi yang sesuai.
- Menggunakan `fdisk`, `blkid`, dan `lsblk`.
- Melakukan backup dan restore partition table dengan hati-hati.

## 12.2 Common Disk Types

### SATA (Serial Advanced Technology Attachment)
Antarmuka yang menggantikan IDE. SATA menggunakan kabel lebih ringkas, mendukung hot swapping pada perangkat yang mendukungnya, dan umum digunakan pada komputer maupun server.

### SCSI (Small Computer System Interface)
Keluarga antarmuka penyimpanan yang memiliki berbagai generasi dan kecepatan. Perangkat SATA dan USB dapat ditampilkan Linux melalui lapisan perangkat SCSI.

### SAS (Serial Attached SCSI)
Antarmuka point-to-point yang banyak digunakan pada server. SAS umumnya dirancang untuk kebutuhan enterprise dan fitur keandalan yang lebih tinggi dibanding SATA konsumen.

### USB (Universal Serial Bus)
Digunakan oleh media eksternal seperti flash drive dan hard disk eksternal. Perangkat penyimpanan USB sering muncul sebagai perangkat blok dengan nama `/dev/sdX`.

### SSD (Solid State Drive)
Menggunakan memori flash tanpa komponen mekanis yang berputar. SSD umumnya memiliki latensi lebih rendah, transfer data lebih cepat, dan konsumsi daya lebih rendah daripada hard disk mekanis.

### IDE/EIDE
Antarmuka penyimpanan lama yang kini sudah usang. Perangkat lama dapat menggunakan nama seperti `/dev/hda`.

## 12.3 Disk Geometry

Pada hard disk mekanis, data secara historis dijelaskan menggunakan konsep platter, head, track, sector, dan cylinder. Istilah ini membantu memahami cara penyimpanan disk tradisional, walaupun detail fisik modern banyak disembunyikan oleh kontroler.

- **Sector**: unit penyimpanan terkecil yang dialamatkan oleh perangkat.
- **Track**: jalur melingkar pada platter.
- **Cylinder**: kumpulan track yang berada pada posisi radial sama di beberapa platter.
- **Logical sector**: ukuran sektor yang dilihat sistem operasi.
- **Physical sector**: ukuran sektor fisik pada media.

Ukuran sektor logis sering 512 byte, sedangkan sektor fisik dapat berukuran 4096 byte. SSD tidak memiliki komponen mekanis seperti platter atau head, sehingga geometri klasik tidak menggambarkan struktur fisiknya.

Untuk memeriksa informasi disk dan sektor:

```bash
sudo fdisk -l
sudo fdisk -l /dev/sda | grep -i sector
```

Ganti `/dev/sda` dengan perangkat yang benar pada sistem. Selalu periksa nama disk terlebih dahulu.

## 12.4 Partition Organization: MBR dan GPT

Partisi membagi satu disk menjadi beberapa wilayah logis yang dapat diformat atau digunakan untuk tujuan berbeda.

### MBR (Master Boot Record)
- Skema partisi lama.
- Mendukung maksimal empat entri partisi primer.
- Salah satu entri dapat berupa extended partition yang menampung logical partition.
- Memiliki batas ukuran disk sekitar 2 TiB pada skema umum dengan sektor 512 byte.

### GPT (GUID Partition Table)
- Skema modern yang umum digunakan bersama sistem UEFI.
- Mendukung banyak partisi; implementasi umum memungkinkan hingga 128 entri.
- Tidak memerlukan extended/logical partition.
- Menyimpan metadata GPT di awal dan akhir disk untuk membantu pemulihan.

GPT biasanya menjadi pilihan untuk sistem modern. Dukungan dan batas aktual tetap bergantung pada sistem operasi, firmware, dan alat yang digunakan.

## 12.5 Why Partition?

Beberapa alasan disk dibagi menjadi beberapa partisi:

- Memisahkan file sistem operasi dari data pengguna.
- Mempermudah backup, pemulihan, atau pengelolaan kapasitas.
- Menerapkan kebijakan keamanan dan kuota yang berbeda.
- Memisahkan data yang sering berubah dari data yang relatif stabil.
- Menyediakan area swap.
- Memisahkan kebutuhan boot, root filesystem (`/`), dan home (`/home`).

Contoh layout yang dapat ditemui adalah partisi EFI/boot, root (`/`), swap, dan `/home`. Tidak semua instalasi membutuhkan partisi terpisah untuk setiap fungsi. Rencanakan kapasitas sejak awal karena mengubah ukuran partisi setelah sistem digunakan dapat lebih rumit.

## 12.6 MBR Partition Table

MBR berada pada sektor pertama disk. Secara konvensional, sektor ini memuat kode boot, tabel partisi dengan empat entri, dan signature `0x55AA`.

Setiap entri partisi mencatat informasi seperti tipe partisi, sektor awal, dan jumlah sektor. Linux modern menggunakan pengalamatan blok logis (LBA), bukan mengandalkan geometri CHS lama.

## 12.7 GPT Partition Table

GPT menyimpan header dan metadata partisi pada awal disk serta salinan cadangan di akhir disk. GPT juga menyediakan identitas unik untuk disk dan partisi.

Perintah untuk memeriksa informasi partisi dan filesystem:

```bash
sudo fdisk -l
sudo blkid
```

Perbedaan penting:

- **UUID** mengidentifikasi filesystem. Nilainya dapat berubah ketika filesystem diformat ulang.
- **PARTUUID** mengidentifikasi partisi. Biasanya tetap sama ketika filesystem di dalam partisi diformat ulang, selama partisinya tidak dibuat ulang.

Jangan mengubah skema MBR ke GPT tanpa backup dan rencana pemulihan yang jelas. Kesalahan partisi dapat membuat data atau sistem tidak dapat diakses.

## 12.8 Naming Disk Devices and Device Nodes

Linux menyediakan device node untuk perangkat blok di direktori `/dev`. Nama yang umum antara lain:

- `/dev/sda`: disk pertama yang menggunakan penamaan `sd`.
- `/dev/sdb`: disk berikutnya.
- `/dev/sdb1`: partisi pertama pada `/dev/sdb`.
- `/dev/sdc4`: partisi keempat pada `/dev/sdc`.

Huruf perangkat dapat berubah tergantung urutan deteksi perangkat. Karena itu, jangan mengasumsikan bahwa sebuah disk selalu memakai nama tertentu; verifikasi dengan `lsblk`, `blkid`, atau identifier persisten.

Perintah pemeriksaan:

```bash
ls -l /dev/sd*
lsblk
```

**Peringatan:** akses langsung atau penulisan ke device node dapat merusak filesystem dan seluruh data pada disk jika perangkat atau perintah salah dipilih.

## 12.9 `blkid`

`blkid` menampilkan atribut perangkat blok yang dikenali, misalnya label, UUID, tipe filesystem, dan PARTUUID.

```bash
sudo blkid
sudo blkid /dev/sda1
sudo blkid -L root
```

Opsi `-L` mencari perangkat berdasarkan label filesystem. Partisi kosong atau yang belum memiliki metadata filesystem mungkin tidak menampilkan semua atribut.

## 12.10 `lsblk`

`lsblk` menampilkan perangkat blok dalam bentuk pohon sehingga disk, partisi, dan relasinya mudah dilihat.

```bash
lsblk
lsblk -f
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
lsblk -p
```

- `-f`: menampilkan informasi filesystem.
- `-o`: memilih kolom output.
- `-p`: menampilkan path perangkat lengkap.

Gunakan `lsblk -f` sebagai pemeriksaan awal sebelum melakukan operasi partisi atau filesystem.

## 12.11 Sizing Up Partitions

### Root (`/`)
Root filesystem memuat struktur utama sistem operasi dan dapat mencakup direktori sistem maupun aplikasi. Banyak sistem memakai beberapa filesystem yang dipasang pada mount point berbeda. LVM dapat memberi fleksibilitas tambahan untuk mengelola volume, tetapi tetap perlu perencanaan.

### Swap
Swap menyediakan ruang pendukung ketika kebutuhan memori melebihi RAM yang tersedia dan dapat digunakan untuk hibernasi pada konfigurasi tertentu. Ukuran swap yang tepat bergantung pada kapasitas RAM, beban kerja, kebutuhan hibernasi, dan kebijakan distribusi. Menambah swap tidak selalu menyelesaikan kekurangan RAM.

Swap dapat berupa partisi khusus atau swap file, tergantung distribusi dan konfigurasi.

## 12.12 Backing Up and Restoring Partition Tables

Backup tabel partisi harus dilakukan sebelum mengubah partisi. Perintah berikut hanya contoh dan harus disesuaikan dengan disk yang benar.

### MBR: backup sektor pertama

```bash
sudo dd if=/dev/sda of=mbrbackup bs=512 count=1
```

Pemulihan contoh:

```bash
sudo dd if=mbrbackup of=/dev/sda bs=512 count=1
```

**Peringatan penting:** `dd` dapat menimpa data secara permanen. Salah menentukan `if`, `of`, atau perangkat target bisa menghancurkan data. Simpan backup di media yang berbeda dan pastikan perangkat target dengan teliti. Backup satu sektor MBR tidak mencakup semua kemungkinan metadata partisi di luar sektor tersebut.

### GPT: gunakan `sgdisk`

Untuk memeriksa tabel GPT:

```bash
sudo sgdisk -p /dev/sda
```

Backup tabel GPT ke file dapat dilakukan dengan:

```bash
sudo sgdisk --backup=partition-table.gpt /dev/sda
```

Pulihkan hanya setelah memastikan target disk dan file backup benar:

```bash
sudo sgdisk --load-backup=partition-table.gpt /dev/sda
```

Jangan menjalankan perintah restore pada disk produksi tanpa prosedur pemulihan dan verifikasi yang memadai.

## 12.13 Partition Table Editors

| Utilitas | Kegunaan |
|---|---|
| `fdisk` | Editor tabel partisi berbasis menu; umum digunakan untuk MBR dan juga mendukung GPT pada versi modern. |
| `sfdisk` | Alat partisi berbasis command line yang cocok untuk skrip. |
| `parted` | Membuat, menghapus, dan mengelola partisi. |
| `gparted` | Antarmuka grafis untuk `parted`. |
| `gdisk` | Editor GPT interaktif; juga dapat mengenali skema MBR. |
| `sgdisk` | Antarmuka command line untuk operasi GPT dan otomasi. |

Sebelum mengubah tabel partisi, catat kondisi awal atau buat backup terlebih dahulu.

## 12.14 Using `fdisk`

`fdisk` dapat digunakan untuk melihat dan mengelola tabel partisi. Untuk menampilkan tabel tanpa masuk ke mode interaktif:

```bash
sudo fdisk -l
sudo fdisk -l /dev/sdb
```

Untuk membuka editor interaktif pada disk tertentu:

```bash
sudo fdisk /dev/sdb
```

Perintah utama dalam mode interaktif:

| Tombol | Fungsi |
|---|---|
| `m` | Menampilkan bantuan/menu. |
| `p` | Menampilkan tabel partisi. |
| `n` | Membuat partisi baru. |
| `d` | Menghapus partisi. |
| `t` | Mengubah tipe partisi. |
| `w` | Menulis perubahan ke disk dan keluar. |
| `q` | Keluar tanpa menyimpan perubahan. |

Perubahan biasanya baru ditulis ketika `w` dipilih. Periksa ulang tabel dengan `p` sebelum menyimpan. Setelah perubahan, sistem mungkin perlu membaca ulang tabel partisi:

```bash
sudo partprobe -s
cat /proc/partitions
```

Jika sistem masih menggunakan partisi atau filesystem yang sedang diubah, reboot mungkin diperlukan. Jangan memformat atau mengubah partisi aktif secara sembarangan.

## Ringkasan Perintah

| Tujuan | Perintah |
|---|---|
| Melihat disk dan partisi | `lsblk` |
| Melihat filesystem, UUID, dan mount point | `lsblk -f` |
| Melihat atribut filesystem | `sudo blkid` |
| Melihat tabel partisi | `sudo fdisk -l` |
| Memeriksa tabel GPT | `sudo sgdisk -p /dev/sda` |
| Melihat partisi yang dikenali kernel | `cat /proc/partitions` |
| Meminta kernel membaca ulang tabel partisi | `sudo partprobe -s` |

## Catatan Praktik dan Keamanan

1. Jalankan `lsblk -f` dan `sudo fdisk -l` sebelum memilih perangkat.
2. Pastikan target benar; nama `/dev/sda` atau `/dev/sdb` tidak boleh ditebak.
3. Buat dan verifikasi backup sebelum mengubah tabel partisi.
4. Jangan menjalankan `mkfs`, `dd`, atau perintah penghapusan pada perangkat yang belum dipastikan.
5. Praktikkan perubahan partisi pada lab atau disk uji, bukan pada disk sistem utama.
