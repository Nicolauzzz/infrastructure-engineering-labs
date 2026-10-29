# Quiz 24.2 — Increase The Server Security

## Tujuan

Meningkatkan keamanan server dengan:
1. Mengubah password akun lokal `root`.
2. Membuat group lokal `administrators`.
3. Menambahkan `lucas`, `john`, dan `paul` ke group tersebut tanpa mengubah primary group.
4. Mengizinkan `lucas` menjalankan perintah apa pun sebagai root melalui `sudo` tanpa prompt password.

## Langkah Pengerjaan

### 1. Persiapan lab

```bash
dcomctl build-lab quiz-24.2
```

### 2. Mengubah password `root`

```bash
sudo passwd root
```

Password yang diminta oleh soal:

```text
bbAA22dd
```

Masukkan password tersebut dua kali saat diminta. Input password di terminal Linux tidak menampilkan karakter apa pun.

**Catatan hasil:** Walaupun password `bbAA22dd` sudah dimasukkan, grading melaporkan bahwa password root salah. Karena itu, bagian ini belum dinyatakan berhasil.

Kemungkinan penyebab yang bisa diperiksa bila ingin mencoba lagi:
- Pastikan tidak ada salah ketik, termasuk huruf besar/kecil.
- Pastikan password baru benar-benar dikonfirmasi oleh `passwd` dengan pesan bahwa password berhasil diperbarui.
- Jalankan `sudo passwd root` kembali dan masukkan password sesuai soal secara teliti.
- Setelah itu jalankan grading ulang.

### 3. Membuat group `administrators`

```bash
sudo groupadd administrators
```

Memeriksa group:

```bash
getent group administrators
```

### 4. Menambahkan user ke group

```bash
sudo usermod -aG administrators lucas
sudo usermod -aG administrators john
sudo usermod -aG administrators paul
```

Gunakan `-aG` supaya keanggotaan supplementary group yang sudah ada tetap dipertahankan dan primary group tidak berubah.

Verifikasi:

```bash
id lucas
id john
id paul
```

### 5. Memberi izin sudo tanpa password kepada `lucas`

Buat atau edit file konfigurasi khusus menggunakan `visudo`:

```bash
sudo visudo -f /etc/sudoers.d/lucas
```

Isi file:

```text
lucas ALL=(ALL:ALL) NOPASSWD: ALL
```

Simpan file, kemudian periksa sintaks konfigurasi sudo:

```bash
sudo visudo -c
```

Verifikasi aturan untuk `lucas`:

```bash
sudo -l -U lucas
```

Aturan `NOPASSWD: ALL` memberikan hak administratif penuh kepada `lucas` melalui `sudo`; gunakan hanya sesuai kebutuhan lab.

## Hasil Grading

Perintah penilaian:

```bash
dcomctl grading quiz-24.2
```

Hasil pada percobaan ini:

| Pemeriksaan | Hasil |
|---|---|
| Password root berubah sesuai ketentuan | Gagal — grader melaporkan password salah |
| Group `administrators` | Benar |
| Primary group `administrators` sesuai konfigurasi soal | Benar |
| `lucas` dapat menjalankan perintah sebagai root tanpa password | Benar |
| Skor akhir | **75/100** |

Pesan utama dari grading:

```text
Checking root password has been changed: The password is incorrect.
administrators: configuration is correct.
Checking administrators group: The configuration is correct.
Checking Primary administrators group: The configuration is correct.
Checking if user 'lucas' can execute any command as root without password: The configuration is correct.
There is a configuration mismatch.
Your Score: 75
```

## Ringkasan dan Catatan

Tiga bagian konfigurasi group dan sudo telah dinyatakan benar oleh grader. Satu-satunya bagian yang belum lolos adalah password akun `root`.

Walaupun password `bbAA22dd` sudah dicoba, hasil grading tetap menandainya salah. Dokumentasi ini mencatat hasil aktual tersebut dan tidak menyatakan quiz berhasil penuh. Jika ingin memperbaiki skor, ulangi pengaturan password dengan `sudo passwd root`, pastikan pesan sukses muncul, lalu jalankan `dcomctl grading quiz-24.2` kembali.
