# Program dan Proses pada Linux

## 1. Apa Itu Program?

Program adalah sekumpulan instruksi yang dijalankan oleh komputer untuk melakukan tugas tertentu. Program dapat menggunakan data internal maupun data eksternal, seperti data dari database.

Program dapat berupa instruksi mesin yang dieksekusi CPU atau kumpulan perintah yang dijalankan melalui interpreter.

Contoh program Linux:
- `ls` — menampilkan isi direktori.
- `cat` — menampilkan isi berkas.
- `rm` — menghapus berkas atau direktori.
- `passwd` — mengubah kata sandi pengguna.

### Program dan Script

- **Program terkompilasi:** kode diterjemahkan menjadi bentuk yang dapat dieksekusi sebelum dijalankan.
- **Script:** kumpulan instruksi yang dijalankan oleh interpreter, seperti Bash, Python, atau Perl.

## 2. Apa Itu Proses?

Proses (*process*) adalah sebuah instance dari program yang sedang dijalankan.

Sebagai contoh, `ls` merupakan program. Ketika perintah `ls` dijalankan, sistem membuat proses untuk mengeksekusi program tersebut.

Setiap proses memiliki beberapa identitas:

| Atribut | Penjelasan |
|---|---|
| PID | Identitas unik proses (*Process ID*) |
| PPID | Identitas proses induk (*Parent Process ID*) |
| PGID | Identitas kelompok proses (*Process Group ID*) |

Proses dapat berada dalam beberapa keadaan, misalnya berjalan atau menunggu. Kernel Linux mengatur penjadwalan agar proses dapat berbagi waktu penggunaan CPU.

### Program vs. Proses

| Program | Proses |
|---|---|
| Berisi instruksi untuk melakukan tugas | Instans program yang sedang dijalankan |
| Dapat tersimpan sebagai berkas di disk | Memiliki status dan sumber daya saat berjalan |
| Satu program dapat dijalankan berkali-kali | Setiap eksekusi dapat menghasilkan proses berbeda |

## 3. Atribut Proses

Setiap proses memiliki atribut utama berikut:

- **Program:** instruksi yang sedang dijalankan.
- **Context:** keadaan proses, termasuk informasi register CPU dan keadaan eksekusi yang diperlukan untuk melanjutkan proses.
- **Permissions:** hak akses yang menentukan tindakan yang dapat dilakukan proses.
- **Resources:** sumber daya yang digunakan, seperti memori dan file descriptor.

### Context Switching

Context switching adalah proses ketika kernel menyimpan keadaan suatu proses dan memulihkan keadaan proses lain agar CPU dapat menjalankan proses yang berbeda.

Mekanisme ini memungkinkan banyak proses berbagi waktu CPU.

### Hak Akses Proses

Hak akses proses berkaitan dengan identitas pengguna dan izin yang berlaku ketika program dijalankan.

Program tertentu menggunakan mekanisme **setuid**, yaitu mekanisme yang memungkinkan proses berjalan dengan effective user ID dari pemilik program, sesuai pengaturan dan ketentuan sistem.

Contohnya adalah program `passwd`, yang memerlukan hak istimewa untuk memperbarui informasi kata sandi yang dilindungi. Mekanisme setuid harus dikonfigurasi dengan hati-hati karena dapat menimbulkan risiko keamanan.

## 4. Perintah Praktik Linux

### Melihat Proses yang Sedang Berjalan

```bash
ps
```

Menampilkan proses yang terkait dengan terminal saat ini.

```bash
ps aux
```

Menampilkan daftar proses yang berjalan dengan informasi pengguna dan penggunaan sumber daya.

```bash
top
```

Menampilkan proses dan penggunaan sumber daya sistem secara dinamis.

### Melihat PID dan PPID

```bash
ps -eo pid,ppid,pgid,stat,comm
```

Menampilkan PID, PPID, PGID, status, dan nama perintah untuk proses yang terdaftar.

### Mencari Proses Tertentu

```bash
pgrep -a ssh
```

Mencari proses berdasarkan pola nama `ssh` dan menampilkan informasi proses yang cocok.

## 5. Kesimpulan

- Program adalah kumpulan instruksi, sedangkan proses adalah instans program yang sedang dieksekusi.
- Setiap proses memiliki identitas, status, hak akses, dan sumber daya.
- Kernel mengatur proses melalui penjadwalan dan context switching.
- Perintah `ps`, `top`, dan `pgrep` membantu administrator memeriksa proses Linux.

Pemahaman proses Linux penting untuk memantau layanan server, memeriksa aplikasi yang berjalan, menganalisis penggunaan CPU dan memori, serta melakukan troubleshooting pada server dan container.