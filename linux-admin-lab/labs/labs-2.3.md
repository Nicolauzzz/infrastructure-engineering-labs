# Quiz 2.3 — Membandingkan Berkas Linux

## Tujuan
Memahami cara membandingkan isi berkas dan seluruh berkas dalam dua direktori menggunakan perintah `diff`.

## Soal 1 — Membandingkan Isi Berkas

Membandingkan `file_d.txt` dan `file_1.txt` tanpa membedakan huruf besar dan kecil.

```bash
diff -i /opt/quiz-2.3/resources/file_d.txt /opt/quiz-2.3/resources/file_1.txt > /opt/quiz-2.3/answers/result_file.txt
```

**Penjelasan:**
- `diff`: membandingkan isi dua berkas.
- `-i`: mengabaikan perbedaan huruf besar dan kecil.
- `>`: mengalihkan output ke berkas tujuan.

## Soal 2 — Membandingkan Direktori

Membandingkan seluruh berkas, termasuk subdirektori.

```bash
diff -r -q /opt/quiz-2.3/resources/file_dir /opt/quiz-2.3/resources/file_dir2 > /opt/quiz-2.3/answers/differences_file.txt
```

**Penjelasan:**
- `-r`: membandingkan direktori secara rekursif.
- `-q`: menampilkan informasi ringkas tentang perbedaan berkas.
- `>`: menyimpan output ke berkas tujuan.



## Bukti
Lihat tangkapan layar pada folder `../evidence/` jika bukti telah ditambahkan.