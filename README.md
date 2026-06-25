# Sistem Case-Based Reasoning (CBR) Putusan Perkara Pembunuhan (PN Manado)

Proyek ini merupakan implementasi sistem **Case-Based Reasoning (CBR)** berbasis Python untuk mendukung analisis dan prediksi putusan pengadilan pada perkara pidana pembunuhan (Pasal 338 dan Pasal 340 KUHP).

Proyek ini disusun untuk memenuhi **Tugas Proyek Mata Kuliah Penalaran Komputer**, Program Studi Informatika, Fakultas Teknik, Universitas Muhammadiyah Malang (UMM), Semester Genap Tahun Akademik 2025/2026.

---

## Informasi Kelompok

| Nama Anggota 1      | NIM Anggota 1   | Nama Anggota 2       | NIM Anggota 2   | Perkara                     | Nama<br />Pengadilan          | Jumlah<br />Putusan |
| ------------------- | --------------- | -------------------- | --------------- | --------------------------- | ----------------------------- | ------------------- |
| Muhammad Fadhil Y.Z | 202310370311293 | Thariq Fadhlurrahman | 202310370311299 | Pidana Umum<br />Pembunuhan | Pengadilan<br />Negeri Manado | 40                  |

---

## 1. Struktur Repositori

Penyusunan kode dan penyimpanan berkas dalam repositori ini diatur sebagai berikut:

```text
├── data/
│   ├── downloaded_pdf/              # Berkas PDF putusan asli hasil unduhan manual
│   ├── raw/                         # Teks bersih hasil ekstraksi PDF (.txt)
│   ├── processed/                   # Dataset terstruktur dalam format CSV
│   └── eval/                        # Berkas pengujian kueri dan metrik evaluasi
│
├── notebooks/
│   ├── 01_case_base_building.ipynb  # Tahap 1: Konversi & Pembersihan Teks
│   ├── 02_case_representation.ipynb # Tahap 2: Ekstraksi Fitur & Metadata
│   ├── 03_retrieval.ipynb           # Tahap 3: Vektorisasi TF-IDF & Retrieval
│   ├── 04_predict.ipynb             # Tahap 4: Prediksi (Majority Vote)
│   └── 05_evaluation.ipynb          # Tahap 5: Evaluasi Model
│
├── results/
│   └── predictions.csv              # Hasil prediksi akhir
│
├── requirements.txt                 # Daftar dependensi Python
└── README.md                        # Dokumentasi utama proyek
```

---

## 2. Dataset dan Ranah Hukum

### Domain Hukum

Pidana Umum — Perkara Pembunuhan:

- **Pasal 338 KUHP** (Pembunuhan Biasa)
- **Pasal 340 KUHP** (Pembunuhan Berencana)

### Volume Data

- 40 dokumen putusan resmi Pengadilan Negeri Manado.

### Komposisi Dataset

| Kategori Kasus | Jumlah |
| -------------- | ------ |
| Pasal 340 KUHP | 19     |
| Pasal 338 KUHP | 18     |
| Pasal Lainnya  | 3      |
| **Total**      | **40** |

---

## 3. Instalasi dan Dependensi

Pastikan Python versi **3.8 atau lebih baru** telah terpasang pada sistem Anda.

### Clone Repository

```bash
git clone https://github.com/username/cbr-pembunuhan.git
cd cbr-pembunuhan
```

### Install Dependensi

```bash
pip install -r requirements.txt
```

### Dependensi Utama

- pypdf
- pandas
- numpy
- scikit-learn
- selenium
- beautifulsoup4
- webdriver-manager

---

## 4. Eksekusi Pipeline End-to-End

Jalankan notebook secara berurutan untuk mereplikasi seluruh siklus Case-Based Reasoning.

### 1. `01_case_base_building.ipynb`

Tahap ini:

- Membaca file PDF dari folder `data/downloaded_pdf/`
- Melakukan pembersihan teks:
  - lowercase
  - penghapusan header
  - penghapusan footer
  - penghapusan nomor halaman

- Menyimpan hasil ekstraksi ke folder `data/raw/`

### 2. `02_case_representation.ipynb`

Tahap ini:

- Menggunakan Regular Expression (Regex)
- Mengekstrak metadata penting:
  - Nomor perkara
  - Tanggal putusan
  - Nama pihak
  - Pasal yang digunakan
  - Ringkasan fakta perkara

- Menyimpan hasil ke:

```text
data/processed/cases.csv
```

### 3. `03_retrieval.ipynb`

Tahap ini:

- Melakukan pembagian data train-test (80:20)
- Membangun representasi TF-IDF
- Menghitung kemiripan menggunakan Cosine Similarity
- Menyediakan fungsi:

```python
retrieve()
```

untuk menemukan kasus yang paling mirip.

### 4. `04_predict.ipynb`

Tahap ini:

- Mengambil 5 kasus terdekat hasil retrieval
- Melakukan voting mayoritas (_Majority Vote_)
- Menghasilkan prediksi pasal yang paling mungkin

Fungsi utama:

```python
predict_outcome()
```

### 5. `05_evaluation.ipynb`

Tahap ini:

- Mengukur performa sistem menggunakan:
  - Accuracy
  - Precision
  - Recall
  - F1-Score

- Memanfaatkan modul:

```python
sklearn.metrics
```

---

## 5. Hasil Evaluasi Model

Berikut hasil evaluasi pada data uji.

| Metrik Evaluasi            | Nilai       |
| -------------------------- | ----------- |
| Accuracy                   | 0.60 (60%)  |
| Precision                  | 0.50 (50%)  |
| Recall                     | 0.50 (50%)  |
| F1-Score                   | 0.44 (44%)  |
| Retrieval Hit Rate (Top-5) | 1.00 (100%) |

---

## 6. Analisis Kegagalan Model

### Error Analysis

Model berhasil mencapai **Retrieval Hit Rate sebesar 100%**, yang menunjukkan bahwa kasus dengan rumpun hukum yang sama hampir selalu muncul dalam daftar lima kasus terdekat.

Namun demikian, **akurasi prediksi solusi hanya mencapai 60%**.

Penyebab utama kegagalan berasal dari strategi ekstraksi teks yang hanya menggunakan sebagian awal dokumen (`[:1200]` karakter pertama).

Dokumen putusan pengadilan memiliki format baku yang diawali oleh:

- kop surat
- identitas pengadilan
- teks administratif
- frasa formal yang berulang

Contohnya:

> "Mahkamah Agung Republik Indonesia ..."

Akibatnya, metode TF-IDF lebih banyak menangkap kesamaan kata-kata birokrasi daripada substansi kronologi tindak pidana.

Dampaknya:

- Kasus Pasal 338 dan Pasal 340 sering dianggap sangat mirip.
- Voting mayoritas menghasilkan prediksi yang keliru.
- Solusi akhir tidak selalu sesuai dengan kasus sebenarnya.

---

## 7. Rekomendasi Pengembangan

### 1. Perbaikan Segmentasi Teks

Memodifikasi proses ekstraksi agar melewati bagian administratif dan langsung mengambil isi perkara setelah frasa seperti:

> "Menimbang, bahwa berdasarkan surat dakwaan..."

Dengan pendekatan ini, representasi dokumen akan lebih berfokus pada kronologi dan fakta hukum.

### 2. Penggunaan Text Embedding Modern

Menggantikan pendekatan TF-IDF dengan model berbasis Transformer, seperti:

- IndoBERT
- IndoBERTweet
- Sentence-BERT Bahasa Indonesia

Keuntungan pendekatan embedding:

- Memahami konteks kalimat secara semantik.
- Membedakan pembunuhan berencana dan spontan dengan lebih baik.
- Lebih robust terhadap variasi kosakata.

---

## 8. Kesimpulan

Sistem CBR yang dibangun berhasil menunjukkan bahwa pendekatan retrieval berbasis TF-IDF mampu menemukan kasus-kasus yang relevan dengan tingkat keberhasilan tinggi.

Meskipun demikian, kualitas prediksi akhir masih dipengaruhi oleh kualitas representasi teks yang digunakan. Oleh karena itu, peningkatan pada tahap ekstraksi informasi dan penggunaan metode representasi berbasis embedding menjadi langkah penting untuk meningkatkan performa sistem pada penelitian selanjutnya.
