# Quiz 27.1 — System Routing Table

## Tujuan

Menyimpan salinan system routing table ke file `/opt/quiz-27.1/table.txt`.

**Hasil quiz: 100/100.**

## 1. Persiapan

```bash
dcomctl build-lab quiz-27.1
```

## 2. Melihat routing table

Perintah:

```bash
route -n
```

Opsi `-n` menampilkan alamat IP dan gateway dalam format numerik, tanpa menerjemahkannya menjadi hostname.

Output pada lab:

```text
Kernel IP routing table
Destination     Gateway       Genmask         Flags Metric Ref Use Iface
0.0.0.0         192.168.0.1   0.0.0.0         UG    0      0   0   enp0s3
10.10.10.0      0.0.0.0       255.255.255.0   U     0      0   0   enp0s8
172.17.0.0      0.0.0.0       255.255.0.0     U     0      0   0   docker0
192.168.0.0     0.0.0.0       255.255.255.0   U     0      0   0   enp0s3
192.168.122.0   0.0.0.0       255.255.255.0   U     0      0   0   virbr0
```

## 3. Menyimpan routing table

Perintah yang digunakan:

```bash
route -n | sudo tee /opt/quiz-27.1/table.txt
```

Penjelasan:
- `route -n`: menampilkan routing table dalam format numerik.
- `|`: meneruskan output ke perintah berikutnya.
- `sudo tee`: menulis output ke file dengan hak akses root sekaligus menampilkannya di terminal.
- `/opt/quiz-27.1/table.txt`: lokasi file tujuan.

Verifikasi file:

```bash
cat /opt/quiz-27.1/table.txt
```

## 4. Memahami kolom pada `route -n`

| Kolom | Penjelasan |
|---|---|
| **Destination** | Jaringan atau host tujuan yang dicocokkan dengan alamat IP tujuan paket. `0.0.0.0` berarti default route. |
| **Gateway** | Router/next hop untuk meneruskan paket. `0.0.0.0` berarti jaringan tujuan terhubung langsung (*directly connected*). |
| **Genmask** | Subnet mask yang menentukan cakupan alamat tujuan pada rute tersebut. |
| **Flags** | Penanda karakteristik rute. `U` berarti rute aktif (*up*), `G` berarti menggunakan gateway, dan `H` berarti rute menuju satu host. |
| **Metric** | Nilai biaya/prioritas rute. Jika ada beberapa rute yang sama-sama cocok, metric lebih rendah umumnya lebih diutamakan. |
| **Ref** | Kolom referensi rute yang bersifat historis dan biasanya bernilai `0` pada Linux modern. |
| **Use** | Penghitung penggunaan rute pada format tabel klasik; pada sistem modern sering bernilai `0` atau tidak terlalu informatif. |
| **Iface** | Interface jaringan yang digunakan untuk mengirim paket melalui rute tersebut. |

### Arti flags yang umum

- `U`: route aktif.
- `G`: route menggunakan gateway.
- `H`: route menuju satu host tertentu.

Contoh `UG` berarti rute aktif dan paket diteruskan melalui gateway.

## 5. Membaca routing table hasil lab

### A. Default route

```text
0.0.0.0  192.168.0.1  0.0.0.0  UG  ...  enp0s3
```

Jika tujuan paket tidak cocok dengan rute yang lebih spesifik, sistem mengirim paket ke gateway `192.168.0.1` melalui interface `enp0s3`.

### B. Jaringan 10.10.10.0/24

```text
10.10.10.0  0.0.0.0  255.255.255.0  U  ...  enp0s8
```

Jaringan `10.10.10.0/24` terhubung langsung melalui `enp0s8`. Gateway `0.0.0.0` menandakan paket dikirim langsung ke jaringan tersebut, bukan melalui router lain.

### C. Jaringan Docker

```text
172.17.0.0  0.0.0.0  255.255.0.0  U  ...  docker0
```

Jaringan `172.17.0.0/16` terhubung melalui interface virtual `docker0`, yang umumnya dibuat Docker untuk jaringan bridge default.

### D. Jaringan lokal utama

```text
192.168.0.0  0.0.0.0  255.255.255.0  U  ...  enp0s3
```

Jaringan `192.168.0.0/24` terhubung langsung melalui `enp0s3`. Gateway `192.168.0.1` pada default route berada dalam jaringan ini.

### E. Jaringan virtualisasi

```text
192.168.122.0  0.0.0.0  255.255.255.0  U  ...  virbr0
```

Jaringan `192.168.122.0/24` terhubung melalui `virbr0`, yaitu interface bridge virtual yang umumnya dibuat libvirt/KVM untuk jaringan VM.

## 6. Hasil grading

Jalankan:

```bash
dcomctl grading quiz-27.1
```

Hasil penilaian:

```text
Checking /opt/quiz-27.1/table.txt: The configuration is correct.
Your Score: 100
Grading completed successfully!
```

**Skor: 100/100.**

## 7. Kesimpulan

Routing table digunakan kernel Linux untuk menentukan ke mana paket jaringan harus diteruskan. Kernel mencocokkan alamat tujuan dengan rute yang tersedia dan memilih rute yang paling spesifik. Jika tidak ada rute yang lebih spesifik, default route dapat digunakan.

Quiz ini melatih cara melihat routing table menggunakan `route -n`, memahami setiap kolom, dan menyimpan hasilnya ke file untuk kebutuhan pemeriksaan maupun dokumentasi.
