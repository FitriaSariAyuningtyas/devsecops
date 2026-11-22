# LAPORAN PRAKTIKUM BAB 6
## CENTRALIZED LOGGING DENGAN FLUENT BIT DAN POSTGRESQL

**Disusun untuk Memenuhi Tugas Praktikum Mata Kuliah DevOps**


# 1. Tujuan Praktikum

Praktikum Bab 6 bertujuan untuk memahami penerapan centralized logging menggunakan Fluent Bit dan PostgreSQL sebagai media pengumpulan serta penyimpanan log dari beberapa layanan aplikasi. Melalui praktikum ini, mahasiswa diharapkan mampu:

1. Memahami perbedaan fungsi database transaksi dan database penyimpanan log terpusat.
2. Mengimplementasikan pengumpulan log dari beberapa layanan menggunakan Fluent Bit.
3. Memahami penggunaan structured logging untuk mencatat aktivitas aplikasi secara terstruktur.
4. Melakukan korelasi log antar layanan menggunakan `request_id`.
5. Menganalisis urutan kejadian atau timeline insiden berdasarkan data log menggunakan query SQL.
6. Menguji mekanisme retry, buffering, dan recovery ketika database log tidak tersedia.
7. Mengidentifikasi kemungkinan kehilangan log serta mengumpulkan bukti pengujian tanpa mengekspos data sensitif.
