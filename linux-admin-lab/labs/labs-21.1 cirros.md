# Lab 21.1 — Instalasi VM Menggunakan QEMU/KVM

## Tujuan
Memahami pemasangan perangkat virtualisasi, membuat VM CirrOS dari image yang tersedia, dan mengelolanya dengan `virsh`.

## 1. Memasang paket virtualisasi

```bash
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virtinst virt-manager -y
sudo systemctl is-active libvirtd
sudo usermod -aG libvirt "$USER"
sudo usermod -aG kvm "$USER"
```

Status layanan yang aktif biasanya ditunjukkan oleh `active`. Perubahan grup pengguna umumnya berlaku setelah login ulang.

## 2. Mengunduh image CirrOS

CirrOS adalah Linux sangat ringan yang sering dipakai untuk menguji image VM dan lingkungan cloud. CirrOS adalah sistem operasi guest, bukan hypervisor.

```bash
wget http://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img
sudo mv cirros-0.6.2-x86_64-disk.img /var/lib/libvirt/images/
ls -lh /var/lib/libvirt/images/cirros-0.6.2-x86_64-disk.img
```

## 3. Membuat VM `cirros`

Contoh perintah:

```bash
sudo virt-install \
  --name cirros \
  --vcpus 1 \
  --ram 1024 \
  --disk path=/var/lib/libvirt/images/cirros-0.6.2-x86_64-disk.img,format=qcow2 \
  --import \
  --osinfo detect=on,require=off \
  --autostart \
  --network network=default \
  --noautoconsole
```

Keterangan:
- `--name cirros`: nama domain VM.
- `--vcpus 1`: satu vCPU.
- `--ram 1024`: RAM 1024 MiB.
- `--disk`: menentukan image disk.
- `--import`: menggunakan image yang sudah ada.
- `--osinfo detect=on,require=off`: mencoba deteksi OS dan mengizinkan nilai generik.
- `--autostart`: VM dinyalakan otomatis saat host boot.
- `--network network=default`: memakai jaringan virtual default libvirt jika tersedia.
- `--noautoconsole`: tidak langsung membuka konsol VM.

**Catatan:** Pastikan format image benar. Jika muncul error terkait format atau ukuran disk, periksa dengan `qemu-img info /var/lib/libvirt/images/cirros-0.6.2-x86_64-disk.img` dan sesuaikan konfigurasi. Jika muncul `KVM acceleration not available, using 'qemu'`, VM dapat berjalan dengan emulasi QEMU tetapi biasanya lebih lambat.

## 4. Mengelola VM dengan `virsh`

```bash
sudo virsh list
sudo virsh list --all
sudo virsh dominfo cirros
sudo virsh start cirros
sudo virsh shutdown cirros
sudo virsh reboot cirros
sudo virsh autostart --disable cirros
```

`virsh list` menampilkan VM yang sedang berjalan, sedangkan `virsh list --all` juga menampilkan VM yang mati. `virsh destroy cirros` mematikan VM secara paksa dan sebaiknya hanya digunakan jika shutdown normal gagal.

## 5. Apakah CirrOS bisa diakses melalui SSH?

Bisa jika konfigurasi guest, jaringan, alamat IP, layanan SSH, dan kredensialnya mendukung. Periksa dahulu:

```bash
sudo virsh domifaddr cirros
sudo virsh net-list --all
```

Jika `domifaddr` tidak menampilkan IP, belum tentu VM tidak memiliki alamat; metode pelaporan alamat mungkin tidak tersedia. Periksa jaringan dan dokumentasi image untuk mengetahui akun/kredensial yang benar.

## 6. Memahami arsitektur

- **Host:** mesin yang menjalankan lingkungan virtualisasi.
- **QEMU:** menyediakan emulasi perangkat keras dan menjalankan VM.
- **KVM:** memanfaatkan dukungan virtualisasi CPU dari Linux jika tersedia.
- **libvirt:** lapisan manajemen VM, jaringan, dan penyimpanan.
- **`virsh`:** alat command-line untuk mengelola domain libvirt.
- **CirrOS:** sistem operasi guest di dalam VM.

VM `cirros` dan `webserver` merupakan domain berbeda yang dikelola oleh host yang sama. VM tidak dibuat di dalam CirrOS.

## Ringkasan

Lab ini mengenalkan pemasangan QEMU/KVM dan libvirt, penggunaan image CirrOS, pembuatan VM dengan `virt-install`, serta pengelolaan VM melalui `virsh`. CirrOS berguna untuk pengujian ringan dan bukan pilihan umum untuk server produksi.
