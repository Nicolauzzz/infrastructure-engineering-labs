# Hierarki Sistem Berkas Linux

## Gambaran Umum

Materi ini membahas struktur sistem berkas Linux, standar Filesystem Hierarchy Standard (FHS), serta fungsi direktori penting dalam sistem operasi Linux.

## Konsep Utama

- **Hierarki Sistem Berkas:** Struktur direktori Linux berbentuk pohon yang berawal dari direktori root (`/`).
- **Filesystem Hierarchy Standard (FHS):** Standar yang menjelaskan fungsi dan penempatan direktori dalam sistem Linux.
- **Satu Struktur Sistem Berkas:** Berbagai perangkat penyimpanan dan filesystem dapat diakses melalui satu hierarki direktori menggunakan mekanisme mount.
- **Filesystem Virtual:** `/proc` dan `/sys` menyediakan informasi mengenai proses, kernel, dan perangkat sistem.

## Direktori Penting Linux

| Direktori | Fungsi |
|---|---|
| `/` | Direktori utama yang menjadi akar hierarki sistem berkas |
| `/bin` | Perintah dasar pengguna pada sistem yang menggunakan struktur tradisional |
| `/boot` | Berkas yang dibutuhkan untuk proses boot |
| `/dev` | Berkas perangkat |
| `/etc` | Berkas konfigurasi sistem |
| `/home` | Direktori pribadi pengguna |
| `/root` | Direktori pribadi pengguna root |
| `/lib` dan `/lib64` | Pustaka sistem pada sistem yang menggunakan direktori tersebut |
| `/media` | Titik mount media yang dapat dilepas |
| `/mnt` | Titik mount untuk filesystem |
| `/opt` | Perangkat lunak tambahan |
| `/proc` | Informasi proses dan kernel |
| `/sys` | Informasi perangkat dan subsistem kernel |
| `/sbin` | Perintah administrasi sistem pada struktur tradisional |
| `/srv` | Data yang disediakan oleh layanan sistem |
| `/tmp` | Berkas sementara |
| `/usr` | Program, pustaka, dan sumber daya pendukung |
| `/var` | Data yang berubah, seperti log dan cache |
| `/run` | Data runtime sejak sistem dijalankan |

## Praktik Laboratorium

- **Lab 2.1:** Memeriksa ukuran direktori standar Linux.
- **Lab 2.2:** Menjelajahi filesystem `/proc`.
- **Quiz 2.1–2.3:** Berlatih mengelola isi berkas, direktori, dan melakukan perbandingan berkas.

## Hasil Pembelajaran

Setelah mempelajari materi ini, saya memahami:

- Struktur hierarki sistem berkas Linux.
- Fungsi direktori penting pada Linux.
- Penggunaan `/proc` untuk memeriksa informasi proses dan sistem.
- Dasar pengelolaan serta perbandingan berkas melalui terminal.

## Relevansi dengan Cloud Engineering

Pemahaman filesystem Linux menjadi dasar dalam administrasi server, analisis log, pemecahan masalah, deployment aplikasi, dan pengelolaan infrastruktur berbasis Linux.

## Bukti Praktik

Bukti berupa tangkapan layar dan hasil eksekusi perintah akan ditambahkan sesuai praktik laboratorium yang telah diselesaikan.