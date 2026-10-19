# Quiz 21.2 — Create Virtual Machine

## Tujuan
Membuat VM bernama `jammy` menggunakan `virt-install` dan image Ubuntu Jammy yang disediakan Datacomm. RAM yang diminta adalah **1024 MB**, dan pembuatan VM harus menggunakan `--noautoconsole`.

## Soal
Jalankan persiapan lab:

```bash
dcomctl build-lab quiz-21.2
```

Buat VM dengan spesifikasi:
- Nama domain: `jammy`
- RAM: `1024 MB`
- Image: `jammy-server-cloudimg-amd64.img`
- Utilitas: `virt-install`
- Opsi wajib: `--noautoconsole`

Setelah selesai, jalankan:

```bash
dcomctl grading quiz-21.2
```

## Konsep penting
- **QEMU/KVM** menjalankan virtual machine di Linux.
- **libvirt** menyediakan pengelolaan VM/domain.
- **`virt-install`** membuat dan mengonfigurasi VM.
- **`--import`** menggunakan image OS yang sudah tersedia.
- **`--memory 1024`** menetapkan RAM sebesar 1024 MB.
- **`--noautoconsole`** mencegah konsol VM terbuka otomatis setelah proses pembuatan.

---

## Metode A — Mengunduh image langsung menggunakan `wget`

Metode ini paling langsung, tetapi kecepatannya bergantung pada koneksi internet server lab. Pada pengerjaan ini, unduhan melalui `wget` sangat lambat sehingga digunakan Metode B.

### 1. Unduh image

```bash
wget https://s3-jkt1.dcloud.co.id/lms-datacomm-asset-qre62/quiz-21.1/jammy-server-cloudimg-amd64.img
```

Periksa file:

```bash
ls -lh jammy-server-cloudimg-amd64.img
```

### 2. Buat VM

Jika file berada di direktori kerja saat ini:

```bash
sudo virt-install \
  --name jammy \
  --memory 1024 \
  --vcpus 1 \
  --disk path="$PWD/jammy-server-cloudimg-amd64.img",format=qcow2 \
  --import \
  --osinfo detect=on,require=off \
  --network network=default \
  --noautoconsole
```

Jika image dipindahkan ke direktori lain, sesuaikan nilai `path=` dengan lokasi file yang benar.

---

## Metode B — Unduh di Windows lalu gunakan VirtualBox Shared Folder

Metode ini digunakan karena unduhan `wget` di Linux terlalu lambat. Image diunduh melalui browser Windows, lalu dibagikan ke Linux melalui fitur Shared Folders VirtualBox.

### 1. Unduh image melalui browser Windows

Buka tautan berikut:

<https://s3-jkt1.dcloud.co.id/lms-datacomm-asset-qre62/quiz-21.1/jammy-server-cloudimg-amd64.img>

Simpan file dengan nama `jammy-server-cloudimg-amd64.img`. Pada pengerjaan ini, file disimpan di:

`C:\labs-21.2\jammy-server-cloudimg-amd64.img`

Ukuran file sekitar 621 MB.

### 2. Konfigurasikan Shared Folder di VirtualBox

1. Pilih VM Linux yang menjalankan `lab-playground`.
2. Buka **Settings → Shared Folders**.
3. Tambahkan folder Windows `C:\labs-21.2`.
4. Beri nama share `labs-21.2`.
5. Aktifkan **Auto-mount** dan **Make Permanent**.

### 3. Aktifkan modul dan mount folder di Linux

Periksa modul:

```bash
lsmod | grep vboxsf
```

Jika belum muncul, coba:

```bash
sudo modprobe vboxsf
```

Buat lokasi mount dan mount share:

```bash
sudo mkdir -p /mnt/shared
sudo mount -t vboxsf labs-21.2 /mnt/shared
```

Penjelasan:
- `vboxsf` adalah filesystem Shared Folders VirtualBox.
- `labs-21.2` adalah nama share yang dikonfigurasi di VirtualBox.
- `/mnt/shared` adalah lokasi folder tersebut di Linux.

### 4. Verifikasi file

```bash
ls -lh /mnt/shared
ls -lh /mnt/shared/jammy-server-cloudimg-amd64.img
```

Pada pengerjaan ini, file berhasil terlihat di `/mnt/shared` dengan ukuran sekitar 621 MB. Jadi, image tidak perlu diunduh ulang melalui `wget` di Linux.

### 5. Buat VM menggunakan image lokal

```bash
sudo virt-install \
  --name jammy \
  --memory 1024 \
  --vcpus 1 \
  --disk path=/mnt/shared/jammy-server-cloudimg-amd64.img,format=qcow2 \
  --import \
  --osinfo detect=on,require=off \
  --network network=default \
  --noautoconsole
```

**Catatan:** VM menggunakan image dari Shared Folder. Untuk penggunaan jangka panjang, lebih baik salin image ke penyimpanan Linux/libvirt lalu gunakan salinannya. Jangan memindahkan atau menghapus file sumber ketika VM masih bergantung padanya.

---

## Penjelasan opsi `virt-install`

| Opsi | Fungsi |
|---|---|
| `sudo virt-install` | Membuat VM melalui libvirt dengan hak administrator. |
| `--name jammy` | Memberi nama domain `jammy`. |
| `--memory 1024` | Mengalokasikan RAM 1024 MB. |
| `--vcpus 1` | Mengalokasikan satu virtual CPU. |
| `--disk path=...,format=qcow2` | Menentukan file disk dan formatnya. |
| `--import` | Menggunakan image OS yang sudah tersedia. |
| `--osinfo detect=on,require=off` | Mengizinkan deteksi OS otomatis tanpa menjadikannya syarat wajib. |
| `--network network=default` | Menghubungkan VM ke jaringan libvirt `default`. |
| `--noautoconsole` | Tidak otomatis membuka konsol VM setelah pembuatan. |

Jika jaringan `default` tidak tersedia atau konfigurasi lab mensyaratkan jaringan lain, sesuaikan opsi jaringan dengan lingkungan lab.

## Verifikasi hasil

### 1. Periksa daftar VM

```bash
sudo virsh list --all
```

Pada pengerjaan ini, daftar VM menunjukkan `jammy` dalam status `running`. ID dan status dapat berbeda pada lingkungan lain.

### 2. Jalankan penilaian

```bash
dcomctl grading quiz-21.2
```

Hasil pengerjaan:

```text
Checking configuration VM jammy: The configuration is correct.
Your Score: 100
Grading completed successfully!
```

**Nilai: 100/100.**

## Alur pengerjaan singkat

1. Login ke sesi Linux Playground.
2. Jalankan `dcomctl build-lab quiz-21.2`.
3. Dapatkan image Jammy melalui `wget` atau unduh melalui browser Windows.
4. Jika memakai Windows, konfigurasi Shared Folder VirtualBox dan mount ke `/mnt/shared`.
5. Jalankan `virt-install` dengan nama `jammy`, RAM 1024 MB, `--import`, dan `--noautoconsole`.
6. Periksa dengan `sudo virsh list --all`.
7. Jalankan `dcomctl grading quiz-21.2` dan pastikan hasilnya benar.

## Catatan operasional

- Gunakan image dari sumber course yang dipercaya.
- Periksa `sudo virsh list --all` sebelum mengulangi `virt-install`; nama domain `jammy` mungkin sudah digunakan.
- Shared Folder praktis untuk transfer file. Untuk VM yang akan terus digunakan, simpan disk image di penyimpanan Linux yang sesuai.
- Perintah `dcomctl reset-lab quiz-21.2` dapat membersihkan resource lab. Jalankan setelah penilaian hanya jika memang ingin mereset lab.
