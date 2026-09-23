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

## 4. Langkah Praktikum

### 4.1 Membuat Struktur Direktori

Pada tahap pertama dilakukan pembuatan struktur direktori kerja untuk laboratorium DevSecOps. Direktori utama yang digunakan adalah `devsecops-lab` dengan beberapa subdirektori, yaitu `app`, `policy`, `reports`, `sbom`, dan `keys`.

Perintah yang digunakan:

```bash
mkdir -p ~/devsecops-lab/{app,policy,reports,sbom,keys}
cd ~/devsecops-lab
```

Setelah perintah dijalankan, direktori kerja berhasil dibuat dan proses praktikum dilanjutkan dari direktori `~/devsecops-lab`.

### 4.2 Mencatat Versi Perangkat

Tahap berikutnya dilakukan pemeriksaan versi perangkat yang digunakan dalam praktikum. Pemeriksaan ini bertujuan untuk mencatat kondisi awal lingkungan sehingga dapat diketahui versi perangkat yang digunakan selama praktikum.

Perintah yang digunakan:

```bash
docker version
docker compose version
git --version
openssl version
curl --version
docker info --format '{{json .SecurityOptions}}'
```

Dari pemeriksaan tersebut diperoleh versi Docker, Docker Compose, Git, OpenSSL, dan cURL. Selain itu, dilakukan pemeriksaan terhadap Security Options pada Docker untuk mengetahui mekanisme keamanan yang tersedia pada lingkungan Docker.

### 4.3 Verifikasi Direktori dan Permission

Tahap terakhir dilakukan verifikasi terhadap direktori `reports`, `sbom`, dan `keys`. Pemeriksaan dilakukan untuk memastikan direktori tersebut telah tersedia serta mengetahui permission dan kepemilikannya.

Perintah yang digunakan:

```bash
ls -ld reports sbom keys
```

Hasil pemeriksaan menunjukkan bahwa ketiga direktori tersebut telah tersedia dengan permission `drwxr-xr-x` dan dimiliki oleh pengguna `asus` dengan group `docker`.

Pemeriksaan ini digunakan untuk mengetahui kondisi permission pada direktori. Konfigurasi web server tidak diperiksa pada praktikum ini, sehingga status direktori tersebut sebagai web root belum dapat diverifikasi.
