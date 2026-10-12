# Quiz 3.1: Configure Resource Limits

## Tujuan

Mengatur batas jumlah proses (*maximum number of processes*) menggunakan
konfigurasi `/etc/security/limits.conf`, dengan ketentuan:

-   Grup `instructor` dibatasi maksimal 15 proses.
-   Soft limit dan hard limit untuk grup memiliki nilai yang sama.
-   User `instructor` dikecualikan dari batas proses grup dan
    menggunakan nilai `unlimited` untuk soft dan hard limit.

## Persiapan

Bangun lingkungan lab:

``` bash
dcomctl build-lab quiz-3.1
```

## Konfigurasi

Edit file konfigurasi:

``` bash
sudo nano /etc/security/limits.conf
```

Tambahkan aturan berikut:

``` text
@instructor    soft    nproc    15
@instructor    hard    nproc    15
instructor     soft    nproc    unlimited
instructor     hard    nproc    unlimited
```

### Penjelasan

  -----------------------------------------------------------------------
  Bagian                              Arti
  ----------------------------------- -----------------------------------
  `@instructor`                       Menerapkan aturan kepada anggota
                                      grup `instructor`

  `instructor`                        Menerapkan aturan khusus kepada
                                      user bernama `instructor`

  `soft`                              Batas lunak untuk resource

  `hard`                              Batas keras untuk resource

  `nproc`                             Batas jumlah proses

  `15`                                Batas proses untuk grup
                                      `instructor`

  `unlimited`                         Tidak menetapkan batas jumlah
                                      proses melalui aturan tersebut
                                      untuk user `instructor`
  -----------------------------------------------------------------------

Aturan khusus user `instructor` memastikan user tersebut tidak terkena
batas 15 proses yang ditetapkan untuk grup.

## Validasi

Jalankan penilaian lab:

``` bash
dcomctl grading quiz-3.1
```

**Hasil:** 100/100 --- konfigurasi dinyatakan benar oleh grader.

## Pembersihan Lab

Setelah penilaian berhasil, lingkungan lab dapat dibersihkan dengan:

``` bash
dcomctl reset-lab quiz-3.1
```

## Ringkasan Pembelajaran

-   File `/etc/security/limits.conf` digunakan untuk mengatur limit
    resource per user atau grup.
-   Awalan `@` menunjukkan nama grup.
-   `nproc` mengatur batas jumlah proses.
-   Soft dan hard limit dapat ditetapkan dengan nilai yang sama.
-   Aturan user yang lebih spesifik dapat digunakan untuk mengecualikan
    user dari batas grup sesuai konfigurasi dan mekanisme penerapan
    limit.

## Bukti

Simpan screenshot hasil grading 100/100 di folder `evidence/` pada
repositori GitHub.
