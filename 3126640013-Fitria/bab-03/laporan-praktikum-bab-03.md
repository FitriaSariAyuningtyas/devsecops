# LAPORAN PRAKTIKUM BAB 3

## Docker Network, Volume, Bind Mount, tmpfs, dan Docker Compose

**Nama:** Fitria Sari Ayuningtyas
**NIM:** 3126640013
**Kelas:** B D4 LJ Teknik Informatika
**Tanggal pelaksanaan:** 23 September 2026

---

## 1. Tujuan Praktikum

Praktikum Bab 3 bertujuan memahami komunikasi antar-container menggunakan user-defined bridge network serta membandingkan penggunaan named volume, bind mount, dan tmpfs sebagai mekanisme penyimpanan data pada container.

Praktikum ini juga bertujuan menerapkan Docker Compose untuk menjalankan aplikasi multi-container yang terdiri dari Nginx, Flask, dan PostgreSQL. Selain itu, praktikum dilakukan untuk melatih proses verifikasi, troubleshooting, serta identifikasi risiko keamanan dan operasional pada lingkungan container.

---

## 2. Dasar Teori Singkat

Docker network digunakan untuk mengatur komunikasi antar-container. User-defined bridge menyediakan network yang dapat dikonfigurasi sesuai kebutuhan dan memungkinkan container saling menemukan menggunakan nama container.

Docker menyediakan beberapa mekanisme penyimpanan data. Named volume dikelola oleh Docker dan dapat mempertahankan data meskipun container dihapus. Bind mount memetakan direktori pada host secara langsung ke dalam container sehingga perubahan file pada host dapat langsung digunakan oleh container. Sementara itu, tmpfs menyimpan data secara sementara di memory dan tidak digunakan untuk penyimpanan data yang membutuhkan persistensi.

Docker Compose digunakan untuk mendefinisikan dan menjalankan beberapa service dalam satu konfigurasi. Pada praktikum ini, Compose digunakan untuk menjalankan Nginx, Flask, dan PostgreSQL dengan network frontend dan backend serta healthcheck pada PostgreSQL.

---

## 3. Alat dan Lingkungan

| Komponen           | Spesifikasi      |
| ------------------ | ---------------- |
| OS                 | Ubuntu pada WSL2 |
| Docker Engine      | 29.7.2           |
| Docker Compose     | v5.5.0           |
| Web Server         | Nginx Alpine     |
| Backend            | Flask            |
| Application Server | Gunicorn 23.0.0  |
| Database           | PostgreSQL 16    |

Direktori kerja praktikum:

```text
~/docker-lab/bab-3/
```

Implementasi Docker Compose berada pada:

```text
~/docker-lab/bab-3/compose-lab/
```

---

## 4. Langkah Praktikum dan Hasil Pengujian

### 4.1 User-defined Bridge Network

Network `lab-net` dibuat menggunakan driver bridge dengan subnet `172.20.0.0/16`. Dua container, yaitu `server-a` dan `server-b`, dijalankan pada network yang sama.

Perintah pengujian:

```bash
docker network inspect lab-net
docker exec server-a ping -c 3 server-b
```

**Gambar 1. Pengujian user-defined bridge network**

![Gambar 1 - User-defined bridge network](assets/ss01-network.png)

Hasil `docker network inspect` menunjukkan bahwa network `lab-net` menggunakan driver `bridge` dengan subnet `172.20.0.0/16`. Container `server-a` memiliki alamat IP `172.20.0.2`, sedangkan `server-b` memiliki alamat IP `172.20.0.3`.

Pengujian menggunakan `ping` menunjukkan bahwa `server-a` berhasil melakukan resolusi nama `server-b` menjadi alamat IP `172.20.0.3`. Sebanyak 3 paket berhasil diterima dari 3 paket yang dikirim dengan `0% packet loss`. Hal ini menunjukkan bahwa container pada user-defined bridge dapat berkomunikasi menggunakan nama container.

---

### 4.2 Named Volume

Named volume `data-vol` digunakan untuk menyimpan file `log.txt` yang dibuat oleh container `writer`. Setelah container `writer` dihapus, isi volume diperiksa kembali menggunakan container baru. Proses backup juga dilakukan dengan membuat file `data-vol-backup.tar.gz`.

Perintah yang digunakan:

```bash
docker volume create data-vol
docker run -d --name writer -v data-vol:/app/data alpine:3.20 \
  sh -c "while true; do date >> /app/data/log.txt; sleep 5; done"
```

Setelah container dihapus, isi volume diperiksa kembali dan dilakukan proses backup.

**Gambar 2. Pengujian persistence dan backup named volume**

![Gambar 2 - Named volume](assets/ss02-volume.png)

Hasil pengujian menunjukkan bahwa file `log.txt` masih dapat dibaca setelah container `writer` dihapus. Hal ini menunjukkan bahwa named volume mempertahankan data secara terpisah dari lifecycle container. File `data-vol-backup.tar.gz` juga berhasil dibuat sebagai hasil backup isi volume.

---

### 4.3 Bind Mount

Bind mount digunakan untuk memetakan direktori pada host ke dalam container Nginx. File `index.html` dibuat pada host dan kemudian dibaca dari dalam container.

Pengujian dilakukan dengan mengubah isi file pada host dari:

```text
Hello from host!
```

menjadi:

```text
Hello after host change!
```

**Gambar 3. Pengujian bind mount**

![Gambar 3 - Bind mount](assets/ss03-bind-mount.png)

Perubahan isi file pada host dapat langsung dibaca dari dalam container tanpa melakukan rebuild image. Hal ini menunjukkan bahwa bind mount memungkinkan container menggunakan file yang dikelola langsung dari host.

---

### 4.4 tmpfs

tmpfs digunakan pada direktori `/app/tmp`. File `test.txt` dibuat dan dibaca sebelum container di-restart. Setelah restart, isi direktori diperiksa kembali.

**Gambar 4. Pengujian tmpfs sebelum dan setelah restart**

![Gambar 4 - tmpfs](assets/ss04-tmpfs.png)

File `test.txt` dapat dibaca sebelum container di-restart. Setelah restart, file tersebut tidak ditemukan pada direktori `/app/tmp`. Hasil ini menunjukkan bahwa data pada tmpfs bersifat sementara dan tidak dipertahankan setelah container di-restart.

---

### 4.5 Compose Multi-container

Docker Compose digunakan untuk menjalankan tiga service, yaitu Nginx sebagai web server/reverse proxy, Flask sebagai backend, dan PostgreSQL sebagai database. Service `web` terhubung ke network `frontend`, sedangkan `app` terhubung ke network `frontend` dan `backend`. PostgreSQL berada pada network `backend` dan menggunakan named volume `pg-data`.

#### 4.5.1 Verifikasi Service

Perintah yang digunakan:

```bash
docker compose ps
```

**Gambar 5. Status service Docker Compose**

![Gambar 5 - Docker Compose ps](assets/ss05-compose-ps.png)

Hasil `docker compose ps` menunjukkan bahwa ketiga service berhasil berjalan. Service PostgreSQL berstatus `healthy`, sedangkan Nginx dipublikasikan melalui `127.0.0.1:8080`.

#### 4.5.2 Pengujian Endpoint Utama

Perintah:

```bash
curl http://localhost:8080/
```

**Gambar 6. Pengujian koneksi Nginx, Flask, dan PostgreSQL**

![Gambar 6 - Pengujian endpoint utama](assets/ss06-curl-root.png)

Pengujian endpoint utama menghasilkan response dengan status `ok` dan menampilkan informasi versi PostgreSQL. Hal ini menunjukkan bahwa Nginx berhasil meneruskan request ke Flask dan Flask berhasil terhubung ke PostgreSQL.

#### 4.5.3 Pengujian Health Check

Perintah:

```bash
curl -i http://localhost:8080/health
```

**Gambar 7. Pengujian health check aplikasi**

![Gambar 7 - Health check](assets/ss07-health.png)

Pengujian endpoint `/health` menghasilkan `HTTP 200 OK` dengan response `{"status":"healthy"}`. Hasil tersebut menunjukkan bahwa aplikasi dapat melakukan pemeriksaan koneksi database dengan baik.

#### 4.5.4 Pengujian Halaman Statis

Halaman statis diuji melalui browser menggunakan alamat:

```text
http://localhost:8080/static.html
```

**Gambar 8. Pengujian halaman statis melalui Nginx**

![Gambar 8 - Browser static.html](assets/ss08-browser.png)

Halaman `static.html` berhasil ditampilkan melalui browser dengan isi halaman Docker Compose Bab 3. Hal ini menunjukkan bahwa konfigurasi Nginx dan bind mount pada Docker Compose dapat digunakan untuk menyajikan halaman statis.

---

## 5. Evaluasi dan Latihan Mandiri

### 1. Mengapa user-defined bridge lebih baik daripada default bridge?

User-defined bridge menyediakan name resolution antar-container secara otomatis dan memberikan konfigurasi network yang lebih terkontrol dibandingkan default bridge.

### 2. Apa risiko bind mount terhadap keamanan host?

Bind mount dapat memberikan container akses langsung ke file atau direktori host. Jika mount menggunakan mode writable atau direktori yang terlalu luas, container yang dikompromikan berpotensi mengubah atau merusak data host.

### 3. Apa perbedaan `docker compose down` dan `docker compose down -v`?

`docker compose down` menghentikan dan menghapus container serta network yang dibuat Compose, sedangkan `docker compose down -v` juga menghapus named volume yang digunakan oleh Compose. Karena itu, `down -v` dapat menyebabkan data persistent pada volume hilang.

### 4. Kapan `depends_on` dengan healthcheck lebih tepat?

`depends_on` dengan healthcheck lebih tepat ketika service bergantung pada service lain yang harus benar-benar siap digunakan, misalnya aplikasi Flask yang membutuhkan PostgreSQL sudah siap menerima koneksi.

### 5. Bagaimana strategi backup volume database produksi?

Backup dilakukan secara berkala menggunakan mekanisme yang konsisten dengan database, disimpan pada lokasi terpisah, dan diuji dengan proses restore secara berkala. Backup juga sebaiknya memiliki retensi dan perlindungan akses yang sesuai.

---

## 6. Analisis

### 6.1 Masalah dan Diagnosis

Pada pengujian awal endpoint `http://localhost:8080/static.html`, Nginx mengembalikan response **404 Not Found**. Untuk mendiagnosis masalah tersebut, dilakukan pemeriksaan terhadap isi direktori `html` menggunakan perintah `ls -la html`.

Hasil pemeriksaan menunjukkan bahwa pada direktori tersebut hanya terdapat file `index.html`, sedangkan konfigurasi Nginx mengarahkan endpoint `/static.html` ke file `static.html`.

Masalah kemudian diperbaiki dengan menambahkan file `static.html` pada direktori `html`. Setelah dilakukan pengujian ulang, endpoint `/static.html` berhasil menampilkan halaman HTML. Hal ini menunjukkan bahwa masalah disebabkan oleh ketidaksesuaian nama file antara konfigurasi Nginx dan file yang tersedia pada direktori host.

**Gambar 9. Cuplikan log Docker Compose**

![Gambar 9 - Docker Compose logs](assets/ss09-logs.png)

Cuplikan log menunjukkan bahwa PostgreSQL berhasil siap menerima koneksi, Gunicorn berhasil menjalankan aplikasi Flask, dan Nginx berhasil melayani request. Request terhadap endpoint aplikasi juga menghasilkan status `200`.

### 6.2 Risiko Keamanan dan Operasional

Penggunaan bind mount dapat menjadi risiko apabila container diberikan akses tulis terhadap direktori host yang terlalu luas. Jika container mengalami kompromi, data pada direktori host yang ter-mount berpotensi diubah atau dihapus. Oleh karena itu, akses mount sebaiknya dibatasi dan menggunakan mode read-only apabila container hanya membutuhkan akses baca.

Credential database seperti `DB_PASS` dan `POSTGRES_PASSWORD` juga masih ditulis langsung pada file Compose. Cara tersebut kurang sesuai untuk lingkungan production karena credential dapat ikut tersimpan pada repository atau konfigurasi yang dibagikan.

### 6.3 Rekomendasi untuk Production-like Environment

Untuk lingkungan production-like, credential database sebaiknya dikelola menggunakan secrets atau mekanisme secret management sehingga tidak ditulis langsung pada file Compose. Bind mount juga perlu dibatasi hanya pada direktori yang diperlukan dan menggunakan mode read-only apabila container hanya membutuhkan akses baca.

Penggunaan Gunicorn dan user non-root yang sudah diterapkan perlu dipertahankan. Selain itu, versi image Docker sebaiknya dipin menggunakan versi atau digest tertentu agar deployment lebih konsisten dan dapat direproduksi. Backup database juga perlu dilakukan secara berkala dan proses restore perlu diuji secara rutin.

---

## 7. Kesimpulan

Praktikum Bab 3 berhasil menerapkan user-defined bridge network, named volume, bind mount, tmpfs, dan Docker Compose. Hasil pengujian menunjukkan bahwa container pada user-defined network dapat melakukan komunikasi menggunakan nama container, named volume mempertahankan data setelah container dihapus, bind mount meneruskan perubahan file host, sedangkan tmpfs tidak mempertahankan data setelah container di-restart.

Docker Compose berhasil menjalankan Nginx, Flask, dan PostgreSQL dengan konfigurasi network frontend dan backend serta healthcheck pada PostgreSQL. Pengujian endpoint utama, health check, dan halaman statis menunjukkan bahwa seluruh service dapat bekerja dan berkomunikasi sesuai rancangan.

---

## 8. Referensi

Ferry Astika Saputra, “Bab 3 — Docker Network, Volume, Bind Mount, tmpfs, dan Compose,” repository DevSecOps PENS, `bab-03.md`, diakses 23 September 2026.

Docker Documentation, “Networking overview.”

Docker Documentation, “Volumes.”

Docker Documentation, “Bind mounts.”

Docker Documentation, “tmpfs mounts.”

Docker Documentation, “Docker Compose.”
