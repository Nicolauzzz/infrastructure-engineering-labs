# Quiz 24.1 — User and Group Management

## Tujuan
Membuat user lokal `david` dengan konfigurasi:
- Default shell: `/bin/sh`
- Primary group: `student`
- Wajib mengganti password saat login pertama
- Menjadi anggota supplementary group `mentor`

## Langkah Pengerjaan

### 1. Buat user `david`
```bash
sudo useradd -m -s /bin/sh -g student david
```

Keterangan:
- `-m`: membuat home directory `/home/david`
- `-s /bin/sh`: menetapkan shell login
- `-g student`: menetapkan primary group menjadi `student`

Jika user `david` sudah ada, periksa dengan `getent passwd david` atau `id david`, lalu sesuaikan konfigurasi menggunakan `usermod` alih-alih membuat user kembali.

### 2. Atur password
```bash
sudo passwd david
```

Masukkan password sesuai ketentuan lab.

### 3. Wajibkan perubahan password saat login pertama
```bash
sudo chage -d 0 david
```

Perintah ini mengatur tanggal perubahan password terakhir menjadi nol sehingga user diminta mengganti password saat login berikutnya.

### 4. Buat group `mentor`
```bash
sudo groupadd mentor
```

Jika group sudah ada, tidak perlu membuatnya kembali.

### 5. Tambahkan `david` ke group `mentor`
```bash
sudo usermod -aG mentor david
```

Gunakan `-aG` agar group tambahan yang sudah dimiliki user tidak tergantikan.

## Verifikasi

Jalankan perintah berikut untuk memastikan konfigurasi:
```bash
getent passwd david
id david
sudo chage -l david
getent group mentor
```

Periksa bahwa:
- User `david` menggunakan shell `/bin/sh`.
- Primary group `david` adalah `student`.
- `david` tercatat sebagai anggota group `mentor`.
- Password diatur agar harus diganti saat login berikutnya.

## Hasil Grading

Perintah grading:
```bash
dcomctl grading quiz-24.1
```

Hasil yang diperoleh:
```text
Running the assessment for quiz-24.1
Initiating the configuration check...
Checking if user 'david' exists: PASS
Checking if 'david' is a member of primary group 'student': PASS
Checking if 'david' must change password on first login: PASS
Checking if group 'mentor' exists: PASS
Checking if 'david' is a member of group 'mentor': PASS

Your Score: 100
Grading completed successfully!
```

**Skor: 100/100 — seluruh pemeriksaan berhasil.**

## Ringkasan Konsep

- **User account** merepresentasikan identitas pengguna pada sistem Linux.
- **Primary group** adalah group utama yang ditetapkan untuk user.
- **Supplementary group** memberikan keanggotaan group tambahan untuk akses dan izin.
- **Login shell** adalah shell yang dijalankan ketika user login.
- `chage -d 0 USER` memaksa perubahan password pada login berikutnya.
- `usermod -aG GROUP USER` menambahkan user ke group tambahan tanpa menghapus keanggotaan group tambahan lainnya.
