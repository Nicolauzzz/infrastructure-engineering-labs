# Quiz 20.1 — Configure Kernel Parameter

## 1. Tujuan Quiz

Quiz ini bertujuan mengaktifkan fitur **IPv4 forwarding** pada kernel Linux secara persisten. Fitur ini memungkinkan kernel meneruskan paket IPv4 antarjaringan, sehingga sering digunakan ketika Linux dikonfigurasi sebagai router.

**Hasil akhir pengerjaan: 100/100.**

## 2. Konsep Penting

- **Kernel parameter:** pengaturan yang mengubah perilaku kernel Linux.
- **`sysctl`:** utilitas untuk membaca dan mengubah parameter kernel saat sistem berjalan.
- **`net.ipv4.ip_forward`:** parameter yang mengatur apakah kernel meneruskan paket IPv4.
  - `0`: IPv4 forwarding tidak aktif.
  - `1`: IPv4 forwarding aktif.
- **Konfigurasi persisten:** pengaturan yang tetap diterapkan setelah sistem reboot, misalnya melalui `/etc/sysctl.conf`.

Mengubah nilai saat runtime saja tidak menjamin pengaturan bertahan setelah reboot. Karena itu, quiz meminta konfigurasi yang persisten.

## 3. Langkah Pengerjaan

### Langkah 1 — Menyiapkan lingkungan quiz

```bash
dcomctl build-lab quiz-20.1
```

Perintah ini menjalankan skrip persiapan dan menyiapkan lingkungan lab.

### Langkah 2 — Memeriksa nilai IPv4 forwarding

```bash
sysctl net.ipv4.ip_forward
```

Perintah ini menampilkan nilai parameter saat ini. Nilai `0` berarti forwarding tidak aktif, sedangkan `1` berarti aktif.

### Langkah 3 — Mengaktifkan IPv4 forwarding saat runtime

```bash
sudo sysctl net.ipv4.ip_forward=1
```

Perintah tersebut langsung mengubah nilai parameter kernel tanpa perlu reboot. Namun, perubahan runtime ini saja belum memenuhi kebutuhan konfigurasi persisten.

### Langkah 4 — Membuat konfigurasi persisten

Buka file konfigurasi:

```bash
sudo nano /etc/sysctl.conf
```

Tambahkan baris berikut:

```text
net.ipv4.ip_forward=1
```

Simpan dengan `Ctrl+O`, tekan `Enter`, lalu keluar dengan `Ctrl+X`.

Terapkan konfigurasi dari file:

```bash
sudo sysctl -p
```

Perintah `sysctl -p` membaca dan menerapkan pengaturan dari `/etc/sysctl.conf` tanpa perlu melakukan reboot.

### Langkah 5 — Memverifikasi konfigurasi

```bash
sysctl net.ipv4.ip_forward
```

Hasil yang diharapkan:

```text
net.ipv4.ip_forward = 1
```

Pada pengerjaan ini, parameter menunjukkan nilai `1`, sehingga IPv4 forwarding aktif.

### Langkah 6 — Menjalankan penilaian

```bash
dcomctl grading quiz-20.1
```

Hasil penilaian yang diperoleh:

```text
Checking IPv4 forwarding: The configuration is correct.

Your Score: 100
Grading completed successfully!
```

**Hasil: seluruh pemeriksaan berhasil dengan nilai 100/100.**

## 4. Ringkasan Perintah

```bash
# Menyiapkan lab
dcomctl build-lab quiz-20.1

# Memeriksa nilai saat ini
sysctl net.ipv4.ip_forward

# Mengaktifkan IPv4 forwarding saat runtime
sudo sysctl net.ipv4.ip_forward=1

# Mengedit konfigurasi persisten
sudo nano /etc/sysctl.conf

# Setelah menambahkan net.ipv4.ip_forward=1, terapkan konfigurasi
sudo sysctl -p

# Verifikasi
sysctl net.ipv4.ip_forward

# Penilaian
dcomctl grading quiz-20.1
```

## 5. Catatan Keamanan dan Operasional

Mengaktifkan IPv4 forwarding memungkinkan sistem meneruskan paket IPv4, tetapi tidak otomatis membuat routing berfungsi dengan benar. Rute, antarmuka jaringan, firewall, dan aturan keamanan tetap perlu dikonfigurasi sesuai kebutuhan. Aktifkan fitur ini hanya jika memang diperlukan oleh fungsi sistem.

Jika ingin mengulang lab, gunakan:

```bash
dcomctl reset-lab quiz-20.1
```

Jalankan reset hanya jika memang ingin membersihkan atau mengulang lingkungan latihan.
