# LAPORAN PRAKTIKUM BAB 1

## Fondasi Teoretis dan Kerangka Kerja DevSecOps

**Nama**: Fitria Sari Ayuningtyas  
**NIM**: 3126640013  
**Kelas**: D4 LJ Teknik Informatika  
**Tanggal pelaksanaan**: 15 September 2026

## 1. Tujuan Praktikum

Praktikum Bab 1 bertujuan untuk menetapkan baseline laboratorium DevSecOps dan melakukan verifikasi awal terhadap lingkungan yang digunakan sebelum melaksanakan eksperimen berikutnya.

## 2. Dasar Teori Singkat

DevSecOps merupakan pendekatan yang mengintegrasikan keamanan ke dalam seluruh proses pengembangan dan delivery perangkat lunak. Keamanan tidak hanya dilakukan pada tahap akhir, tetapi mulai diperhatikan sejak proses perencanaan, pengembangan, pengujian, deployment, hingga tahap operasi.

Salah satu prinsip dalam DevSecOps adalah *shift-left*, yaitu melakukan pemeriksaan keamanan sedini mungkin dalam proses pengembangan. Selain itu, terdapat *shift-right* yang berfokus pada pemantauan dan pemeriksaan keamanan setelah aplikasi dijalankan. Kedua pendekatan tersebut saling melengkapi untuk membantu menemukan dan menangani risiko keamanan.

Baseline diperlukan untuk mengetahui kondisi awal lingkungan praktikum, seperti versi perangkat yang digunakan, konfigurasi, struktur direktori, dan mekanisme keamanan yang tersedia. Dengan adanya baseline, hasil eksperimen dapat dicatat dan dibandingkan apabila terjadi perubahan pada lingkungan praktikum.

## 3. Alat dan Lingkungan

| Komponen | Hasil Identifikasi |
|---|---|
| Operating System | Ubuntu pada WSL 2 |
| Pengguna eksekusi | `asus` |
| Direktori kerja | `~/devsecops-lab` |
| Git | 2.53.0 |
| OpenSSL | 3.5.5 |
| cURL | 8.18.0 |
| Docker Engine | 29.7.2 |
| Docker Compose | 5.5.0 |
| Docker Security Options | `seccomp (builtin)`, `cgroupns` |
