# Quiz 4.1 --- SIGKILL Signal

## Tujuan

Mempelajari penggunaan sinyal Linux untuk menghentikan proses,
mengidentifikasi file yang digunakan proses, dan menemukan port TCP yang
sedang didengarkan oleh layanan.

## Persiapan Lab

Menyiapkan lab dengan perintah:

``` bash
dcomctl build-lab quiz-4.1
```

Saat proses `xrdp` belum tersedia, pemeriksaan awal menunjukkan lab
belum selesai disiapkan. Perintah `build-lab` sempat gagal ketika
mengunduh paket karena repository Ubuntu `http://id.archive.ubuntu.com`
mengembalikan HTTP 403.

### Kendala repository dan penyelesaian

1.  Memeriksa repository dan menemukan bahwa URL HTTP mirror
    mengembalikan `403 Forbidden`.
2.  Mengganti URL repository pada
    `/etc/apt/sources.list.d/original.list` dari
    `http://id.archive.ubuntu.com/ubuntu` menjadi
    `https://archive.ubuntu.com/ubuntu`.
3.  Saat `apt update` mengalami hash sum mismatch, membersihkan
    cache/list APT dan memperbarui indeks paket lagi.
4.  Menginstal paket `xrdp` setelah pembaruan repository berhasil.
5.  Menjalankan layanan dan memeriksa statusnya:

``` bash
sudo systemctl start xrdp
systemctl status xrdp
```

Layanan akhirnya aktif dan berjalan, dengan proses `xrdp` mendengarkan
pada port TCP `3389`.

## Langkah Pengerjaan

### 1. Memeriksa proses dan file yang digunakan

Menemukan PID proses berdasarkan nama yang tepat:

``` bash
PID=$(pgrep -o -x xrdp)
echo "$PID"
```

PID yang ditemukan saat pengerjaan adalah `11747`. PID dapat berbeda
jika proses dimulai ulang.

Kemudian memeriksa file dan resource terkait proses:

``` bash
sudo lsof -nP -p "$PID"
```

Dari output, `/var/log/xrdp.log` terbuka untuk ditulis (`FD 3w`).
Executable `/usr/sbin/xrdp` ditandai `txt`, sedangkan sejumlah library
ditandai `mem`.

Jawaban audit yang dicoba:

``` text
/var/log/xrdp.log
```

**Hasil:** belum sesuai dengan kriteria grader. Daftar file yang
diharapkan belum berhasil dipastikan, sehingga jawaban ini tidak dicatat
sebagai jawaban final yang benar.

### 2. Memeriksa port TCP

``` bash
sudo lsof -nP -a -p "$PID" -iTCP -sTCP:LISTEN
```

Hasil menunjukkan `xrdp` mendengarkan pada port `3389`. Isi file
`/opt/quiz-4.1/answers/port.txt`:

``` text
3389
```

**Hasil:** benar menurut grader.

### 3. Memperbaiki path direktori jawaban

Awalnya file jawaban dibuat di:

``` text
/opt/quiz41/answers/
```

Namun, grader mencari file di:

``` text
/opt/quiz-4.1/answers/
```

Perbedaannya adalah tanda hubung (`-`) pada nama direktori quiz. Karena
direktori yang benar belum ada, `tee` awalnya menampilkan:

``` text
No such file or directory
```

Penyelesaiannya adalah membuat direktori yang diminta:

``` bash
sudo mkdir -p /opt/quiz-4.1/answers
```

Lalu membuat file jawaban pada path yang benar:

``` bash
sudo tee /opt/quiz-4.1/answers/file_audit.txt <<'EOF'
/var/log/xrdp.log
EOF

echo 3389 | sudo tee /opt/quiz-4.1/answers/port.txt
```

Path berhasil diperbaiki. Pemeriksaan port lolos, tetapi isi audit file
tetap ditolak grader.

### 4. Mengirim SIGKILL

Proses dihentikan secara paksa dengan:

``` bash
sudo kill -KILL "$PID"
```

Alternatifnya, `sudo kill -9 "$PID"` menggunakan nomor sinyal yang sama.

**Hasil:** benar menurut grader; proses `xrdp` tidak ditemukan saat
penilaian.

## Hasil Penilaian

Penilaian dijalankan dengan:

``` bash
dcomctl grading quiz-4.1
```

  Komponen                   Hasil
  -------------------------- ------------------
  File audit                 Belum sesuai
  Port TCP                   Benar --- `3389`
  Proses `xrdp` dihentikan   Benar
  Nilai akhir                **66/100**

## Ringkasan Masalah dan Solusi

  -----------------------------------------------------------------------
  Masalah                             Penyelesaian
  ----------------------------------- -----------------------------------
  Repository Ubuntu mengembalikan     Mengganti mirror HTTP ke HTTPS,
  HTTP 403 saat lab disiapkan         membersihkan daftar/cache APT saat
                                      terjadi hash mismatch, lalu
                                      memperbarui indeks dan memasang
                                      `xrdp`

  Proses `xrdp` belum tersedia pada   Memastikan paket terpasang, lalu
  pemeriksaan awal                    menjalankan layanan dengan
                                      `systemctl`

  File jawaban tidak dapat dibuat     Membuat direktori jawaban dengan
  karena direktori tidak ditemukan    `sudo mkdir -p`

  Grader tidak menemukan file jawaban Memperbaiki path dari
                                      `/opt/quiz41/answers/` menjadi
                                      `/opt/quiz-4.1/answers/`

  Audit file masih gagal              Belum terselesaikan; perlu meneliti
                                      daftar file yang diharapkan grader.
                                      Jangan menganggap
                                      `/var/log/xrdp.log` sebagai jawaban
                                      benar karena hasil grading
                                      menolaknya
  -----------------------------------------------------------------------

## Pembelajaran

-   `pgrep -x` membantu menemukan PID berdasarkan nama proses secara
    tepat.
-   `lsof -p PID` menampilkan file dan resource yang terkait dengan
    proses.
-   `lsof -iTCP -sTCP:LISTEN` membantu memeriksa port TCP yang sedang
    didengarkan.
-   `kill -KILL` atau `kill -9` mengirimkan SIGKILL untuk menghentikan
    proses secara paksa.
-   Path file harus sama persis dengan lokasi yang ditentukan instruksi
    dan grader.
-   Ketika repository paket bermasalah, periksa status HTTP mirror dan
    lakukan pembaruan indeks APT setelah memperbaiki sumber paket.
-   Hasil grader penting untuk membedakan langkah yang berhasil dari
    jawaban yang masih perlu diperbaiki.

------------------------------------------------------------------------

*Lab: Linux Administration --- Quiz 4.1*
