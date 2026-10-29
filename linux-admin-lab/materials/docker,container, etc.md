# Docker, Container, dan Manajemen Akun Linux
**Tanggal belajar:** 23 Oktober 2026  
**Lingkungan praktik:** Datacomm Linux Playground / Ubuntu 22.04

Dokumen ini menggabungkan materi Docker dan container dari latihan sebelumnya dengan materi Linux tanggal 23 tentang manajemen akun pengguna, SSH, dan pengelolaan grup.

---

## Daftar Isi

1. [Docker dan Container](#1-docker-dan-container)
2. [Quiz 22.1 — Working With Container](#2-quiz-221--working-with-container)
3. [Quiz 22.2 — Dockerfile](#3-quiz-222--dockerfile)
4. [Troubleshooting Docker di Linux Playground](#4-troubleshooting-docker-di-linux-playground)
5. [Manajemen Akun Pengguna Linux](#5-manajemen-akun-pengguna-linux)
6. [Password dan Masa Berlaku Akun](#6-password-dan-masa-berlaku-akun)
7. [Restricted Shell dan Restricted Account](#7-restricted-shell-dan-restricted-account)
8. [Root dan Hak Administratif](#8-root-dan-hak-administratif)
9. [SSH dan Transfer File](#9-ssh-dan-transfer-file)
10. [Pengelolaan Grup Linux](#10-pengelolaan-grup-linux)
11. [Ringkasan Perintah](#11-ringkasan-perintah)

---

# 1. Docker dan Container

## 1.1 Konsep dasar

- **Image** adalah template read-only yang digunakan untuk membuat container.
- **Container** adalah instance image yang dijalankan sebagai proses terisolasi.
- **Docker daemon** (`dockerd`) mengelola image, container, jaringan, volume, dan operasi Docker.
- **Dockerfile** berisi instruksi untuk membangun image.
- **Port mapping** meneruskan koneksi dari port host ke port container.
- **Bind mount** memasang direktori host ke direktori di dalam container.
- **Restart policy** mengatur apakah Docker akan menjalankan kembali container ketika container berhenti atau Docker daemon dimulai ulang.

Contoh sederhana:

```bash
sudo docker run -d --name web-nginx -p 8080:80 nginx:latest
```

Artinya:
- `run`: membuat dan menjalankan container.
- `-d`: berjalan di background.
- `--name web-nginx`: nama container.
- `-p 8080:80`: port 8080 pada host diteruskan ke port 80 di container.
- `nginx:latest`: image yang digunakan.

## 1.2 Perintah pemeriksaan

```bash
docker --version
sudo systemctl status docker --no-pager
sudo docker images
sudo docker ps
sudo docker ps -a
sudo docker inspect CONTAINER
sudo docker logs CONTAINER
```

- `docker ps` menampilkan container yang sedang berjalan.
- `docker ps -a` menampilkan container dalam semua status, termasuk `Created`, `Exited`, dan `Up`.
- `docker images` menampilkan image lokal.
- `docker inspect` menampilkan konfigurasi detail container.
- `docker logs` menampilkan output aplikasi container.

## 1.3 Menghapus container

```bash
sudo docker stop NAMA_CONTAINER
sudo docker rm NAMA_CONTAINER
```

Untuk menghentikan dan menghapus container sekaligus:

```bash
sudo docker rm -f NAMA_CONTAINER
```

`-f` memaksa penghentian jika container masih berjalan. Pastikan nama container benar sebelum menghapusnya.

## 1.4 Port mapping

Format umum:

```bash
-p PORT_HOST:PORT_CONTAINER
```

Contoh:

```bash
-p 8088:80
```

Artinya pengguna mengakses port `8088` pada host, sedangkan aplikasi di dalam container mendengarkan port `80`.

Periksa port yang sedang digunakan:

```bash
sudo ss -ltnp | grep ':8088'
sudo docker ps --filter publish=8088
```

Jika muncul pesan `address already in use`, port host sedang digunakan oleh proses lain atau konfigurasi jaringan container yang masih aktif. Jangan menghapus file socket atau menghentikan proses secara sembarang; cari proses/container pemilik port terlebih dahulu.

## 1.5 Restart policy

Restart policy yang umum:

- `no`: tidak otomatis memulai ulang.
- `on-failure`: memulai ulang jika container keluar dengan status error.
- `always`: Docker mencoba memulai ulang container ketika berhenti dan memulainya kembali saat daemon Docker dimulai.
- `unless-stopped`: seperti `always`, tetapi container yang sengaja dihentikan pengguna tidak otomatis dimulai kembali saat daemon dimulai ulang.

Contoh:

```bash
sudo docker run -d \
  --name persistcontainer \
  -p 8088:80 \
  --restart always \
  nginx:latest
```

Periksa policy:

```bash
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' persistcontainer
```

Output yang diharapkan untuk quiz adalah:

```text
always
```

Perlu dibedakan: restart policy berlaku untuk container, sedangkan `systemctl enable --now docker` mengatur agar layanan Docker berjalan dan diaktifkan saat boot. Perintah `systemctl enable docker` tidak membuat container memiliki restart policy tertentu.

## 1.6 Dockerfile dan image

Dockerfile adalah berkas instruksi untuk membangun image. Contoh alur:

```bash
sudo docker build -t myapp:latest /path/ke/direktori
sudo docker images
```

- `-t myapp:latest`: memberi nama image `myapp` dengan tag `latest`.
- Path terakhir adalah build context, yaitu direktori yang tersedia bagi instruksi build.

Jika Dockerfile berada di `/opt/quiz-22.2/container-image/Dockerfile`, build dapat dijalankan dari direktori mana pun dengan:

```bash
sudo docker build \
  -t myapp:latest \
  -f /opt/quiz-22.2/container-image/Dockerfile \
  /opt/quiz-22.2/container-image
```

## 1.7 Membuat container dengan limit memori dan bind mount

Contoh:

```bash
sudo docker run -d \
  --name student-app \
  -p 8080:80 \
  --memory 256m \
  -v /opt/quiz-22.2/sharing:/usr/share/nginx/html \
  --restart always \
  myapp:latest
```

Penjelasan:
- `--name student-app`: nama container.
- `-p 8080:80`: host port 8080 ke container port 80.
- `--memory 256m`: batas memori container 256 MiB.
- `-v HOST_PATH:CONTAINER_PATH`: memasang direktori host ke dalam container.
- `--restart always`: mengatur restart policy.
- `myapp:latest`: image yang digunakan.

Bind mount dapat membuat isi direktori host menutupi isi bawaan direktori container yang dipasang. Pastikan path sumber di host benar.

Verifikasi konfigurasi:

```bash
sudo docker inspect -f '{{.HostConfig.Memory}}' student-app
sudo docker inspect -f '{{json .HostConfig.PortBindings}}' student-app
sudo docker inspect -f '{{json .Mounts}}' student-app
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' student-app
sudo docker ps -a
```

Nilai memori dari `inspect` biasanya ditampilkan dalam byte. `268435456` byte setara dengan 256 MiB.

---

# 2. Quiz 22.1 — Working With Container

## Tugas

1. Menghapus container `testcontainer`.
2. Membuat container `persistcontainer` dari image `nginx`.
3. Memetakan port container 80 ke port host 8088.
4. Mengatur restart policy menjadi `always`.

## Langkah pengerjaan

### 1. Periksa container yang ada

```bash
sudo docker ps -a
```

### 2. Hapus `testcontainer`

```bash
sudo docker rm -f testcontainer
```

Jika muncul `No such container`, periksa kembali daftar container dengan `sudo docker ps -a`. Jangan menganggap container telah terhapus tanpa memverifikasi.

### 3. Periksa apakah port 8088 sedang digunakan

```bash
sudo ss -ltnp | grep ':8088'
sudo docker ps --filter publish=8088
```

Jika port masih dipakai, identifikasi pemiliknya terlebih dahulu. Jangan langsung menghapus container lain yang mungkin diperlukan.

### 4. Buat container

```bash
sudo docker run -d \
  --name persistcontainer \
  -p 8088:80 \
  --restart always \
  nginx:latest
```

### 5. Verifikasi

```bash
sudo docker ps -a
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' persistcontainer
sudo docker port persistcontainer
```

Policy yang diharapkan: `always`. Port mapping yang diharapkan menunjukkan port 80 container dipetakan ke host port 8088.

### 6. Jalankan penilaian

```bash
dcomctl grading quiz-22.1
```

Setelah hasil penilaian tersimpan, reset lab hanya jika memang ingin mengembalikan lingkungan ke kondisi awal:

```bash
dcomctl reset-lab quiz-22.1
```

---

# 3. Quiz 22.2 — Dockerfile

## Tugas

1. Build Dockerfile `/opt/quiz-22.2/container-image/Dockerfile`.
2. Beri tag image `myapp:latest`.
3. Buat container `student-app`.
4. Petakan port 80 container ke port 8080 host.
5. Batasi memori menjadi 256 MiB.
6. Mount `/opt/quiz-22.2/sharing` ke `/usr/share/nginx/html`.
7. Atur restart policy agar container otomatis dimulai kembali.

## Langkah pengerjaan

### 1. Build image

```bash
sudo docker build \
  -t myapp:latest \
  -f /opt/quiz-22.2/container-image/Dockerfile \
  /opt/quiz-22.2/container-image
```

Periksa image:

```bash
sudo docker images
```

Pastikan terdapat image `myapp` dengan tag `latest`.

### 2. Periksa port 8080 dan container lama

```bash
sudo ss -ltnp | grep ':8080'
sudo docker ps -a
```

Jika `student-app` sudah ada dan memang merupakan container percobaan yang boleh diganti:

```bash
sudo docker rm -f student-app
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

### 4. Verifikasi konfigurasi

```bash
sudo docker ps -a
sudo docker inspect -f '{{.HostConfig.Memory}}' student-app
sudo docker inspect -f '{{.HostConfig.RestartPolicy.Name}}' student-app
sudo docker port student-app
sudo docker inspect -f '{{json .Mounts}}' student-app
```

Ekspektasi:
- Image: `myapp:latest`
- Nama: `student-app`
- Status: berjalan (`Up`)
- Port: host `8080` ke container `80/tcp`
- Memori: `268435456` byte
- Mount: `/opt/quiz-22.2/sharing` ke `/usr/share/nginx/html`
- Restart policy: `always`

### 5. Penilaian

```bash
dcomctl grading quiz-22.2
```

**Catatan hasil praktik:** penilaian akhirnya menunjukkan **100/100**. Sebelumnya penilaian gagal karena perintah grading dijalankan tanpa akses ke Docker socket. Setelah container dibuat dengan `sudo docker`, pemeriksaan konfigurasi berhasil.

---

# 4. Troubleshooting Docker di Linux Playground

## 4.1 Permission denied pada Docker socket

Contoh pesan:

```text
permission denied while trying to connect to the Docker API at unix:///var/run/docker.sock
```

Artinya pengguna saat ini tidak memiliki izin untuk mengakses Docker daemon melalui socket.

Periksa:

```bash
sudo systemctl status docker --no-pager
ls -l /var/run/docker.sock
id
```

Untuk latihan, gunakan `sudo` pada perintah Docker:

```bash
sudo docker ps -a
sudo docker images
sudo docker inspect student-app
```

Keanggotaan grup `docker` juga memberikan akses yang sangat tinggi, setara dengan hak administratif dalam banyak konfigurasi. Jangan menambahkan pengguna ke grup tersebut tanpa memahami risikonya.

## 4.2 Paket `docker.io` tidak ditemukan, tetapi Docker tersedia

Pada lingkungan tertentu, Docker bisa dipasang melalui Snap. Karena itu, `dpkg -l docker.io` dapat menunjukkan paket tidak ditemukan, sementara:

```bash
which docker
docker --version
```

tetap menampilkan Docker yang tersedia. Periksa metode instalasi sebelum mencoba menghapus atau memasang ulang.

## 4.3 `docker.service` tidak ditemukan

Jika perintah berikut gagal:

```bash
sudo systemctl enable --now docker
```

dengan pesan `Unit file docker.service does not exist`, artinya layanan Docker belum terpasang atau tidak tersedia dengan nama tersebut. Periksa `which docker`, `docker --version`, dan layanan yang tersedia sebelum melanjutkan.

## 4.4 APT terkunci oleh unattended-upgrades

Contoh pesan:

```text
Could not get lock /var/lib/dpkg/lock-frontend
```

Penyebabnya biasanya proses pembaruan paket otomatis sedang berjalan. Periksa:

```bash
ps -p PID -o pid,stat,etime,cmd
sudo systemctl status unattended-upgrades --no-pager
sudo tail -n 30 /var/log/unattended-upgrades/unattended-upgrades.log
```

Ganti `PID` dengan nomor proses yang muncul pada pesan error.

Langkah aman:
1. Tunggu proses pembaruan selesai.
2. Periksa log untuk memastikan proses masih bekerja atau sudah berhenti.
3. Setelah proses selesai, jalankan kembali perintah APT.
4. Jika proses benar-benar macet, lakukan diagnosis lebih lanjut sebelum menghentikannya.

**Jangan menghapus file lock secara manual** saat proses paket masih berjalan. Jangan menjalankan `dpkg --configure -a` bersamaan dengan proses APT/dpkg lain.

## 4.5 Port `address already in use`

Contoh:

```text
failed to bind host port 0.0.0.0:8088/tcp: address already in use
```

Cari pemilik port:

```bash
sudo ss -ltnp | grep ':8088'
sudo docker ps --filter publish=8088
sudo docker ps -a
```

Jika port dimiliki `docker-proxy` tetapi tidak tampak pada hasil `docker ps`, periksa apakah ada container lama, proses Docker yang belum selesai, atau konflik konfigurasi. Jangan langsung mematikan proses secara paksa. Pada latihan, periksa juga apakah container dibuat tetapi statusnya `Created` akibat kegagalan bind port.

## 4.6 Memahami status container

| Status | Makna |
|---|---|
| `Up` | Container sedang berjalan |
| `Exited` | Container berhenti |
| `Created` | Container sudah dibuat, tetapi belum berhasil berjalan |
| `Restarting` | Docker sedang mencoba memulai ulang container |

Container dengan status `Created` belum berarti tugas selesai. Periksa error, port mapping, log, dan konfigurasi lalu coba jalankan kembali setelah penyebabnya diperbaiki.

---

# 5. Manajemen Akun Pengguna Linux

## 5.1 Mengapa akun pengguna diperlukan?

Linux adalah sistem multi-user. Akun yang terpisah membantu:
- Memberikan ruang kerja pribadi bagi setiap pengguna.
- Memisahkan hak akses dan tanggung jawab.
- Menjalankan layanan dengan akun khusus, bukan selalu sebagai `root`.
- Membantu audit aktivitas dan pengelolaan keamanan.

Akun biasa digunakan oleh manusia. Akun layanan seperti `daemon` dibuat agar proses tertentu berjalan dengan hak yang terbatas.

## 5.2 Atribut akun pengguna

Informasi dasar akun disimpan dalam `/etc/passwd`. Contoh:

```text
beav:x:1000:1000:Theodore Cleaver:/home/beav:/bin/bash
```

Terdapat tujuh field yang dipisahkan tanda titik dua:

1. **Username** — nama akun yang unik.
2. **Password placeholder** — biasanya `x`, yang berarti hash password disimpan di `/etc/shadow`.
3. **UID** — User ID, identitas numerik pengguna.
4. **GID** — ID grup utama pengguna.
5. **GECOS/comment** — nama lengkap atau informasi tambahan.
6. **Home directory** — direktori kerja utama pengguna.
7. **Login shell** — program yang dijalankan ketika pengguna login, misalnya `/bin/bash`.

Periksa akun:

```bash
cat /etc/passwd
id USERNAME
getent passwd USERNAME
```

## 5.3 UID dan akun sistem

Pada banyak distribusi Linux, UID di bawah `UID_MIN` digunakan untuk akun sistem, sementara akun pengguna biasa umumnya dimulai dari 1000. Nilai sebenarnya dapat berbeda dan ditentukan oleh konfigurasi distribusi, misalnya di `/etc/login.defs`.

Lihat konfigurasi:

```bash
grep '^UID_MIN' /etc/login.defs
```

## 5.4 Membuat akun dengan `useradd`

Contoh:

```bash
sudo useradd -m -s /bin/bash dexter
sudo passwd dexter
```

- `-m`: membuat home directory.
- `-s /bin/bash`: menentukan login shell.
- `passwd`: mengatur password akun.

Perintah `useradd dexter` tanpa `-m` tidak selalu membuat home directory; perilakunya bergantung pada konfigurasi sistem. Untuk akun interaktif, `-m` membuat maksud perintah lebih jelas.

Menentukan nama lengkap:

```bash
sudo useradd -m -s /bin/bash -c "Bullwinkle J Moose" bmoose
```

## 5.5 Mengubah akun dengan `usermod`

Contoh umum:

```bash
sudo usermod -s /bin/bash USERNAME
sudo usermod -d /home/newhome -m USERNAME
sudo usermod -l nama_baru nama_lama
```

Mengunci dan membuka akun:

```bash
sudo usermod -L USERNAME
sudo usermod -U USERNAME
```

`-L` mengunci password akun sehingga autentikasi berbasis password tidak dapat digunakan. Ini tidak selalu menghentikan sesi yang sudah aktif atau semua metode autentikasi lain. Untuk menonaktifkan akun secara menyeluruh, evaluasi juga shell, masa berlaku akun, kunci SSH, dan sesi yang sedang berjalan.

## 5.6 Menghapus akun dengan `userdel`

```bash
sudo userdel USERNAME
```

Untuk menghapus akun sekaligus home directory dan mail spool:

```bash
sudo userdel -r USERNAME
```

File milik pengguna yang berada di luar home directory mungkin tetap tertinggal. Sebelum menghapus akun, pastikan tidak ada data atau layanan penting yang masih bergantung padanya.

## 5.7 Akun terkunci dan `nologin`

Akun layanan dapat menggunakan shell seperti `/usr/sbin/nologin` agar tidak bisa digunakan untuk login interaktif.

Contoh entri:

```text
daemon:x:2:2:daemon:/usr/sbin:/usr/sbin/nologin
```

Periksa akun yang menggunakan `nologin`:

```bash
grep -E 'nologin|false' /etc/passwd
```

Akun layanan masih dapat digunakan untuk menjalankan proses tertentu, walaupun login interaktifnya dibatasi.

---

# 6. Password dan Masa Berlaku Akun

## 6.1 `/etc/shadow`

`/etc/shadow` menyimpan hash password dan informasi penuaan password. File ini dibatasi aksesnya karena berisi data autentikasi yang sensitif.

Contoh pemeriksaan aman:

```bash
sudo passwd -S USERNAME
sudo chage -l USERNAME
```

Jangan membagikan isi `/etc/shadow` ke publik.

Field `/etc/shadow` secara umum mencakup:
1. Username.
2. Hash password atau penanda status password.
3. Hari sejak epoch ketika password terakhir diubah.
4. Jumlah hari minimum sebelum password boleh diubah.
5. Jumlah hari maksimum sebelum password harus diganti.
6. Jumlah hari peringatan sebelum kedaluwarsa.
7. Hari setelah password kedaluwarsa sebelum akun dinonaktifkan.
8. Tanggal kedaluwarsa akun.
9. Field cadangan.

Hash password bukan password teks biasa. Format hash dapat berbeda menurut sistem dan konfigurasi; jangan mengasumsikan semua sistem memakai algoritma atau format yang sama.

## 6.2 Mengatur password dengan `passwd`

Pengguna biasa mengganti password sendiri:

```bash
passwd
```

Administrator mengganti password pengguna lain:

```bash
sudo passwd USERNAME
```

## 6.3 Password aging dengan `chage`

Lihat kebijakan saat ini:

```bash
sudo chage -l USERNAME
```

Contoh pengaturan:
```bash
sudo chage -m 14 -M 30 USERNAME
sudo chage -W 7 USERNAME
sudo chage -E 2026-12-31 USERNAME
```

- `-m 14`: minimal 14 hari sebelum password dapat diganti lagi.
- `-M 30`: password berlaku maksimal 30 hari.
- `-W 7`: peringatan 7 hari sebelum password kedaluwarsa.
- `-E`: tanggal kedaluwarsa akun.

Memaksa pengguna mengganti password saat login berikutnya:

```bash
sudo chage -d 0 USERNAME
```

Catatan: kebijakan rotasi password harus mempertimbangkan kebijakan organisasi dan risiko. Penggantian berkala yang terlalu sering dapat mendorong pengguna memilih password yang mudah ditebak atau menuliskannya di tempat yang tidak aman.

---

# 7. Restricted Shell dan Restricted Account

## 7.1 Restricted shell

Bash memiliki mode terbatas yang dapat dijalankan dengan:

```bash
bash -r
```

Mode ini membatasi beberapa operasi, seperti:
- Mengubah direktori dengan `cd`.
- Mengubah variabel tertentu seperti `PATH`, `SHELL`, dan `ENV`.
- Menjalankan perintah menggunakan path yang mengandung `/` pada kondisi tertentu.
- Melakukan beberapa bentuk redirection.

Restricted shell **bukan batas keamanan yang sempurna**. Konfigurasi yang tidak tepat dapat memungkinkan pengguna mengakses program atau lingkungan yang tidak dimaksudkan.

## 7.2 Restricted account

Akun terbatas bisa menerapkan beberapa kontrol sekaligus:
- Restricted shell.
- Daftar program yang diizinkan.
- Batas sumber daya.
- Pembatasan waktu dan lokasi login.
- Izin home directory yang ketat.

Jangan menambahkan direktori sistem secara sembarangan ke `PATH`, karena pengguna mungkin bisa menjalankan shell atau program lain yang tidak dibatasi.

Jika menggunakan `/bin/rbash`, pastikan shell tersebut tersedia dan dikonfigurasi dengan benar. Uji pembatasan dengan hati-hati sebelum menerapkannya pada sistem produksi.

---

# 8. Root dan Hak Administratif

`root` adalah akun superuser dengan hak yang sangat luas. Kesalahan saat menggunakan `root` dapat mengubah atau menghapus file sistem, merusak konfigurasi, dan menurunkan keamanan.

Praktik yang disarankan:
- Gunakan akun biasa untuk aktivitas harian.
- Gunakan `sudo` hanya untuk perintah yang memerlukan hak administratif.
- Hindari login langsung sebagai root melalui jaringan kecuali benar-benar diperlukan dan sudah diamankan.
- Gunakan log audit untuk melacak aktivitas administratif bila sesuai kebutuhan.

Contoh:

```bash
sudo systemctl status docker
sudo useradd -m newuser
sudo journalctl -u docker
```

`sudo` membantu mengaitkan eksekusi perintah administratif dengan pengguna yang menjalankannya, bergantung pada konfigurasi logging.

---

# 9. SSH dan Transfer File

## 9.1 Apa itu SSH?

SSH (Secure Shell) adalah protokol terenkripsi untuk:
- Login ke server jarak jauh.
- Menjalankan perintah pada server jarak jauh.
- Memindahkan file dengan aman.

Login menggunakan nama pengguna yang sama:

```bash
ssh remote_computer
```

Login sebagai pengguna tertentu:

```bash
ssh USERNAME@remote_computer
ssh -l USERNAME remote_computer
```

Menjalankan satu perintah jarak jauh:

```bash
ssh USERNAME@remote_computer 'uptime'
```

## 9.2 Transfer file dengan SCP

Mengirim file ke server:

```bash
scp file.txt USERNAME@remote_computer:/tmp/
```

Mengambil file dari server:

```bash
scp USERNAME@remote_computer:/tmp/file.txt .
```

Menyalin direktori secara rekursif:

```bash
scp -r some_dir USERNAME@remote_computer:/tmp/
```

Pastikan path, username, hostname, dan izin akses benar. Sintaks SCP untuk lokasi jarak jauh menggunakan format `user@host:path`.

## 9.3 Kunci SSH

Direktori konfigurasi pengguna biasanya berada di `~/.ssh/`.

File penting:
- `id_rsa` atau file private key lain: kunci privat, harus dirahasiakan.
- `id_rsa.pub` atau file public key lain: kunci publik yang boleh dibagikan.
- `authorized_keys`: daftar public key yang diizinkan untuk login ke akun tersebut.
- `known_hosts`: identitas host yang pernah dikenali.
- `config`: opsi koneksi SSH per pengguna.

Buat pasangan kunci:

```bash
ssh-keygen
```

Pasang public key ke server:

```bash
ssh-copy-id USERNAME@remote_computer
```

Simpan private key dengan aman dan jangan mengirimkannya kepada orang lain. Peringatan perubahan host key pada `known_hosts` perlu diselidiki, bukan diabaikan begitu saja, karena dapat menandakan host berubah atau risiko serangan perantara.

## 9.4 Remote graphical login

Untuk desktop jarak jauh, salah satu teknologi yang bisa digunakan adalah VNC. Server dan client VNC harus terpasang dan dikonfigurasi dengan benar. Contoh perintah tergantung implementasi:

```bash
vncserver
vncviewer localhost:2
```

VNC bukan pengganti SSH untuk administrasi command line. Akses grafis jarak jauh harus dilindungi dengan autentikasi dan jaringan yang sesuai.

---

# 10. Pengelolaan Grup Linux

## 10.1 Mengapa grup diperlukan?

Grup mengumpulkan beberapa pengguna agar mereka dapat berbagi izin terhadap file, direktori, atau sumber daya tertentu. Keanggotaan grup membantu mengelola hak akses tanpa harus memberikan izin satu per satu kepada setiap pengguna.

## 10.2 `/etc/group`

Setiap baris umumnya mengikuti format:

```text
groupname:password-placeholder:GID:user1,user2,...
```

Field:
- **Group name**: nama grup.
- **Password placeholder**: placeholder atau field password grup.
- **GID**: Group ID.
- **Members**: daftar anggota tambahan yang dipisahkan koma.

Pengguna tidak harus tercantum pada daftar anggota di akhir baris jika grup tersebut adalah primary group-nya.

Periksa grup:

```bash
cat /etc/group
getent group NAMA_GRUP
```

## 10.3 Mengelola grup

Membuat grup:

```bash
sudo groupadd projectteam
```

Membuat grup sistem:

```bash
sudo groupadd -r servicegroup
```

Mengubah nama grup atau GID:

```bash
sudo groupmod -n nama_baru nama_lama
sudo groupmod -g 1200 NAMA_GRUP
```

Menghapus grup:

```bash
sudo groupdel NAMA_GRUP
```

Jangan menghapus primary group yang masih digunakan oleh akun tanpa memeriksa dampaknya terlebih dahulu.

## 10.4 Primary group dan secondary group

- **Primary group** adalah grup utama pengguna dan umumnya menjadi grup yang digunakan ketika pengguna membuat file baru.
- **Secondary groups** adalah grup tambahan yang memberikan izin lain kepada pengguna.

Periksa keanggotaan:

```bash
groups
groups USERNAME
id USERNAME
id -Gn USERNAME
```

Menambahkan pengguna ke grup tambahan tanpa menghapus grup tambahan lainnya:

```bash
sudo usermod -aG projectteam USERNAME
```

Opsi `-a` sangat penting ketika menggunakan `-G`. Tanpa `-a`, daftar grup tambahan pengguna dapat diganti oleh daftar yang diberikan.

Setelah perubahan grup, pengguna mungkin perlu logout lalu login kembali agar sesi memperoleh keanggotaan grup terbaru.

## 10.5 User Private Groups (UPG)

Pada skema UPG, pengguna baru umumnya mendapat grup utama yang namanya sama dengan username dan GID yang sama dengan UID. Ini memudahkan pengelolaan kepemilikan file.

Contoh:
- Username: `student`
- UID: `1000`
- Primary group: `student`
- GID: `1000`

UPG tidak berarti grup tersebut tidak bisa memiliki anggota lain; anggota tambahan masih dapat ditambahkan sesuai kebutuhan.

---

# 11. Ringkasan Perintah

| Tujuan | Perintah |
|---|---|
| Melihat container berjalan | `sudo docker ps` |
| Melihat semua container | `sudo docker ps -a` |
| Melihat image | `sudo docker images` |
| Melihat detail container | `sudo docker inspect NAMA` |
| Menghapus container | `sudo docker rm -f NAMA` |
| Build image | `sudo docker build -t myapp:latest PATH` |
| Menjalankan container dengan port dan restart policy | `sudo docker run -d --name NAMA -p HOST:CONTAINER --restart always IMAGE` |
| Membuat akun | `sudo useradd -m USERNAME` |
| Mengatur password | `sudo passwd USERNAME` |
| Mengubah akun | `sudo usermod ... USERNAME` |
| Mengunci akun | `sudo usermod -L USERNAME` |
| Membuka kunci akun | `sudo usermod -U USERNAME` |
| Menghapus akun | `sudo userdel USERNAME` |
| Melihat detail UID/GID | `id USERNAME` |
| Melihat usia password | `sudo chage -l USERNAME` |
| Memaksa ganti password saat login | `sudo chage -d 0 USERNAME` |
| Login SSH | `ssh USERNAME@HOST` |
| Menyalin file via SSH | `scp FILE USERNAME@HOST:/PATH/` |
| Membuat grup | `sudo groupadd GROUP` |
| Menghapus grup | `sudo groupdel GROUP` |
| Menambahkan anggota grup tambahan | `sudo usermod -aG GROUP USERNAME` |
| Memeriksa grup pengguna | `id USERNAME` |

---

## Poin penting untuk diingat

1. Image adalah template; container adalah instance yang dijalankan dari image.
2. Port mapping memakai format `HOST:CONTAINER`.
3. Restart policy `always` adalah konfigurasi container, bukan sekadar status layanan Docker.
4. Untuk Quiz 22.2, verifikasi image, status container, port, memori, mount, dan restart policy secara terpisah.
5. Gunakan `sudo` untuk Docker bila pengguna belum memiliki izin ke Docker socket.
6. Jangan menghapus lock APT secara manual saat proses pembaruan masih berjalan.
7. `/etc/passwd` menyimpan atribut akun; `/etc/shadow` menyimpan hash password dan kebijakan penuaan.
8. Gunakan `usermod -aG` untuk menambahkan grup tanpa mengganti grup tambahan yang sudah ada.
9. `root` memiliki hak luas; gunakan `sudo` seperlunya.
10. SSH mengenkripsi koneksi, sedangkan private key harus dijaga kerahasiaannya.
