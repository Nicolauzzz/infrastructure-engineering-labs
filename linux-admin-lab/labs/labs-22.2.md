# Quiz 22.2 — Dockerfile

## Tujuan
Membangun image dari Dockerfile yang disediakan dan menjalankan container dengan nama `student-app`, port `8080:80`, batas memori `256m`, bind mount folder sharing, dan restart policy `always`.

## Langkah Pengerjaan

### 1. Build lab
```bash
dcomctl build-lab quiz-22.2
```

Tunggu sampai build selesai.

### 2. Build image `myapp:latest`
```bash
sudo docker build -t myapp:latest /opt/quiz-22.2/container-image/
```

- `-t myapp:latest`: memberi nama dan tag image.
- `/opt/quiz-22.2/container-image/`: direktori build context yang berisi Dockerfile.

Verifikasi:
```bash
docker images myapp
```

### 3. Buat dan jalankan container
```bash
sudo docker run -d \
  --name student-app \
  -p 8080:80 \
  --memory 256m \
  -v /opt/quiz-22.2/sharing:/usr/share/nginx/html \
  --restart always \
  myapp:latest
```

| Opsi | Fungsi |
|---|---|
| `-d` | Menjalankan container di background |
| `--name student-app` | Nama container |
| `-p 8080:80` | Port host 8080 menuju port container 80 |
| `--memory 256m` | Batas memori 256 MiB |
| `-v HOST:CONTAINER` | Bind mount folder host ke direktori container |
| `--restart always` | Menjalankan kembali container ketika berhenti atau Docker daemon restart |
| `myapp:latest` | Image yang digunakan |

### 4. Verifikasi konfigurasi

Status dan port:
```bash
docker ps -a
```

Memory limit:
```bash
docker inspect -f '{{.HostConfig.Memory}}' student-app
```
Nilai yang diharapkan: `268435456` byte (256 MiB).

Restart policy:
```bash
docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' student-app
```
Hasil yang diharapkan: `always`.

Port mapping:
```bash
docker inspect -f '{{json .HostConfig.PortBindings}}' student-app
```
Pastikan host port `8080` diarahkan ke port container `80/tcp`.

Bind mount:
```bash
docker inspect -f '{{range .Mounts}}{{.Source}} -> {{.Destination}}{{println}}{{end}}' student-app
```
Hasil yang diharapkan:
```text
/opt/quiz-22.2/sharing -> /usr/share/nginx/html
```

## Kendala Permission Docker Socket

Pada awalnya, grading gagal dengan pesan `permission denied` saat mengakses `/var/run/docker.sock`. `sudo docker ps -a` berhasil, tetapi `docker ps -a` tanpa `sudo` gagal. Socket dimiliki `root:docker` dan user `student` belum memiliki akses grup Docker.

Perbaikan yang dilakukan:
```bash
sudo usermod -aG docker student
newgrp docker
```

Setelah itu, Docker bisa diakses tanpa `sudo` dan grading diulang.

**Catatan keamanan:** keanggotaan grup `docker` memberikan hak istimewa tinggi, pada praktiknya setara akses root. Gunakan hanya di mesin/lab yang dipercaya.

## Hasil Akhir

**Grading: 100/100 — berhasil.**

Semua pemeriksaan lulus:
- Image `myapp:latest` berhasil dibangun.
- Container `student-app` berjalan.
- Port mapping benar.
- Memory limit benar.
- Folder sharing ter-mount.
- Restart policy benar.

## Ringkasan
Quiz ini menguji pembuatan image Docker dan konfigurasi container. Pastikan nama image dan container tepat, port host `8080` dipetakan ke port container `80`, batas memori `256m`, mount folder sharing, serta restart policy `always`. Pastikan juga user dapat mengakses Docker API agar grader dapat memeriksa konfigurasi.
