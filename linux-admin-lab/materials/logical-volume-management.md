# Module 18 — Logical Volume Management (LVM)

## 1. Tujuan Pembelajaran

Setelah mempelajari modul ini, kamu diharapkan mampu:

- Menjelaskan konsep Logical Volume Management (LVM).
- Membuat Logical Volume (LV).
- Menampilkan informasi PV, VG, dan LV.
- Mengubah ukuran Logical Volume beserta filesystem-nya.
- Membuat dan menghapus snapshot LVM.

## 2. Konsep Dasar LVM

LVM membuat pengelolaan kapasitas penyimpanan lebih fleksibel daripada mengandalkan partisi fisik secara langsung.

Struktur LVM:

```text
Disk/Partisi → PV → VG → LV → Filesystem → Mount Point
```

- **PV (Physical Volume):** disk atau partisi yang disiapkan untuk digunakan oleh LVM.
- **VG (Volume Group):** kumpulan satu atau lebih PV yang menyediakan kumpulan kapasitas penyimpanan.
- **LV (Logical Volume):** volume logis yang dialokasikan dari kapasitas VG.
- **Extent:** unit alokasi di dalam VG. Ukuran default yang dijelaskan pada materi adalah 4 MiB, tetapi ukurannya dapat ditentukan saat VG dibuat.

Filesystem seperti ext4 dapat dibuat di atas LV, kemudian dipasang (*mount*) ke direktori Linux.

## 3. Perintah Utama LVM

### Physical Volume (PV)

| Perintah | Fungsi |
|---|---|
| `pvcreate` | Menyiapkan disk/partisi sebagai PV |
| `pvdisplay` atau `pvs` | Menampilkan informasi PV |
| `pvmove` | Memindahkan data dari satu PV ke PV lain di dalam VG |
| `pvremove` | Menghapus label PV setelah PV tidak lagi digunakan oleh VG |

### Volume Group (VG)

| Perintah | Fungsi |
|---|---|
| `vgcreate` | Membuat VG |
| `vgextend` | Menambahkan PV ke VG |
| `vgreduce` | Mengeluarkan PV dari VG setelah datanya dipindahkan |
| `vgdisplay` atau `vgs` | Menampilkan informasi VG |

### Logical Volume (LV)

| Perintah | Fungsi |
|---|---|
| `lvcreate` | Membuat LV |
| `lvdisplay` atau `lvs` | Menampilkan informasi LV |
| `lvextend` | Memperbesar LV |
| `lvresize` | Mengubah ukuran LV |
| `lvremove` | Menghapus LV |

## 4. Membuat Logical Volume

Urutan umum membuat LV:

1. Siapkan partisi yang memang akan digunakan untuk LVM.
2. Buat PV dari partisi tersebut.
3. Buat VG dari satu atau beberapa PV.
4. Alokasikan LV dari VG.
5. Buat filesystem pada LV.
6. Buat direktori mount point dan mount LV.
7. Atur `/etc/fstab` jika filesystem harus dipasang otomatis saat boot.

Contoh perintah berikut hanya ilustrasi. Pastikan nama disk dan partisi benar serta tidak berisi data penting sebelum menjalankannya.

```bash
sudo pvcreate /dev/sdb1
sudo pvcreate /dev/sdc1

sudo vgcreate vgdata /dev/sdb1
sudo vgextend vgdata /dev/sdc1

sudo lvcreate -L 1G -n lvdata vgdata
sudo mkfs.ext4 /dev/vgdata/lvdata

sudo mkdir -p /data
sudo mount /dev/vgdata/lvdata /data
```

Untuk membuat mount persisten, tambahkan entri yang sesuai ke `/etc/fstab`. Penggunaan UUID filesystem umumnya lebih stabil daripada bergantung pada nama perangkat.

## 5. Menampilkan Informasi LVM

Perintah ringkas:

```bash
sudo pvs
sudo vgs
sudo lvs
```

Perintah dengan informasi lebih terperinci:

```bash
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay
```

Untuk mengetahui kapasitas filesystem dan mount point:

```bash
df -h /data
```

Selalu periksa nama PV, VG, dan LV yang benar sebelum melakukan perubahan kapasitas.

## 6. Mengubah Ukuran Logical Volume

Opsi `-r` pada `lvextend` atau `lvresize` meminta LVM menyesuaikan filesystem bersama perubahan ukuran LV, jika filesystem dan konfigurasi mendukungnya.

Contoh menambahkan 100 MiB:

```bash
sudo lvextend -r -L +100M /dev/vgdata/lvdata
```

Perbedaan penting:

- `-L +2G` berarti **menambahkan** 2 GiB ke ukuran LV saat ini.
- `-L 2G` berarti menetapkan ukuran total LV menjadi 2 GiB.

Filesystem yang didukung, seperti ext4, umumnya dapat diperbesar ketika masih ter-mount. Pengecilan filesystem lebih berisiko: filesystem harus diperkecil terlebih dahulu sebelum LV diperkecil. Ikuti prosedur khusus filesystem dan buat backup sebelum melakukan operasi pengecilan.

Untuk mengeluarkan PV dari VG, pindahkan dahulu extent yang masih digunakan, lalu keluarkan PV:

```bash
sudo pvmove /dev/sdc1
sudo vgreduce vgdata /dev/sdc1
```

Jangan jalankan contoh ini tanpa memeriksa kondisi storage dan tujuan operasi.

## 7. LVM Snapshot

Snapshot LVM merekam keadaan volume pada suatu titik waktu. Snapshot klasik menggunakan mekanisme *copy-on-write*: blok data lama dicatat ketika data pada volume asal berubah.

Snapshot dapat membantu backup konsisten atau pengujian, tetapi **bukan pengganti backup independen**. Kapasitas snapshot perlu dipantau; jika ruang snapshot klasik habis, snapshot dapat menjadi tidak valid.

Contoh membuat snapshot berukuran 512 MiB:

```bash
sudo lvcreate -L 512M -s -n lvdata-snap /dev/vgdata/lvdata
sudo mkdir -p /mnt/lvdata-snap
sudo mount -o ro /dev/vgdata/lvdata-snap /mnt/lvdata-snap
```

Setelah selesai digunakan:

```bash
sudo umount /mnt/lvdata-snap
sudo lvremove /dev/vgdata/lvdata-snap
```

## 8. Quiz 18.1 — Extend LVM Filesystem

### Soal

Filesystem berbasis LVM yang ter-mount pada `/lvm-extend` hampir penuh. Tambahkan **2 GB** dari Volume Group yang tersedia tanpa melakukan unmount filesystem.

Instruksi soal menyebut VG bernama `system`. Namun, pada lingkungan lab yang dikerjakan, hasil pemeriksaan menunjukkan nama VG dan LV yang berbeda. Karena itu, nama target harus diverifikasi sebelum menjalankan perintah.

### Persiapan lab

```bash
dcomctl build-lab quiz-18.1
```

### Pemeriksaan awal

Periksa filesystem:

```bash
df -h /lvm-extend
```

Periksa Logical Volume:

```bash
sudo lvs
```

Periksa Volume Group yang ditemukan:

```bash
sudo vgs pool-1
```

Pada lab yang dikerjakan, hasil pemeriksaan menunjukkan:

- Mount point: `/lvm-extend`
- VG: `pool-1`
- LV: `LVM-backend`
- Ukuran awal LV: `1.00g`
- Ruang kosong VG: kurang dari `9.00g`

Perintah `sudo vgs system` menghasilkan pesan bahwa VG `system` tidak ditemukan. Nama target yang digunakan harus mengikuti hasil pemeriksaan aktual.

### Perintah penyelesaian

```bash
sudo lvextend -r -L +2G /dev/pool-1/LVM-backend
```

Penjelasan:

- `lvextend`: memperbesar Logical Volume.
- `-r`: menyesuaikan ukuran filesystem bersama ukuran LV.
- `-L +2G`: menambahkan 2 GiB pada ukuran yang ada.
- `/dev/pool-1/LVM-backend`: LV yang digunakan oleh `/lvm-extend` pada lab ini.

Filesystem tidak perlu di-unmount untuk operasi pembesaran yang didukung ini.

### Verifikasi

```bash
sudo lvs
df -h /lvm-extend
```

Setelah perintah berhasil, ukuran LV `LVM-backend` berubah dari `1.00g` menjadi `3.00g`.

Jalankan penilaian:

```bash
dcomctl grading quiz-18.1
```

Hasil yang diperoleh:

```text
Checking Extend LVM Filesystem: The configuration is correct.
Your Score: 100
Grading completed successfully!
```

**Hasil quiz: 100/100.**

Untuk mengatur ulang lab setelah selesai, perintah yang tersedia adalah:

```bash
dcomctl reset-lab quiz-18.1
```

Jalankan reset hanya jika memang ingin membersihkan atau mengulang lingkungan lab.

## 9. Catatan Keselamatan

- Pastikan disk dan partisi yang dipilih memang disiapkan untuk LVM.
- Periksa `pvs`, `vgs`, `lvs`, dan `df` sebelum mengubah kapasitas.
- Jangan menganggap nama VG/LV akan sama pada semua lingkungan.
- Periksa kapasitas kosong sebelum memperbesar LV.
- Operasi partisi, format filesystem, penghapusan LV, dan pengecilan filesystem dapat menyebabkan kehilangan data jika salah dilakukan.
