# Quiz 11.1: Hard and Soft Links

## Tujuan

Mempraktikkan pembuatan symbolic link dan hard link di Linux, menyelesaikan symbolic link menjadi path absolut, serta memeriksa jumlah hard link pada filesystem.

## 1. Persiapan Lab

```bash
dcomctl build-lab quiz-11.1
```

## 2. Membuat Symbolic Link

Source file: `/opt/quiz-11.1/resources/source_object.dat`

Target symbolic link: `/opt/quiz-11.1/answers/softlink`

```bash
ln -s /opt/quiz-11.1/resources/source_object.dat /opt/quiz-11.1/answers/softlink
ls -l /opt/quiz-11.1/answers/softlink
```

## 3. Menyimpan Path Absolut Target Symbolic Link

```bash
readlink -f /opt/quiz-11.1/answers/softlink | sudo tee /opt/quiz-11.1/answers/dereference.txt
cat /opt/quiz-11.1/answers/dereference.txt
```

Hasil:

```text
/opt/quiz-11.1/resources/source_object.dat
```

## 4. Membuat Hard Link

Source: `/opt/quiz-11.1/resources/hard_target`

Hard link: `/opt/quiz-11.1/answers/hard_link`

```bash
ln /opt/quiz-11.1/resources/hard_target /opt/quiz-11.1/answers/hard_link
ls -li /opt/quiz-11.1/resources/hard_target /opt/quiz-11.1/answers/hard_link
```

Hasil menunjukkan kedua path memiliki inode yang sama (`2887652`) dan jumlah hard link `2`. Ini mengonfirmasi bahwa keduanya menunjuk ke inode yang sama.

## 5. Memeriksa Jumlah Hard Link pada Filesystem

Filesystem yang diperiksa berada di `/opt/quiz-11.1/resources/hardlink/`.

```bash
sudo find /opt/quiz-11.1/resources/hardlink -type f -printf '%n %p\n' | sort -nr | head
```

Hasil teratas menunjukkan jumlah link terbesar yang ditemukan adalah `2804`, pada file `/opt/quiz-11.1/resources/hardlink/file-m.app`.

Nilai tersebut dimasukkan ke file jawaban:

```bash
echo '2804' | sudo tee /opt/quiz-11.1/answers/hard_link_count.txt
cat /opt/quiz-11.1/answers/hard_link_count.txt
```

**Catatan:** Nilai jumlah hard link ini tidak diterima oleh pemeriksaan quiz. Kemungkinan nilai yang dicari grader memiliki kriteria yang lebih spesifik daripada mengambil angka terbesar dari daftar. Bagian ini belum berhasil diverifikasi lebih lanjut.

## 6. Hasil Grading

```bash
dcomctl grading quiz-11.1
```

Hasil:
- Symbolic link: benar.
- `dereference.txt`: benar.
- Hard link: benar.
- `hard_link_count.txt`: tidak sesuai dengan jawaban yang diharapkan.
- **Skor akhir: 75/100.**

Quiz dihentikan pada skor tersebut; nilai jumlah hard link belum diperbaiki.

## 7. Pelajaran yang Didapat

- `ln -s` membuat symbolic link yang menunjuk ke path target.
- `readlink -f` membantu mendapatkan path absolut target symbolic link.
- `ln` tanpa opsi `-s` membuat hard link.
- Hard link berbagi inode yang sama dengan file yang ditautkan.
- `%n` pada `find -printf '%n %p\n'` menampilkan jumlah hard link yang tercatat pada inode file.
- Hasil pencarian harus dicocokkan dengan kriteria soal; jumlah link terbesar belum tentu otomatis menjadi jawaban yang diterima grader.

## 8. Cleanup

Setelah hasil grading dicatat dan tidak perlu mengulang quiz, lab dapat di-reset:

```bash
dcomctl reset-lab quiz-11.1
```
