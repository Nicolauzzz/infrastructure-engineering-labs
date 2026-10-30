# Quiz 26.1 — Working with SSL Certificate

## Tujuan
Mengidentifikasi Subject Common Name (CN) pada sertifikat SSL di `/ssl`, kemudian memindahkan sertifikat ke subdirektori yang namanya sesuai dengan CN.

## Persiapan
```bash
dcomctl build-lab quiz-26.1
ls -l /ssl
find /ssl -maxdepth 2 -type d
```

File sertifikat yang diperiksa adalah `/ssl/lab-ssl.pem`.

## Langkah Pengerjaan

### 1. Membaca Subject CN
```bash
openssl x509 -in /ssl/lab-ssl.pem -noout -subject
```

Output:
```text
subject=C = ID, ST = Jakarta, L = Jakarta Selatan, O = PT. Datacomm Diangraha, OU = Cloud Development, CN = lab.playground.com
```

Subject CN sertifikat adalah `lab.playground.com`.

### 2. Memindahkan sertifikat
```bash
sudo mv /ssl/lab-ssl.pem /ssl/lab.playground.com/
```

Percobaan awal gagal karena nama file sumber salah ditulis sebagai `lab.playground.pem`. Nama file yang benar adalah `lab-ssl.pem`.

### 3. Verifikasi
```bash
ls -l /ssl/lab.playground.com/
```

Pastikan file `lab-ssl.pem` berada di direktori yang sesuai dengan Subject CN.

## Hasil Grading

```bash
dcomctl grading quiz-26.1
```

Output:
```text
Running the assessment for quiz-26.1
Initiating the configuration check...
Checking SSL: The configuration is correct.

Your Score: 100
Grading completed successfully!
```

**Skor akhir: 100/100 — konfigurasi dinyatakan benar.**

## Ringkasan Konsep
- Sertifikat X.509 menyimpan identitas di bagian Subject.
- CN (*Common Name*) adalah salah satu atribut Subject yang menyatakan nama umum sertifikat.
- `openssl x509 -in FILE -noout -subject` menampilkan Subject sertifikat.
- `mv` memindahkan file; pastikan nama sumber dan direktori tujuan benar.

Setelah hasil grading dicatat, lab dapat dibersihkan jika diperlukan:
```bash
dcomctl reset-lab quiz-26.1
```
