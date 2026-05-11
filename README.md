# 🛍️ ETL Pipeline — Fashion Products Data

> **Microsoft Elevate 2025 — Submission Project**
> End-to-end ETL (Extract, Transform, Load) pipeline untuk scraping data produk fashion dari website, melakukan pembersihan data, dan menyimpannya ke multiple repositories (CSV, Google Sheets, PostgreSQL).

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Coverage](https://img.shields.io/badge/coverage-92%25-brightgreen.svg)]()
[![Tests](https://img.shields.io/badge/tests-passing-success.svg)]()
[![License](https://img.shields.io/badge/license-MIT-yellow.svg)]()

---

## 📖 Tentang Project

Project ini merupakan implementasi **ETL Pipeline** yang dirancang untuk mengotomasi proses pengumpulan, pembersihan, dan penyimpanan data produk fashion. Pipeline ini melakukan scraping ~1000 produk dari [Fashion Studio Dicoding](https://fashion-studio.dicoding.dev), melakukan standarisasi data (termasuk konversi mata uang USD → IDR), kemudian menyimpan hasilnya ke tiga repositori berbeda.

### ✨ Fitur Utama

- 🕸️ **Web Scraping** — Mengumpulkan data dari 50 halaman website fashion.
- 🧹 **Data Cleaning & Transformation** — Standarisasi format, konversi tipe data, dan penghapusan data invalid.
- 💾 **Multi-Repository Loading** — Menyimpan ke CSV, Google Sheets, dan PostgreSQL secara bersamaan.
- ✅ **Comprehensive Testing** — Coverage gabungan mencapai **92%**.
- 🛡️ **Robust Error Handling** — Penanganan error pada setiap tahap pipeline.

---

## 🏗️ Arsitektur ETL

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   EXTRACT    │ ──▶ │  TRANSFORM   │ ──▶ │     LOAD     │
│              │     │              │     │              │
│ Web Scraping │     │ Cleaning &   │     │  CSV         │
│ (50 pages)   │     │ Standardize  │     │  Google Sheet│
│              │     │              │     │  PostgreSQL  │
└──────────────┘     └──────────────┘     └──────────────┘
```

### Diagram Alur Lengkap

```
                    ┌─────────────────────────────┐
                    │         EXTRACT             │
                    └─────────────┬───────────────┘
                                  │
                       ┌──────────┴──────────┐
                       │   Website Response   │
                       └──────────┬──────────┘
                                  │
                  ┌───────────────┴───────────────┐
                  │  Apakah HTML rusak / error?   │
                  ├───────────────┬───────────────┤
                Yes│               │No
                   ▼               ▼
        extract_product() → None  extract_product() → dict
                   │               │
                   └───────┬───────┘
                           ▼
                  ┌─────────────────┐
                  │ DataFrame terisi?│
                  └────┬────────────┘
                Yes │       │ No → stop
                    ▼
              ┌──────────────┐
              │  TRANSFORM   │
              └──────┬───────┘
                     ▼
            ┌──────────────────┐
            │      LOAD        │
            │  ├─ save_csv()   │
            │  ├─ save_gsheet()│
            │  └─ save_postgres│
            └──────────────────┘
```

---

## 🗂️ Struktur Repository

```
microsoft-elevate-2025-etl-fashion-pipeline/
│
├── utils/                      # Modul-modul ETL
│   ├── extract.py              # Web scraping logic
│   ├── transform.py            # Data cleaning & transformation
│   └── load.py                 # Multi-target data loading
│
├── tests/                      # Unit tests
│   ├── test_extract.py         # Test untuk modul extract
│   ├── test_transform.py       # Test untuk modul transform
│   └── test_load.py            # Test untuk modul load
│
├── main.py                     # Entry point pipeline ETL
├── products.csv                # Output hasil ETL (sample)
├── requirements.txt            # Python dependencies
├── .gitignore
└── README.md
```

---

## ⚙️ Instalasi

### 1. Prasyarat

- Python 3.8+
- pip (package manager)
- Akun Google Cloud (untuk Google Sheets API)
- PostgreSQL (opsional, jika ingin menyimpan ke database)

### 2. Clone Repository

```bash
git clone https://github.com/ninditya/microsoft-elevate-2025-etl-fashion-pipeline.git
cd microsoft-elevate-2025-etl-fashion-pipeline
```

### 3. Buat Virtual Environment (Recommended)

```bash
python -m venv venv

# Aktifkan environment
# Linux / Mac:
source venv/bin/activate

# Windows:
venv\Scripts\activate
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

### 5. Setup Google Sheets API

1. Buka [Google Cloud Console](https://console.cloud.google.com/).
2. Buat project baru dan aktifkan **Google Sheets API** & **Google Drive API**.
3. Buat **Service Account**, lalu download credentials JSON.
4. Rename file menjadi `google-sheets-api.json` dan simpan di root folder.
5. Bagikan akses **Editor** pada Google Sheets target ke email service account.

---

## 🚀 Cara Menjalankan

### Menjalankan Pipeline ETL Lengkap

```bash
python3 main.py
```

Pipeline akan mengeksekusi tahapan secara berurutan:
1. **Extract** — scraping data dari 50 halaman website.
2. **Transform** — membersihkan dan standarisasi ~1000 baris data.
3. **Load** — menyimpan hasil ke CSV, Google Sheets, dan PostgreSQL.

### Menjalankan Unit Test

```bash
python3 -m pytest tests
```

### Menjalankan Test dengan Coverage Report

```bash
python3 -m pytest --cov=utils tests/ -v
```

---

## 🔄 Detail Pipeline

### 1️⃣ EXTRACT — Web Scraping

- **Sumber data:** [https://fashion-studio.dicoding.dev](https://fashion-studio.dicoding.dev)
- **Coverage halaman:** Halaman 1 sampai 50 (~1000 produk)
- **Kolom yang di-extract:**
  - `Title` — Nama produk
  - `Price` — Harga (USD)
  - `Rating` — Rating produk
  - `Colors` — Jumlah warna tersedia
  - `Size` — Ukuran (S, M, L, XL, dll)
  - `Gender` — Target gender
  - `Timestamp` — Waktu data di-extract
- **Error handling:** try-except untuk HTML rusak & request gagal.
- **Test coverage:** 97%

### 2️⃣ TRANSFORM — Data Cleaning

| Kolom    | Transformasi                                                  |
| -------- | ------------------------------------------------------------- |
| `Price`  | Konversi USD → IDR (kurs Rp16.000), tipe `int`                |
| `Rating` | Diubah ke `float`, dibersihkan dari karakter tambahan         |
| `Colors` | Ambil angka saja (mis. "3 colors" → `3`)                      |
| `Size`   | Bersihkan prefix (mis. "Size: M" → "M")                       |
| `Gender` | Bersihkan prefix (mis. "Gender: Men" → "Men")                 |

**Penghapusan data invalid:**
- ❌ Null values
- ❌ Duplikat
- ❌ Produk dengan title `"Unknown Product"`
- ❌ Data dengan tipe tidak valid

**Test coverage:** 90%

### 3️⃣ LOAD — Multi-Repository Storage

| Target          | Status      | Output                              |
| --------------- | ----------- | ----------------------------------- |
| **CSV**         | ✅ Wajib    | `products.csv`                      |
| **Google Sheet**| ✅ Wajib    | Service account dengan akses editor |
| **PostgreSQL**  | 🔧 Opsional | Tabel `products`                    |

**Error handling per fungsi:**
- DataFrame kosong → print pesan, skip save
- DataFrame berisi → simpan ke target
- Exception → ditangkap & dicetak

**Test coverage:** 89%

---

## 🧪 Testing & Coverage

| Modul       | Coverage |
| ----------- | -------- |
| `extract`   | 97%      |
| `transform` | 90%      |
| `load`      | 89%      |
| **Total**   | **92%**  |

Skenario testing mencakup:
- ✅ Happy path (data valid)
- ✅ Edge cases (data kosong, partial data)
- ✅ Error scenarios (HTML rusak, network error, DB exception)

---

## 📊 Hasil Output

### Sample Data (`products.csv`)

| Title          | Price       | Rating | Colors | Size | Gender | Timestamp           |
| -------------- | ----------- | ------ | ------ | ---- | ------ | ------------------- |
| T-Shirt Casual | 480000      | 4.5    | 3      | M    | Men    | 2025-01-15 10:30:00 |
| Hoodie Premium | 720000      | 4.8    | 5      | L    | Unisex | 2025-01-15 10:30:01 |

### Google Sheets Output

🔗 [Lihat hasil di Google Sheets](https://docs.google.com/spreadsheets/d/1nKOSdLbGnllou1yGTFGQAtgbFPhXWZ2cVsDv13R668U)

---

## 🛠️ Tech Stack

- **Bahasa:** Python 3.8+
- **Web Scraping:** `requests`, `BeautifulSoup4`
- **Data Processing:** `pandas`, `numpy`
- **Database:** `psycopg2` / `SQLAlchemy` (PostgreSQL)
- **Google Sheets:** `gspread`, `oauth2client`
- **Testing:** `pytest`, `pytest-cov`, `unittest.mock`

---

## 📝 Catatan Penting

- 🔐 File `google-sheets-api.json` **TIDAK** boleh di-commit ke repository (sudah di-handle di `.gitignore`).
- 💱 Kurs konversi USD ke IDR menggunakan nilai tetap **Rp16.000** (dapat disesuaikan di `transform.py`).
- 🌐 Pastikan koneksi internet stabil saat menjalankan pipeline (proses scraping 50 halaman).
- 🐘 PostgreSQL bersifat opsional — pipeline tetap berjalan meski database tidak tersedia.

---

## 🎯 Prinsip Desain

Project ini menerapkan beberapa prinsip software engineering:

- ✨ **Modularisasi** — Setiap tahap ETL terpisah dalam modul independen.
- 🔌 **Separation of Concerns** — Logic extract, transform, dan load tidak saling tergantung.
- 🛡️ **Defensive Programming** — Error handling pada setiap titik kritis.
- 🧪 **Test-Driven** — Unit test untuk setiap modul dengan coverage tinggi.

---

## 📄 Lisensi

Project ini dibuat sebagai bagian dari submission **Microsoft Elevate 2025**.

---

## 👤 Author

**ninditya**
- GitHub: [@ninditya](https://github.com/ninditya)

---

<p align="center">
  Made with ☕ for Microsoft Elevate 2025
</p>
