# Quiz 22.1 — Working With Container

## Tujuan
Mengelola container Docker: menghapus container yang ditentukan, menjalankan container Nginx dengan port mapping yang sesuai, dan memastikan container memiliki restart policy `always`.

## Instruksi Quiz
1. Hapus container `testcontainer`.
2. Buat container baru bernama `persistcontainer` menggunakan image `nginx`.
3. Petakan port 80 di dalam container ke port 8088 pada host.
4. Pastikan container selalu dijalankan kembali jika berhenti atau Docker daemon direstart.

## Konsep Penting

### Container dan image
- **Image** adalah template yang digunakan untuk membuat container.
- **Container** adalah instance image yang dijalankan.
- `nginx` adalah image web server Nginx.

### Port mapping
Opsi `-p 8088:80` berarti port **8088 pada host** diteruskan ke port **80 di dalam container**.

### Restart policy
Opsi `--restart always` membuat Docker berusaha menjalankan kembali container ketika container berhenti, dan menjalankannya kembali ketika Docker daemon dimulai ulang. Ini berbeda dari `unless-stopped`, yang menghormati penghentian manual yang disengaja.

## Perintah Utama

### 1. Periksa container yang ada
```bash
sudo docker ps -a
```

- `docker ps` menampilkan container yang sedang berjalan.
- `docker ps -a` menampilkan semua container, termasuk yang berhenti atau gagal dibuat.

### 2. Hapus container `testcontainer`
```bash
sudo docker rm -f testcontainer
```

Opsi `-f` memaksa penghentian dan penghapusan container jika container tersebut ada. Jika muncul pesan `No such container`, container itu memang tidak ditemukan; lanjutkan setelah memeriksa daftar container.

### 3. Ambil image Nginx
```bash
sudo docker pull nginx:latest
```

### 4. Buat container sesuai persyaratan
```bash
sudo docker run -d \
  --name persistcontainer \
  -p 8088:80 \
  --restart always \
  nginx:latest
```

Penjelasan:
- `run`: membuat dan menjalankan container baru.
- `-d`: menjalankan di latar belakang.
- `--name persistcontainer`: menetapkan nama container.
- `-p 8088:80`: memetakan port host 8088 ke port container 80.
- `--restart always`: menetapkan restart policy.
- `nginx:latest`: image yang digunakan.

### 5. Verifikasi
```bash
sudo docker ps -a
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' persistcontainer
curl -I http://localhost:8088
```

Hasil yang diharapkan:
- Container bernama `persistcontainer` berstatus `Up`.
- Port ditampilkan sebagai `0.0.0.0:8088->80/tcp` (mungkin juga ada pemetaan IPv6).
- Restart policy menghasilkan `always`.
- `curl` menerima respons HTTP dari Nginx.

## Troubleshooting: Grading hanya 50/100

Dalam percobaan lab ini, restart policy `always` sudah terkonfirmasi, tetapi container tetap berstatus `Created` karena Docker melaporkan:

```text
failed to bind host port 0.0.0.0:8088/tcp: address already in use
```

Pemeriksaan `ss` menunjukkan `docker-proxy` sedang mendengarkan port 8088, sedangkan `docker ps --filter publish=8088` tidak menemukan container yang memublikasikan port itu. Dengan demikian, port kemungkinan tertahan oleh proses proxy yang tertinggal atau ada ketidaksesuaian state jaringan Docker. Restart Docker telah dicoba, tetapi grading tetap 50/100. Akar masalah belum dapat dipastikan dari bukti yang tersedia.

### Pemeriksaan yang disarankan jika melanjutkan lab

Jalankan perintah berikut dan simpan hasilnya:

```bash
sudo docker ps -a
sudo docker inspect persistcontainer
sudo ss -ltnp | grep ':8088'
sudo systemctl status docker --no-pager
sudo journalctl -u docker -n 100 --no-pager
```

Periksa pula apakah ada container lain yang memakai port host 8088, termasuk container yang dibuat melalui runtime berbeda. Jangan menghapus file socket atau file lock secara manual.

Jika `persistcontainer` berstatus `Created` dan tidak berjalan, hapus container gagal tersebut sebelum mencoba lagi:

```bash
sudo docker rm -f persistcontainer
```

Kemudian, hanya setelah port 8088 benar-benar bebas, buat ulang container dengan perintah pada bagian 4. Jika `docker-proxy` masih mendengarkan port tanpa container yang cocok, jangan membunuh proses atau menghapus resource secara membabi buta; periksa log daemon dan lingkungan lab terlebih dahulu.

## Catatan Hasil Lab
- Image `nginx:latest` berhasil diunduh.
- Container `persistcontainer` berhasil dibuat, tetapi pada percobaan terakhir tidak berhasil mulai karena konflik port 8088.
- Restart policy `always` telah diverifikasi.
- Hasil grading terakhir: **50/100**.
- Status: **belum selesai sepenuhnya**; penyebab konflik port masih perlu ditelusuri.

## Ringkasan
Kunci quiz ini adalah tiga hal: menghapus `testcontainer`, menjalankan `persistcontainer` dengan `-p 8088:80`, dan menetapkan `--restart always`. Pastikan container benar-benar berstatus `Up`, bukan hanya `Created`, sebelum menjalankan grading.
