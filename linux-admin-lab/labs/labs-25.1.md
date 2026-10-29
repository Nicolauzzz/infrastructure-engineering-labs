# Quiz 25.1 — File Permissions and Ownership

## Tujuan
Menyelesaikan konfigurasi ownership, SGID, permission file, penghapusan file, dan ACL di Linux.

## Persiapan
```bash
dcomctl build-lab quiz-25.1
```

## Langkah Pengerjaan

### 1. Mengubah ownership secara rekursif
```bash
sudo chown -R david:student /opt/quiz-25.1/resources/change_me
```
Opsi `-R` menerapkan perubahan pada direktori `change_me` dan seluruh isi di dalamnya. Pemilik menjadi `david` dan group pemilik menjadi `student`.

### 2. Menambahkan bit SGID saja
```bash
sudo chmod g+s /opt/quiz-25.1/answers/permissions.txt
```
Perintah `g+s` menambahkan bit SGID tanpa menetapkan ulang permission dasar yang sudah ada.

Verifikasi:
```bash
ls -l /opt/quiz-25.1/answers/permissions.txt
```

### 3. Menghapus file `garbage.out`
```bash
sudo rm /opt/quiz-25.1/answers/garbage.out
```

### 4. Mengatur permission `perm_0001.txt`
```bash
sudo chmod 750 /opt/quiz-25.1/answers/perm_0001.txt
```
Mode `750` berarti:
- Owner (`7`): read, write, execute (`rwx`)
- Group (`5`): read dan execute (`r-x`)
- Others (`0`): tidak memiliki permission (`---`)

Permission yang diharapkan: `-rwxr-x---`.

### 5. Memberikan akses tulis melalui ACL kepada `john`
```bash
sudo setfacl -m u:john:w /opt/quiz-25.1/answers/acl_mnop.txt
```
Opsi `-m` memodifikasi ACL, sedangkan `u:john:w` memberikan akses tulis kepada user `john`.

Verifikasi:
```bash
getfacl /opt/quiz-25.1/answers/acl_mnop.txt
```
Pastikan entri untuk user `john` tercantum. Periksa juga ACL mask jika ingin memastikan permission efektif.

## Hasil Grading
Perintah penilaian:
```bash
dcomctl grading quiz-25.1
```

Hasil grading:
```text
Checking Ownership: Owned by user david and group student
Check SGID configuration: The configuration is correct.
File garbage.out has been removed
Check permissions file: The configuration is correct.
Checking John write-access to the file: The configuration is correct.

Your Score: 100
Grading completed successfully!
```

**Skor akhir: 100/100 — seluruh pemeriksaan berhasil.**

## Ringkasan Konsep
- `chown -R USER:GROUP PATH`: mengubah pemilik dan group secara rekursif.
- `chmod g+s FILE`: menambahkan bit SGID tanpa mengubah permission dasar lainnya.
- `chmod 750 FILE`: owner `rwx`, group `r-x`, others `---`.
- `setfacl`: mengatur izin khusus melalui Access Control List (ACL).
- `getfacl`: menampilkan ACL file atau direktori.

Setelah hasil grading dicatat, lab dapat dibersihkan jika diperlukan:
```bash
dcomctl reset-lab quiz-25.1
```
