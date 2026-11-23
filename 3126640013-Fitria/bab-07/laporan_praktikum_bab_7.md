# BAB 7 — MONITORING RESOURCE DENGAN PROMETHEUS, CADVISOR, NODE EXPORTER, DAN GRAFANA

**Disusun untuk Memenuhi Tugas Praktikum Mata Kuliah DevOps**

- **Nama:** Fitria Sari Ayuningtyas

- **NIM:** 3126640013

- **Kelas:** STr LJ Teknik Informatika B

- **Judul Praktikum:** Monitoring Resource dengan Prometheus, cAdvisor, Node Exporter, dan Grafana
  
## 1. Tujuan Praktikum

Praktikum ini bertujuan untuk:
1. Membedakan monitoring metrics dan logging events.
2. Menjalankan Prometheus sebagai scraper dan time-series database.
3. Mengumpulkan metrik resource host menggunakan Node Exporter dan metrik container menggunakan cAdvisor.
4. Membuat dashboard Grafana untuk memantau penggunaan CPU, memori, disk, jaringan, dan container.
5. Mengintegrasikan data log PostgreSQL dari praktikum Bab 6 ke Grafana.

## 2. Dasar Teori Singkat

Monitoring merupakan proses pengamatan kondisi sistem melalui pengumpulan metrik secara berkala, seperti penggunaan CPU, memori, disk, dan jaringan. Monitoring membantu mengetahui kondisi dan perubahan performa sistem, sedangkan logging digunakan untuk mencatat kejadian atau aktivitas secara lebih terperinci.

Prometheus merupakan sistem monitoring yang menggunakan metode *pull*, yaitu mengambil metrik dari endpoint target secara berkala dan menyimpannya dalam bentuk time-series database. Node Exporter digunakan untuk mengumpulkan metrik resource host, sedangkan cAdvisor digunakan untuk mengumpulkan metrik penggunaan resource container Docker.

Grafana merupakan platform visualisasi yang digunakan untuk menampilkan data dari berbagai sumber, termasuk Prometheus dan PostgreSQL, dalam bentuk dashboard. Melalui PromQL, pengguna dapat melakukan query dan analisis terhadap metrik yang dikumpulkan Prometheus. Integrasi monitoring dan logging memungkinkan kondisi sistem diamati sekaligus membantu proses analisis ketika terjadi gangguan.

## 3. Alat dan Lingkungan

| **Komponen** | **Hasil Identifikasi** |
|---|---|
| Operating System | Ubuntu pada WSL 2 |
| Pengguna eksekusi | `asus` |
| Direktori kerja | `~/docker-lab/bab-7` |
| Docker Engine | 29.7.2 |
| Docker Compose | 5.5.0 |
| Prometheus | v3.14.0 |
| Grafana | 13.1.4 |
| Node Exporter | v1.12.1 |
| cAdvisor | v0.60.5 |
| PostgreSQL | Database log Bab 6 |
| Web Browser | Digunakan untuk mengakses Prometheus dan Grafana |

