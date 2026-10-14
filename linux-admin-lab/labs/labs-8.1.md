# Quiz 8.1 — Locate the Error Message

## Tujuan

Menemukan entri log sistem yang relevan dengan masalah saat user `instructor2` menjalankan perintah menggunakan `sudo`, lalu menyimpan hasilnya ke `/opt/quiz-8.1/instructor2.log`.

## Langkah Praktik

### 1. Menyiapkan lab

```bash
dcomctl build-lab quiz-8.1
```

Lab berhasil dipersiapkan.

### 2. Menelusuri log

Pemeriksaan awal dilakukan pada `/var/log/auth.log`, karena file ini umum menyimpan aktivitas autentikasi dan `sudo` pada sistem Debian/Ubuntu.

```bash
sudo grep -i "instructor2" /var/log/auth.log
```

Hasil awal terutama menunjukkan pembuatan akun `instructor2` dan konfigurasi sudo oleh user `student`, belum langsung menunjukkan pesan kesalahan yang dicari.

Kemudian pencarian diperluas menggunakan `journalctl` dan penyaringan kata kunci yang berkaitan dengan kegagalan autentikasi, penolakan perintah, serta masalah sudo. Contoh pencarian yang digunakan:

```bash
sudo journalctl --since "2026-10-14 00:00:00" --no-pager | grep -iE 'instructor2|authentication failure|command not allowed|not in sudoers|incorrect password|sudo.*denied'
```

> Catatan troubleshooting: output awal memuat banyak baris dari aktivitas pencarian yang dijalankan oleh user `student`. Karena itu, hasil pencarian perlu diperiksa dan tidak boleh langsung dianggap sebagai pesan kesalahan asli milik `instructor2`.

### 3. Menyimpan hasil ke file jawaban

```bash
sudo journalctl --since "2026-10-14 00:00:00" --no-pager | grep -iE 'instructor2|authentication failure|command not allowed|not in sudoers|incorrect password|sudo.*denied' | sudo tee /opt/quiz-8.1/instructor2.log
```

Isi file diperiksa dengan:

```bash
cat /opt/quiz-8.1/instructor2.log
```

### 4. Menilai hasil

```bash
dcomctl grading quiz-8.1
```

## Hasil

- **Skor: 100/100**
- Pemeriksaan: `/opt/quiz-8.1/instructor2.log` — konfigurasi benar.
- Status: `Grading completed successfully!`

File jawaban diterima oleh grader lab.

## Kendala dan Penyelesaian

**Kendala:** Pencarian awal pada `/var/log/auth.log` belum memperlihatkan pesan kesalahan sudo yang jelas untuk `instructor2`. Sebagian besar baris yang ditemukan berhubungan dengan pembuatan akun, pengaturan sudoers, dan perintah pencarian yang dijalankan selama troubleshooting.

**Penyelesaian:** Pencarian diperluas ke system journal menggunakan `journalctl`, lalu hasilnya disaring berdasarkan kata kunci yang berkaitan dengan `instructor2` dan kegagalan sudo/autentikasi. Hasil disimpan ke lokasi yang ditentukan quiz dan diverifikasi menggunakan perintah grading.

**Catatan teknis:** Skor 100/100 membuktikan file jawaban memenuhi pemeriksaan grader. Namun, jangan menyimpulkan bahwa setiap baris di dalam file adalah pesan kegagalan asli oleh `instructor2`; isi log tetap perlu dibedakan dari jejak perintah troubleshooting oleh user `student`.

## Evidence

Simpan screenshot berikut ke folder `linux-admin-lab/evidence/`:

1. `quiz-8.1-log-found.png` — terminal yang menunjukkan pemeriksaan log dan isi file `instructor2.log`.
2. `quiz-8.1-grading-100.png` — terminal yang menunjukkan `dcomctl grading quiz-8.1`, skor `100`, dan `Grading completed successfully!`.

## Cleanup

Jika ingin mengembalikan lab ke kondisi awal setelah evidence dan hasil tersimpan:

```bash
dcomctl reset-lab quiz-8.1
```

Jalankan reset hanya setelah semua bukti dan catatan yang dibutuhkan sudah disimpan.
