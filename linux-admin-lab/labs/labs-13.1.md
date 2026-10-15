# Quiz 13.1 — NFS Server

## Tujuan
Mengonfigurasi NFS server agar dua host memiliki hak akses berbeda terhadap direktori backup, sementara keduanya dapat membaca dan menulis direktori data.

## Instruksi Quiz
1. Konfigurasikan export `/opt/quiz-13.1/backup` untuk host `10.10.10.10` agar dapat menyimpan backup.
2. Export direktori backup yang sama untuk host `20.20.20.20` agar hanya dapat membaca/memulihkan backup.
3. Konfigurasikan `/opt/quiz-13.1/data` agar kedua host dapat membaca dan menulis.
4. Sertakan opsi `insecure` dan pastikan perubahan ditulis ke disk sebelum server merespons klien (`sync`).

## Langkah Pengerjaan

### 1. Bangun lab
```bash
dcomctl build-lab quiz-13.1
```

### 2. Periksa konfigurasi awal
```bash
sudo systemctl status nfs-server
sudo cat /etc/exports
```

Periksa konfigurasi yang sudah ada sebelum mengubahnya.

### 3. Konfigurasikan `/etc/exports`
Buka file:
```bash
sudo nano /etc/exports
```

Pastikan aturan export berikut tersedia:
```text
/opt/quiz-13.1/backup 10.10.10.10(rw,sync,insecure) 20.20.20.20(ro,sync,insecure)
/opt/quiz-13.1/data 10.10.10.10(rw,sync,insecure) 20.20.20.20(rw,sync,insecure)
```

Keterangan opsi:
- `rw`: klien boleh membaca dan menulis.
- `ro`: klien hanya boleh membaca.
- `sync`: perubahan ditulis ke penyimpanan sebelum server mengonfirmasi permintaan.
- `insecure`: mengizinkan koneksi dari port sumber nonprivileged, sesuai persyaratan quiz.

Simpan file dengan `Ctrl+O`, tekan `Enter`, lalu keluar dengan `Ctrl+X`.

### 4. Terapkan konfigurasi export
```bash
sudo exportfs -rav
sudo exportfs -v
```

Pada output `exportfs -v`, opsi tambahan seperti `wdelay`, `hide`, `no_subtree_check`, `sec=sys`, dan `root_squash` dapat muncul secara otomatis sebagai opsi default. Pastikan hak akses untuk setiap host tetap sesuai.

### 5. Pastikan layanan NFS aktif
```bash
sudo systemctl enable --now nfs-server
```

Perintah ini mengaktifkan layanan sekarang sekaligus mengaturnya agar berjalan saat sistem boot, jika layanan tersebut tersedia di lingkungan lab.

### 6. Jalankan penilaian
```bash
dcomctl grading quiz-13.1
```

## Hasil dan Verifikasi

Hasil penilaian menunjukkan semua pemeriksaan konfigurasi berhasil:

- NFS access untuk `10.10.10.10` ke `/opt/quiz-13.1/backup`: **benar**.
- NFS access untuk `20.20.20.20` ke `/opt/quiz-13.1/backup`: **benar**.
- NFS access untuk `10.10.10.10` dan `20.20.20.20` ke `/opt/quiz-13.1/data`: **benar**.
- **Nilai akhir: 100/100**.
- Status grading: `Grading completed successfully!`

## Perintah yang Digunakan
```bash
sudo cat /etc/exports
sudo nano /etc/exports
sudo exportfs -rav
sudo exportfs -v
sudo systemctl enable --now nfs-server
dcomctl grading quiz-13.1
```

## Catatan
Jangan menjalankan `dcomctl reset-lab quiz-13.1` sebelum bukti atau dokumentasi yang diperlukan sudah disimpan, karena reset dapat mengembalikan kondisi lab.

## Ringkasan Pembelajaran
Quiz ini melatih konfigurasi NFS export berdasarkan alamat IP klien dan hak akses. Direktori backup menggunakan `rw` untuk host yang menyimpan backup dan `ro` untuk host yang hanya perlu membaca atau memulihkan backup. Direktori data menggunakan `rw` untuk kedua host. Opsi `sync` dan `insecure` memenuhi persyaratan khusus quiz.
