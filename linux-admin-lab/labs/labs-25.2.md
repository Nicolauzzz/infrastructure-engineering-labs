# Quiz 25.2 — Find Files

## Tujuan
Menggunakan `find` untuk mencari file berdasarkan bit permission khusus, mode, ukuran, ekstensi, dan pemilik.

## Persiapan
```bash
dcomctl build-lab quiz-25.2
```

## Tugas 1 — File dengan SUID dan SGID

Soal meminta path absolut semua file di `/data/folder1` yang memiliki SUID dan SGID sekaligus, ditulis satu path per baris ke `/opt/quiz-25.2/audit.txt`.

```bash
sudo find /data/folder1 -type f -perm -6000 -print | sudo tee /opt/quiz-25.2/audit.txt
```

- `-type f`: hanya file biasa.
- `-perm -6000`: kedua bit khusus harus aktif, SUID (`4000`) dan SGID (`2000`).
- `-print`: menampilkan path file.
- `tee`: menulis hasil ke file audit.

Hasil pencarian yang terlihat:
```text
/data/folder1/random-data-151
```

Pemeriksaan tambahan:
```bash
sudo find /data/folder1 -type f -perm -6000 -ls
ls -l /data/folder1/random-data-151
stat -c '%A %a %n' /data/folder1/random-data-151
cat -A /opt/quiz-25.2/audit.txt
```

File tersebut menampilkan permission `-rwsr-sr-x` dan mode oktal `7664`, sehingga bit SUID dan SGID tampak aktif. Path di `audit.txt` juga diperiksa sebagai satu baris.

**Hasil tugas 1:** Belum lolos pemeriksaan grader meskipun hasil pencarian dan format path telah diperiksa. Penyebab pastinya belum diketahui.

## Tugas 2 — Menyalin file dengan mode tepat `0600`

```bash
sudo find /data/folder1 -type f -perm 0600 -exec cp -p {} /opt/quiz-25.2/permissions/ \;
```

`-perm 0600` mencari permission tepat `rw-------`. `cp -p` mempertahankan permission dan metadata yang didukung.

**Hasil grading:** Berhasil.

## Tugas 3 — Menyalin file berukuran tepat 8 byte

```bash
sudo find /data/folder1 -type f -size 8c -exec cp {} /opt/quiz-25.2/size/ \;
```

`-size 8c` berarti ukuran file tepat 8 byte.

**Hasil grading:** Berhasil; grader mengonfirmasi ukuran 8 byte.

## Tugas 4 — File `.txt` yang bukan milik `root`

```bash
sudo find /data/folder1 -type f -name '*.txt' ! -user root -print > /opt/quiz-25.2/extension.txt
```

- `-name '*.txt'`: nama file berakhiran `.txt`.
- `! -user root`: pemilik file bukan `root`.
- `-print`: menulis path hasil pencarian ke file output.

**Hasil grading:** Berhasil.

## Hasil Grading

```text
Checking /opt/quiz-25.2/audit.txt: The configuration is incorrect.
Permission check: The configuration is correct.
File size check: The size is correct (8 bytes).
File extension check: The configuration is correct.

There is a configuration mismatch
Your Score: 75
Grading completed successfully!
```

**Skor akhir: 75/100.** Tugas 2, 3, dan 4 lolos. Tugas 1 belum lolos. Pemeriksaan lanjutan hanya menemukan `/data/folder1/random-data-151` dengan kedua bit khusus yang tampak aktif, tetapi alasan grader menolaknya belum dapat dipastikan. Dokumentasi ini mencatat hasil aktual dan tidak mengklaim nilai penuh.

## Konsep Penting

- SUID: bit khusus `4000`.
- SGID: bit khusus `2000`.
- `find -perm -6000`: mensyaratkan kedua bit aktif.
- `find -perm 0600`: mencocokkan mode permission tepat `0600`.
- `find -size 8c`: ukuran tepat 8 byte.
- `find -name '*.txt' ! -user root`: file `.txt` yang bukan milik `root`.
- Gunakan path absolut ketika diminta oleh soal.

Jika tidak ingin melakukan troubleshooting lagi, lab dapat dibersihkan dengan:
```bash
dcomctl reset-lab quiz-25.2
```
Jangan reset jika masih ingin menyelidiki tugas pertama.
