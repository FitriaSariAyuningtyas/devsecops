# LAPORAN PRAKTIKUM BAB 6
## CENTRALIZED LOGGING DENGAN FLUENT BIT DAN POSTGRESQL

**Disusun untuk Memenuhi Tugas Praktikum Mata Kuliah DevOps**

**Topik Praktikum:** Implementasi Centralized Logging menggunakan Fluent Bit dan PostgreSQL pada Sistem PMB DevSecOps

**Nama:** Fitria Sari Ayuningtyas

**NIM:** 3126640013

**Kelas:** STr LJ Teknik Informatika B

# 1. Tujuan Praktikum

Praktikum Bab 6 bertujuan untuk memahami penerapan centralized logging menggunakan Fluent Bit dan PostgreSQL sebagai media pengumpulan serta penyimpanan log dari beberapa layanan aplikasi. Melalui praktikum ini, mahasiswa diharapkan mampu:

1. Memahami perbedaan fungsi database transaksi dan database penyimpanan log terpusat.
2. Mengimplementasikan pengumpulan log dari beberapa layanan menggunakan Fluent Bit.
3. Memahami penggunaan structured logging untuk mencatat aktivitas aplikasi secara terstruktur.
4. Melakukan korelasi log antar layanan menggunakan `request_id`.
5. Menganalisis urutan kejadian atau timeline insiden berdasarkan data log menggunakan query SQL.
6. Menguji mekanisme retry, buffering, dan recovery ketika database log tidak tersedia.
7. Mengidentifikasi kemungkinan kehilangan log serta mengumpulkan bukti pengujian tanpa mengekspos data sensitif.

# 2. Dasar Teori Singkat

### 2.1 Centralized Logging

Centralized logging merupakan mekanisme pengumpulan log dari beberapa layanan aplikasi ke dalam satu tempat penyimpanan terpusat. Dengan mekanisme ini, aktivitas dari setiap layanan dapat dipantau dan dianalisis melalui satu sumber data. Pada praktikum ini, centralized logging digunakan untuk mengumpulkan log dari layanan `registration-api`, `auth-service`, dan `verification-worker`.

### 2.2 Fluent Bit

Fluent Bit merupakan perangkat lunak yang digunakan untuk mengumpulkan, memproses, dan meneruskan data log dari berbagai sumber. Dalam praktikum ini, Fluent Bit berperan sebagai log collector yang menerima log dari beberapa container aplikasi, kemudian meneruskannya ke PostgreSQL sebagai tempat penyimpanan log terpusat.

### 2.3 PostgreSQL sebagai Log Storage

PostgreSQL merupakan sistem manajemen basis data relasional yang digunakan untuk menyimpan dan mengelola data. Pada praktikum ini, PostgreSQL digunakan sebagai media penyimpanan log terpusat. Data log disimpan dalam tabel `logs.fluentbit` dengan struktur yang terdiri dari `tag`, `time`, dan `data` bertipe JSONB. Penggunaan JSONB memungkinkan informasi log disimpan dalam format terstruktur dan dapat dianalisis menggunakan query SQL.

### 2.4 Structured Logging

Structured logging merupakan metode pencatatan log dalam format terstruktur, seperti JSON, sehingga informasi di dalamnya dapat dibaca, dikelompokkan, dan dianalisis dengan lebih mudah. Informasi log dapat mencakup timestamp, nama layanan, tingkat keparahan (severity), jenis kejadian (event), dan `request_id`. Format ini membantu proses pencarian dan analisis aktivitas aplikasi.

### 2.5 Request ID dan Korelasi Log

Request ID merupakan identitas unik yang diberikan pada suatu permintaan untuk membantu melacak aktivitas yang terjadi selama proses berlangsung. Dengan menggunakan `request_id`, log dari beberapa layanan dapat dihubungkan sehingga urutan kejadian dalam suatu proses dapat ditelusuri. Pada praktikum ini, request ID digunakan untuk menganalisis alur pendaftaran dan mengidentifikasi penyebab kegagalan verifikasi.

### 2.6 Retry, Buffering, dan Recovery

Retry merupakan mekanisme percobaan ulang ketika proses pengiriman log mengalami kegagalan. Buffering digunakan untuk menampung sementara data log sebelum diteruskan ke tujuan penyimpanan. Sementara itu, recovery merupakan proses pemulihan setelah layanan yang mengalami gangguan kembali tersedia. Ketiga konsep ini diuji dengan menghentikan sementara PostgreSQL log database, mengamati respons Fluent Bit, kemudian mengaktifkan kembali database untuk memeriksa proses pengiriman log.

### 2.7 Analisis Log Menggunakan SQL

Structured log yang tersimpan di PostgreSQL dapat dianalisis menggunakan Structured Query Language (SQL). Query SQL dapat digunakan untuk menghitung jumlah log, mengelompokkan log berdasarkan layanan dan tingkat keparahan, serta menyusun kronologi kejadian berdasarkan request ID. Analisis ini membantu proses pemantauan dan investigasi insiden pada aplikasi.

# 3. Alat dan Lingkungan Praktikum

Praktikum Bab 6 dilaksanakan menggunakan beberapa perangkat keras dan perangkat lunak untuk menjalankan layanan aplikasi, mengumpulkan log, serta melakukan analisis data log.

### 3.1 Perangkat Keras

- Laptop atau komputer dengan sistem operasi Windows.
- Memori RAM minimal 2 GB.
- Ruang penyimpanan yang tersedia minimal 3 GB.

### 3.2 Perangkat Lunak dan Teknologi

| No. | Perangkat Lunak/Teknologi | Fungsi |
|---|---|---|
| 1 | Windows | Sistem operasi utama |
| 2 | Ubuntu WSL | Lingkungan Linux untuk menjalankan perintah praktikum |
| 3 | Docker Desktop | Menjalankan container aplikasi dan layanan pendukung |
| 4 | Docker Compose | Mengelola beberapa container dalam satu lingkungan |
| 5 | Fluent Bit 4.2.8 | Mengumpulkan dan meneruskan log aplikasi |
| 6 | PostgreSQL 16 | Menyimpan data transaksi dan log terpusat |
| 7 | Bash | Menjalankan skrip praktikum |
| 8 | curl | Melakukan pengujian endpoint layanan |
| 9 | Python 3 | Mendukung aplikasi dan pengujian |
| 10 | GitHub | Menyimpan kode proyek dan laporan praktikum |

### 3.3 Lingkungan Praktikum

Praktikum menggunakan proyek **PMB DevSecOps Laboratory** yang dijalankan melalui Docker Compose. Lingkungan tersebut terdiri dari enam container, yaitu `registration-api`, `auth-service`, `verification-worker`, `app-db`, `fluent-bit`, dan `log-db`.

Seluruh layanan dijalankan dalam lingkungan container agar proses pengumpulan, penyimpanan, dan analisis log dapat dilakukan secara terintegrasi.

