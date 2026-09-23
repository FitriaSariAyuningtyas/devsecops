# LAPORAN PRAKTIKUM BAB 3

## Docker Network, Volume, Bind Mount, tmpfs, dan Docker Compose

**Nama**: Fitria Sari Ayuningtyas  
**NIM**: 3126640013  
**Kelas**: B D4 LJ Teknik Informatika  
**Tanggal pelaksanaan**: 23 September 2026

## 1. Tujuan Praktikum

Praktikum Bab 3 bertujuan memahami komunikasi antar-container menggunakan user-defined bridge network serta memahami penggunaan named volume, bind mount, dan tmpfs sebagai mekanisme penyimpanan data pada container.

Praktikum ini juga bertujuan menerapkan Docker Compose untuk menjalankan aplikasi multi-container yang terdiri dari Nginx, Flask, dan PostgreSQL. Selain itu, praktikum melatih proses verifikasi, troubleshooting, serta identifikasi risiko keamanan dan operasional pada lingkungan container.

## 2. Dasar Teori Singkat

Docker network digunakan untuk mengatur komunikasi antar-container. User-defined bridge memungkinkan container dalam network yang sama saling berkomunikasi menggunakan nama container dan memberikan konfigurasi network yang lebih terkontrol.

Docker menyediakan beberapa mekanisme penyimpanan data. Named volume dikelola oleh Docker dan dapat mempertahankan data meskipun container dihapus. Bind mount memetakan direktori pada host secara langsung ke dalam container sehingga perubahan file pada host dapat langsung digunakan oleh container. Sementara itu, tmpfs menyimpan data secara sementara di memory dan data tersebut tidak dipertahankan setelah container di-restart.

Docker Compose digunakan untuk mendefinisikan dan menjalankan beberapa service dalam satu konfigurasi. Pada praktikum ini, Docker Compose digunakan untuk menjalankan Nginx sebagai web server dan reverse proxy, Flask sebagai backend, serta PostgreSQL sebagai database. Konfigurasi juga menggunakan network frontend dan backend serta healthcheck untuk memastikan PostgreSQL siap menerima koneksi sebelum service Flask digunakan.

## 3. Alat dan Lingkungan

| **Komponen**             | **Hasil identifikasi**            |
| ------------------------ | --------------------------------- |
| Operating system         | Ubuntu pada WSL2                  |
| Docker Engine            | 29.7.2                            |
| Docker Compose           | v5.5.0                            |
| Web Server               | Nginx Alpine                      |
| Backend                  | Flask 3.1.2                       |
| Application Server       | Gunicorn 23.0.0                   |
| Database                 | PostgreSQL 16 Alpine              |
| Direktori kerja          | `~/docker-lab/bab-3/`             |
| Direktori Docker Compose | `~/docker-lab/bab-3/compose-lab/` |

Praktikum dijalankan pada lingkungan Ubuntu melalui WSL2. Docker Engine dan Docker Compose digunakan untuk menjalankan container secara lokal. Aplikasi multi-container terdiri dari Nginx, Flask, dan PostgreSQL.

## 4. Langkah Praktikum

### 4.1 User-defined Bridge Network

Perintah yang digunakan untuk membuat user-defined bridge network dan menjalankan dua container pada network yang sama:

```bash
mkdir -p ~/docker-lab/bab-3
cd ~/docker-lab/bab-3

docker network create --driver bridge --subnet 172.20.0.0/16 lab-net

docker run -d --name server-a --network lab-net nginx:alpine
docker run -d --name server-b --network lab-net nginx:alpine
```

Pengujian komunikasi antar-container dilakukan dengan:

```bash
docker exec server-a ping -c 3 server-b
```

Pemeriksaan konfigurasi network dilakukan menggunakan:

```bash
docker network inspect lab-net
```

**Bukti pengujian:**

![Gambar 1 - User-defined bridge network](assets/ss01-network1.jpg)

![Gambar 1 - User-defined bridge network](assets/ss01-network2.jpg)
Hasil pengujian menunjukkan bahwa `server-a` berhasil berkomunikasi dengan `server-b` menggunakan nama container. Network `lab-net` menggunakan driver `bridge` dengan subnet `172.20.0.0/16`. Pengujian `ping` menghasilkan 3 paket diterima dari 3 paket yang dikirim dengan `0% packet loss`.

### 4.2 Named Volume

Named volume digunakan untuk menyimpan data secara terpisah dari lifecycle container. Pada pengujian ini, volume `data-vol` digunakan untuk menyimpan file `log.txt` yang dibuat oleh container `writer`.

Perintah yang digunakan:

```bash
docker volume create data-vol

docker run -d --name writer \
  -v data-vol:/app/data \
  alpine:3.20 \
  sh -c "while true; do date >> /app/data/log.txt; sleep 5; done"
```

Setelah beberapa saat, container `writer` dihapus untuk menguji apakah data pada volume tetap tersedia:

```bash
sleep 15
docker rm -f writer

docker run --rm \
  -v data-vol:/data \
  alpine:3.20 \
  cat /data/log.txt
```

Backup isi named volume dilakukan dengan:

```bash
docker run --rm \
  -v data-vol:/source:ro \
  -v $(pwd):/backup \
  alpine:3.20 \
  tar czf /backup/data-vol-backup.tar.gz -C /source .

ls -lh data-vol-backup.tar.gz
```

**Bukti pengujian:**

![Gambar 2 - Named volume](assets/ss02-volume1.jpg)

![Gambar 2 - Named volume](assets/ss02-volume2.jpg)

Hasil pengujian menunjukkan bahwa file `log.txt` masih dapat dibaca setelah container `writer` dihapus. File `data-vol-backup.tar.gz` juga berhasil dibuat sebagai hasil backup isi volume. Hal ini menunjukkan bahwa named volume dapat mempertahankan data meskipun container yang menggunakannya telah dihapus.



