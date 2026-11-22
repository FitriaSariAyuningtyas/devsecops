
# LAPORAN PRAKTIKUM BAB 6
## CENTRALIZED LOGGING DENGAN FLUENT BIT DAN POSTGRESQL

**Mata Kuliah:** DevOps

**Topik Praktikum:** Centralized Logging dengan Fluent Bit dan PostgreSQL

**Nama:** Fitria Sari Ayuningtyas

**NIM:** 3126640013

**Kelas:** STr LJ Teknik Informatika B


---

# 1. Pendahuluan

Praktikum Bab 6 membahas penerapan centralized logging menggunakan Fluent Bit dan PostgreSQL. Praktikum ini bertujuan untuk memahami proses pengumpulan log dari beberapa container Docker, pemrosesan log menggunakan Fluent Bit, serta penyimpanan data log ke dalam database PostgreSQL dengan format JSONB.

Melalui praktikum ini, dilakukan simulasi pengiriman log dari beberapa container, penerapan filter untuk penyamaran data, serta pengujian penyimpanan dan analisis log menggunakan query SQL.

# 2. Tujuan Praktikum

1. Memahami konsep centralized logging pada lingkungan container.
2. Mengirimkan log container Docker ke Fluent Bit menggunakan logging driver Fluentd.
3. Menyimpan data log ke PostgreSQL menggunakan tipe data JSONB.
4. Melakukan pemrosesan dan penyamaran data log menggunakan filter Lua.
5. Melakukan analisis serta pengujian terhadap data log yang tersimpan.


# 3. Struktur Direktori Project

Project praktikum Bab 6 disusun dalam beberapa direktori untuk memisahkan konfigurasi layanan, skrip inisialisasi database, query analisis, serta file pendukung pengujian. Struktur direktori yang digunakan adalah sebagai berikut.

```text
bab-6/
├── .env.example
├── .gitignore
├── compose.yaml
├── compose.yaml.bak
├── evidence/
│   ├── generator-logs-backup.csv
│   ├── log-export.csv
│   └── log-export.sha256
├── fluent-bit/
│   ├── fluent-bit.conf
│   ├── fluent-bit.conf.bak
│   └── redact.lua
├── init/
│   └── 01-init-logging.sh
├── queries/
│   └── analysis.sql
└── scripts/
    └── test-buffering.sh
```

Keterangan direktori dan file:

- `.env.example`: Berisi contoh variabel lingkungan untuk konfigurasi PostgreSQL dan Fluent Bit.
- `.gitignore`: Menentukan file dan direktori yang tidak disertakan dalam pelacakan Git.
- `compose.yaml`: Mengatur layanan Docker yang digunakan dalam praktikum.
- `compose.yaml.bak`: File cadangan konfigurasi Docker Compose.
- `evidence/`: Menyimpan hasil ekspor log, backup log generator, dan checksum.
- `fluent-bit/`: Menyimpan konfigurasi Fluent Bit, skrip penyamaran data, dan file cadangan konfigurasi.
- `init/`: Menyimpan skrip inisialisasi database PostgreSQL.
- `queries/`: Menyimpan query SQL untuk analisis data log.
- `scripts/`: Menyimpan skrip otomatisasi pengujian buffering.


# 4. Implementasi dan Konfigurasi

Bagian ini menjelaskan implementasi dan konfigurasi komponen yang digunakan dalam pembangunan sistem centralized logging. Konfigurasi meliputi pengaturan variabel lingkungan, pengelolaan file Git, Docker Compose, Fluent Bit, serta inisialisasi database PostgreSQL.

### 4.1 Konfigurasi Environment (`.env.example`)

File `.env.example` digunakan sebagai contoh konfigurasi variabel lingkungan yang dibutuhkan oleh layanan PostgreSQL dan Fluent Bit.

Berikut isi konfigurasi `.env.example`:

```env
POSTGRES_DB=labdb
POSTGRES_USER=labadmin
POSTGRES_PASSWORD=labadminpass123

FLUENTBIT_DB_USER=fluentbit_user
FLUENTBIT_DB_PASSWORD=fluentbitpass123
```

Variabel `POSTGRES_DB`, `POSTGRES_USER`, dan `POSTGRES_PASSWORD` digunakan untuk mengatur database PostgreSQL. Sementara itu, `FLUENTBIT_DB_USER` dan `FLUENTBIT_DB_PASSWORD` digunakan sebagai kredensial koneksi Fluent Bit ke PostgreSQL.

**Dokumentasi konfigurasi:**

![Konfigurasi Environment](evidence/bab6-09-konfigurasi-env-example.png)

*Gambar 4.1 Tampilan Konfigurasi Variabel Environment pada File `.env.example`.*




### 4.3 Konfigurasi Docker Compose (`compose.yaml`)

File `compose.yaml` digunakan untuk mengatur dan menjalankan seluruh layanan yang dibutuhkan dalam praktikum Centralized Logging. Layanan yang digunakan terdiri dari PostgreSQL sebagai penyimpanan log, Fluent Bit sebagai pengumpul dan pengirim log, serta tiga container generator untuk menghasilkan log secara otomatis.

**a. Konfigurasi layanan PostgreSQL**

```yaml
services:
  postgres-db:
    image: postgres:16-alpine
    container_name: bab6-postgres-db
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "127.0.0.1:5433:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U \"$${POSTGRES_USER}\" -d \"$${POSTGRES_DB}\""]
      interval: 5s
      timeout: 3s
      retries: 10
```

PostgreSQL menggunakan image `postgres:16-alpine` dan berfungsi sebagai tempat penyimpanan log. Data disimpan menggunakan Docker volume agar tetap tersedia meskipun container dihentikan atau dibuat ulang. Health check digunakan untuk memastikan PostgreSQL siap menerima koneksi.

**b. Konfigurasi layanan Fluent Bit**

```yaml
  fluent-bit:
    image: cr.fluentbit.io/fluent/fluent-bit:3.2
    container_name: bab6-fluent-bit
    restart: unless-stopped
    env_file:
      - .env
    depends_on:
      postgres-db:
        condition: service_healthy
    ports:
      - "127.0.0.1:2020:2020"
      - "127.0.0.1:24224:24224"
    volumes:
      - ./fluent-bit/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf:ro
      - fluentbit_storage:/var/log/flb-storage
      - ./fluent-bit/redact.lua:/fluent-bit/etc/redact.lua:ro
    command: ["--config", "/fluent-bit/etc/fluent-bit.conf"]
```

Fluent Bit bertugas menerima log dari container lain, melakukan pemrosesan sesuai konfigurasi, kemudian mengirimkannya ke PostgreSQL. Layanan ini dijalankan setelah PostgreSQL dinyatakan sehat. Volume `fluentbit_storage` digunakan untuk mendukung penyimpanan buffer berbasis filesystem.

**c. Konfigurasi container generator**

```yaml
  generator-auth:
    image: alpine:3.20
    command:
      - /bin/sh
      - -c
      - |
        i=1
        while [ "$$i" -le 20 ]; do
          printf '{"service":"auth","severity":"INFO","event":"login_test","sequence":%s}\n' "$$i"
          i=$$((i + 1))
          sleep 1
        done
    logging:
      driver: fluentd
      options:
        fluentd-address: "127.0.0.1:24224"
        fluentd-async: "true"
        tag: "docker.{{.Name}}"
    depends_on:
      - fluent-bit

  generator-api:
    image: alpine:3.20
    command:
      - /bin/sh
      - -c
      - |
        i=1
        while [ "$$i" -le 20 ]; do
          printf '{"service":"api","severity":"INFO","event":"request_test","sequence":%s,"token":"CANARY_BAB6_SECRET_2026"}\n' "$$i"
          i=$$((i + 1))
          sleep 1
        done
    logging:
      driver: fluentd
      options:
        fluentd-address: "127.0.0.1:24224"
        fluentd-async: "true"
        tag: "docker.{{.Name}}"
    depends_on:
      - fluent-bit

  generator-worker:
    image: alpine:3.20
    command:
      - /bin/sh
      - -c
      - |
        i=1
        while [ "$$i" -le 20 ]; do
          printf '{"service":"worker","severity":"INFO","event":"job_test","sequence":%s}\n' "$$i"
          i=$((i + 1))
          sleep 1
        done
    logging:
      driver: fluentd
      options:
        fluentd-address: "127.0.0.1:24224"
        fluentd-async: "true"
        tag: "docker.{{.Name}}"
    depends_on:
      - fluent-bit

volumes:
  postgres_data:
  fluentbit_storage:
```

Ketiga container generator menggunakan image Alpine dan menghasilkan log dalam format JSON. Log dikirim menggunakan Docker logging driver `fluentd` menuju Fluent Bit melalui port `24224`. Setiap generator menghasilkan 20 event dengan interval satu detik. Pada konfigurasi ini, generator menghasilkan event dengan severity `INFO`.

Secara keseluruhan, Docker Compose menghubungkan container generator, Fluent Bit, dan PostgreSQL sehingga proses pengumpulan, pemrosesan, serta penyimpanan log dapat dilakukan secara terpusat.

