# Quiz 6.1 — Query the Package Management Database

## Tujuan

Mempelajari cara mencari paket yang memiliki file executable tertentu menggunakan `dpkg`, kemudian menyimpan nama paket ke file jawaban sesuai ketentuan quiz.

## Task

Cari nama paket yang memiliki executable `/usr/bin/ssh-keygen` dan tuliskan **nama paket saja** ke file berikut:

```text
/opt/quiz-6.1/package_output.txt
```

## Langkah Pengerjaan

### 1. Persiapan Lab

Jalankan perintah berikut untuk mempersiapkan lingkungan lab:

```bash
dcomctl build-lab quiz-6.1
```

### 2. Identifikasi Pemilik File

Gunakan `dpkg -S` untuk mencari paket yang memiliki executable `ssh-keygen`:

```bash
dpkg -S /usr/bin/ssh-keygen
```

Contoh output:

```text
openssh-client: /usr/bin/ssh-keygen
```

Berdasarkan hasil tersebut, nama paket yang dicari adalah `openssh-client`.

### 3. Simpan Nama Paket

Tuliskan nama paket ke file jawaban:

```bash
echo "openssh-client" | sudo tee /opt/quiz-6.1/package_output.txt
```

### 4. Verifikasi Jawaban

Periksa isi file untuk memastikan hanya nama paket yang tersimpan:

```bash
cat /opt/quiz-6.1/package_output.txt
```

Output:

```text
openssh-client
```

### 5. Penilaian Quiz

Jalankan perintah berikut untuk memeriksa hasil pengerjaan:

```bash
dcomctl grading quiz-6.1
```

Setelah penilaian selesai, bersihkan lingkungan lab:

```bash
dcomctl reset-lab quiz-6.1
```

## Konsep yang Dipelajari

- `dpkg -S`: mencari paket yang memiliki file tertentu.
- `tee`: menulis output ke file, dengan `sudo` untuk mengakses lokasi yang memerlukan hak administrator.
- `cat`: memeriksa isi file.

## Hasil

- **Paket:** `openssh-client`
- **File jawaban:** `/opt/quiz-6.1/package_output.txt`

