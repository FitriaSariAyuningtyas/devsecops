# LAPORAN PRAKTIKUM BAB 5

## Database Service di Docker: PostgreSQL

**Nama**: Fitria Sari Ayuningtyas  
**NIM**: 3126640013  
**Kelas**: B D4 LJ Teknik Informatika  
**Tanggal pelaksanaan**: 9 Oktober 2026

## 1. Tujuan Praktikum

Praktikum ini bertujuan untuk mempelajari cara menjalankan PostgreSQL menggunakan Docker Compose. Praktikum ini juga dilakukan untuk memahami cara membuat tabel dan memasukkan data awal secara otomatis, mengakses database melalui terminal dan pgAdmin, serta menyimpan data menggunakan Docker volume.

Selain itu, praktikum ini mencakup pengelolaan password database menggunakan Docker secrets, pembuatan backup menggunakan `pg_dump`, dan pemulihan database menggunakan `pg_restore`. Melalui pengujian tersebut, dapat diketahui apakah layanan database berjalan dengan baik dan apakah data yang telah dicadangkan dapat dipulihkan kembali.

## 2. Dasar Teori

### 2.1 PostgreSQL

PostgreSQL merupakan sistem manajemen basis data relasional yang digunakan untuk menyimpan dan mengelola data. Pada praktikum ini, PostgreSQL digunakan untuk menyimpan data mahasiswa dalam tabel `students`. Data tersebut kemudian diperiksa menggunakan perintah SQL melalui terminal dan pgAdmin.

### 2.2 Docker Compose

Docker Compose digunakan untuk mengatur beberapa layanan container melalui satu file konfigurasi. Pada praktikum ini, Docker Compose digunakan untuk menjalankan PostgreSQL dan pgAdmin, mengatur jaringan komunikasi antarlayanan, menentukan volume penyimpanan, serta mengatur ketergantungan antara kedua layanan.

### 2.3 Docker Volume

Docker volume digunakan untuk menyimpan data di luar siklus hidup container. Dengan volume, data PostgreSQL dapat tetap tersedia ketika container dihentikan atau dibuat ulang, selama volume yang menyimpan data tersebut tidak dihapus.

### 2.4 Docker Secrets

Docker secrets digunakan untuk menyediakan informasi sensitif, seperti password database, kepada layanan yang membutuhkannya. Pada praktikum ini, password PostgreSQL disimpan dalam file terpisah dan diberikan kepada container melalui `/run/secrets/postgres_password`. Dengan cara ini, password PostgreSQL tidak perlu ditulis langsung sebagai nilai `POSTGRES_PASSWORD` di file Compose.

Penggunaan secrets melalui file pada Docker Compose lokal membantu memisahkan password dari konfigurasi layanan. Namun, file rahasia tetap perlu dilindungi menggunakan pengaturan izin akses yang sesuai dan tidak boleh ikut diunggah ke repository publik.

### 2.5 Backup dan Restore Database

Backup merupakan proses membuat salinan data database agar dapat digunakan kembali apabila terjadi kehilangan atau kerusakan data. Pada praktikum ini, backup dilakukan menggunakan `pg_dump` dengan format custom. File backup kemudian diperiksa menggunakan checksum SHA-256.

Restore merupakan proses mengembalikan data dari file backup ke database. Pada praktikum ini, proses restore dilakukan menggunakan `pg_restore` pada database pengujian terpisah. Cara tersebut digunakan untuk memeriksa apakah data dapat dipulihkan tanpa mengubah database utama.

## 3. Alat dan Bahan

Alat dan bahan yang digunakan dalam praktikum ini ditunjukkan pada tabel berikut.

| No. | Alat/Bahan | Keterangan |
|---|---|---|
| 1 | Ubuntu 26.04 LTS pada WSL2 | Lingkungan untuk menjalankan praktikum. |
| 2 | Docker Engine Community 29.7.2 | Menjalankan layanan dalam container. |
| 3 | Docker Compose 5.5.0 | Mengatur konfigurasi dan menjalankan beberapa layanan container. |
| 4 | PostgreSQL 16 (`postgres:16-alpine`) | Layanan database untuk menyimpan data mahasiswa. |
| 5 | pgAdmin (`dpage/pgadmin4`) | Antarmuka berbasis web untuk mengakses dan mengelola database. |
| 6 | File `compose.yaml` | Konfigurasi layanan, jaringan, volume, dan secrets. |
| 7 | File `init/01-schema.sql` | Membuat tabel `students`, memasukkan data awal, dan membuat indeks. |
| 8 | File `secrets/postgres_password.txt` | Menyimpan password PostgreSQL untuk digunakan sebagai Docker secret. |
| 9 | File `scripts/backup.sh` | Menjalankan backup database dan membuat checksum SHA-256. |
| 10 | Terminal dan browser | Menjalankan perintah Docker serta mengakses pgAdmin. |
