# Quiz 21.1 — Configure Domain

## Tujuan
Mengaktifkan autostart untuk domain `webserver` dan memastikan VM tersebut berjalan.

## Soal

**Preparation:** Siapkan lingkungan kuis:

```bash
dcomctl build-lab quiz-21.1
```

**Task:**
1. Atur domain `webserver` agar otomatis menyala saat host boot.
2. Jalankan domain `webserver`.
3. Periksa hasil penilaian.

## Langkah pengerjaan

### 1. Periksa daftar VM

```bash
sudo virsh list --all
```

Pada hasil praktik, domain `cirros` dan `webserver` tersedia. Keduanya merupakan domain VM yang terpisah.

### 2. Aktifkan autostart

```bash
sudo virsh autostart webserver
```

Perintah ini membuat libvirt menyalakan `webserver` ketika host melakukan boot.

### 3. Jalankan VM dan verifikasi

```bash
sudo virsh start webserver
sudo virsh list --all
sudo virsh dominfo webserver
```

Pada `dominfo`, periksa `Autostart: enable`.

### 4. Jalankan grading

```bash
dcomctl grading quiz-21.1
```

Jika instruksi lab meminta reset, lakukan setelah selesai memeriksa hasil:

```bash
dcomctl reset-lab quiz-21.1
```

Reset dapat mengembalikan konfigurasi lab, jadi jangan menjalankannya sebelum selesai memeriksa hasil yang diperlukan.

## Hasil praktik

Berdasarkan keluaran terminal:
- `cirros` berstatus `running`.
- `webserver` berstatus `running`.
- Pemeriksaan autostart `webserver` dinyatakan benar.
- Pemeriksaan bahwa `webserver` berjalan dinyatakan benar.
- **Nilai kuis: 100/100.**

## Penjelasan konsep

- **Domain:** objek VM yang dikelola oleh libvirt.
- **Autostart:** membuat VM otomatis dimulai ketika host boot.
- **Start:** menyalakan VM sekarang; berbeda dari pengaturan autostart.
- **CirrOS vs. webserver:** keduanya domain berbeda. Nama `webserver` tidak otomatis berarti OS-nya CirrOS; periksa konfigurasi domain untuk memastikan OS dan disk image yang dipakai.

## Ringkasan perintah

| Kebutuhan | Perintah |
|---|---|
| Melihat semua VM | `sudo virsh list --all` |
| Mengaktifkan autostart | `sudo virsh autostart webserver` |
| Menjalankan VM | `sudo virsh start webserver` |
| Melihat detail domain | `sudo virsh dominfo webserver` |
| Menilai kuis | `dcomctl grading quiz-21.1` |
