# LAPORAN PRAKTIKUM BAB 4

## Fondasi Teoretis dan Kerangka Kerja DevSecOps

**Nama**: Fitria Sari Ayuningtyas  
**NIM**: 3126640013  
**Kelas**: B D4 LJ Teknik Informatika  
**Tanggal pelaksanaan**: 4 Oktober 2026

## 1. Tujuan Praktikum

Praktikum Bab 4 bertujuan untuk memahami penerapan web service container menggunakan Apache, Nginx, dan Flask dalam lingkungan Docker. Praktikum ini mencakup pembuatan web server dengan konfigurasi custom, penerapan virtual host berbasis nama, penggunaan Nginx sebagai reverse proxy, serta penerapan TLS menggunakan sertifikat self-signed.

Selain itu, praktikum bertujuan untuk memahami penerapan keamanan pada web service container, khususnya pembatasan akses backend agar tidak langsung dipublikasikan ke host, penggunaan Docker network untuk komunikasi antar-container, pengamanan private key, penerapan security headers, healthcheck, serta pencatatan access log dan error log sebagai bukti operasional sistem.

### 2.1 Apache HTTP Server

Apache HTTP Server merupakan web server yang digunakan untuk menerima dan melayani permintaan HTTP dari client. Apache dapat digunakan untuk menyajikan halaman web statis maupun menjalankan aplikasi web melalui berbagai modul dan konfigurasi. Dalam praktikum ini, Apache digunakan sebagai web server backend yang melayani halaman HTML dan tidak diekspos secara langsung ke host.

### 2.2 Nginx

Nginx merupakan web server yang menggunakan pendekatan event-driven dalam menangani koneksi. Model ini memungkinkan worker process menangani banyak koneksi secara asynchronous sehingga Nginx sesuai digunakan untuk menangani koneksi client dalam jumlah besar. Pada praktikum ini, Nginx ditempatkan sebagai public boundary yang menerima koneksi HTTP/HTTPS dari host, menangani TLS, serta meneruskan request ke service backend.

### 2.3 Perbedaan Apache dan Web Server Event-Driven

Apache dan Nginx memiliki pendekatan yang berbeda dalam menangani koneksi client. Apache secara umum menggunakan model berbasis process atau thread melalui Multi-Processing Module (MPM), sedangkan Nginx menggunakan pendekatan event-driven dan asynchronous.

Pada Apache, setiap koneksi dapat ditangani menggunakan process atau thread sesuai MPM yang digunakan. Pendekatan ini fleksibel dan mendukung berbagai kebutuhan aplikasi web, tetapi penggunaan resource dapat meningkat ketika jumlah koneksi yang harus ditangani semakin banyak.

Sementara itu, Nginx menggunakan event loop sehingga satu worker dapat menangani banyak koneksi secara bersamaan tanpa membuat satu thread atau process untuk setiap koneksi. Oleh karena itu, Nginx banyak digunakan sebagai reverse proxy, load balancer, dan server yang menangani koneksi client.

Dalam praktikum ini kedua pendekatan tersebut digunakan secara bersamaan. Nginx berada di bagian depan sebagai reverse proxy dan terminasi TLS, sedangkan Apache digunakan sebagai web server backend. Dengan pembagian tersebut, client tidak perlu mengakses Apache secara langsung karena seluruh request masuk melalui Nginx.

### 2.4 Reverse Proxy

Reverse proxy merupakan server perantara yang menerima request dari client kemudian meneruskannya ke server backend. Pada praktikum ini, Nginx berfungsi sebagai reverse proxy yang meneruskan request berdasarkan path.

Request ke `/` diteruskan menuju Apache, sedangkan request ke `/api/` diteruskan menuju Flask. Dengan konfigurasi tersebut, backend dapat tetap berada di dalam Docker network dan tidak perlu membuka port secara langsung ke host.

### 2.5 TLS dan Private Key

Transport Layer Security (TLS) digunakan untuk memberikan enkripsi pada komunikasi antara client dan server. Pada praktikum ini digunakan sertifikat self-signed untuk kebutuhan pengujian lokal. Sertifikat dibuat menggunakan OpenSSL dengan nama `lab.crt`, sedangkan private key disimpan dalam file `lab.key`.

Private key merupakan komponen sensitif dalam mekanisme TLS sehingga harus diberikan permission yang ketat. Pada praktikum, permission private key diatur menjadi `600`, sehingga hanya owner yang memiliki hak baca dan tulis terhadap file tersebut. File sertifikat dan private key kemudian di-mount ke container Nginx dalam mode read-only.

Penyimpanan tersebut mengurangi risiko perubahan atau modifikasi private key oleh proses di dalam container.

### 2.6 Peta Konsep Penyimpanan Private Key

Penyimpanan private key pada praktikum menerapkan prinsip least privilege dan read-only mount. Private key dibuat dan disimpan pada host, kemudian digunakan oleh Nginx melalui volume mount dengan mode read-only.

![Peta Konsep Penyimpanan Private Key](./assets/Peta Konsep Penyimpanan Private Key.png)

**Gambar 1. Peta konsep penyimpanan dan penggunaan private key TLS.**

## 3. Alat dan Lingkungan

Praktikum dilakukan menggunakan lingkungan WSL2 Ubuntu dengan Docker sebagai platform containerization. Adapun alat dan lingkungan yang digunakan adalah sebagai berikut.

| No. | Alat/Lingkungan | Keterangan |
|---|---|---|
| 1 | WSL2 Ubuntu | Lingkungan Linux untuk menjalankan praktikum |
| 2 | Docker Engine | 29.7.2 |
| 3 | Docker Compose | 5.5.0 |
| 4 | OpenSSL | 3.5.5 |
| 5 | cURL | 8.18.0 |
| 6 | Nginx | nginx:alpine |
| 7 | Apache | httpd:2.4-alpine |
| 8 | Python | 3.12-slim pada container Flask |
| 9 | Flask | 3.1.2 |
| 10 | Gunicorn | 23.0.0 |

## 4. Langkah Praktikum

### 4.1 Persiapan Struktur Project

Tahap pertama dilakukan dengan membuat struktur direktori untuk menyimpan konfigurasi Docker Compose, konfigurasi Nginx dan Apache, aplikasi Flask, sertifikat TLS, serta log Nginx.

![Struktur Project](./gambar/SS-01.png)

**Gambar 2. Struktur direktori project Docker Lab Bab 4.**
