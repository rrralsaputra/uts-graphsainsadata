# RetailRocket Graph Recommendation System
### Ujian Tengah Semester - Graph Sains Data

Dokumen ini berisi implementasi **Sistem Rekomendasi Berbasis Graf** menggunakan **Neo4j** dan **Python** berbasis *dataset* e-commerce RetailRocket (`events.csv`). Proyek ini mencakup alur *end-to-end* mulai dari *data wrangling*, *exploratory data analysis* (EDA) pada struktur graf, pemodelan ruang vektor (*Vector Space Modelling* / VSM), perhitungan *similarity*, pemodelan rekomendasi *K-Nearest Neighbors* (KNN), hingga evaluasi kinerja model.

---

## 📌 Struktur Repositori

```text
├── data/
│   ├── train_p1.csv.gz       # Data interaksi Periode 1 (Training Set)
│   └── test_p2.csv.gz        # Data interaksi Periode 2 (Testing Set)
├── UTS_GrafSainsData_091.ipynb # Jupyter Notebook utama berisi seluruh eksperimen & kode
├── requirements.txt          # Daftar pustaka Python yang digunakan
├── .gitignore                # Berkas pengecualian untuk Git
└── README.md                 # Dokumentasi utama proyek
```

> **Catatan Dataset:** File mentah `events.csv` dan berkas pendukung berukuran besar diabaikan dari pelacakan Git demi efisiensi repositori. File `train_p1.csv` dan `test_p2.csv` diunggah menggunakan pengelolaan versi data terkompresi / Git LFS.

---

## ⚙️ Metodologi & Alur Pengerjaan

### 1. Preparasi & Ingesti Data (Neo4j)
* **Koneksi Database:** Menghubungkan Python ke *instance* lokal Neo4j melalui protokol `bolt://127.0.0.1:7687`.
* **Graph Ingestion:** Mengimpor interaksi *User-Item* dari dataset RetailRocket ke dalam Neo4j menggunakan skema graf berarah berbasis *batch processing*.
* **Manajemen Database:** Menjalankan pembersihan total (*reset*) database menggunakan perintah Cypher:
  ```cypher
  MATCH (n) DETACH DELETE n;
  ```

---

### 2. Data Wrangling & Segmentasi Waktu (Soal 2.1)
Data dipisahkan berdasarkan rentang waktu (*timestamp*) menjadi dua rentang periode independen:
* **Periode 1 (`P1` - Training Set):** `2015-05-11` s.d. `2015-08-11`
* **Periode 2 (`P2` - Testing Set):** `2015-08-12` s.d. `2015-09-05`

Hasil pemisahan diekspor menjadi file `.csv` terpisah untuk menjaga konsistensi evaluasi model.

---

### 3. Exploratory Data Analysis (EDA) Subgraph
Analisis komposisi *subgraph* dilakukan menggunakan Python API ke Neo4j untuk menghitung struktur simpul (*Node*) dan sisi (*Relationship*):

```cypher
-- Analisis Komposisi Label Node
MATCH (n)
UNWIND labels(n) AS label
RETURN label AS Jenis_Simpul, count(*) AS Jumlah
ORDER BY label;

-- Analisis Komposisi Relasi
MATCH ()-[r]->()
RETURN type(r) AS Jenis_Relasi, count(*) AS Jumlah
ORDER BY Jenis_Relasi;
```

---

### 4. Vector Space Modelling (VSM) & Proyeksi Graf
* **Representasi VSM:** Membangun matriks interaksi *User-Item* berbasis `scipy.sparse.csr_matrix` untuk efisiensi memori.
* **Proyeksi Bipartit:** Memvisualisasikan dan menghitung karakteristik graf bipartit *User-Item* menggunakan pustaka `NetworkX`.
* **Analisis Efisiensi Proyeksi:** Penggunaan representasi *Sparse Matrix* terbukti jauh lebih efisien secara memori dan waktu komputasi dibandingkan *Dense Matrix*, mengingat matriks interaksi pengguna *e-commerce* memiliki tingkat kelangkaan data (*sparsity*) yang sangat tinggi.

---

### 5. Perhitungan Similarity & Update Graf
* **Metode Similarity:** Menghitung *Cosine Similarity* antar-vektor *User* menggunakan representasi matriks VSM.
* **Perubahan Graf Cypher:** Memasukkan nilai *similarity* tertinggi ke dalam Neo4j sebagai bobot relasi baru (`SIMILAR_TO`) antar-simpul pengguna:
  ```cypher
  MATCH (u1:User {id: $user1_id}), (u2:User {id: $user2_id})
  MERGE (u1)-[r:SIMILAR_TO]-(u2)
  SET r.score = $similarity_score;
  ```

---

### 6. Sistem Rekomendasi (KNN) & Analytics (Soal 2.2)
* **Aturan Pemilihan User:** Memilih *User* target secara acak dari data pengujian menggunakan *random seed* agar dapat direproduksi (*reproducible*).
* **Algoritma KNN:** Membangun model *K-Nearest Neighbors* ($k=5$) berdasarkan *User Similarity* untuk memprediksi dan merekomendasikan **3 item teratas** yang belum pernah diinteraksi oleh *User* target.

---

## 📊 Hasil Evaluasi Model

Evaluasi dilakukan dengan membandingkan item rekomendasi hasil prediksi ($k=5$) terhadap interaksi aktual pengguna pada dataset pengujian (Periode `P2`):

| Metrik Evaluasi | Deskripsi |
| :--- | :---: | :--- |
| **Hit@05**  | Mengukur apakah minimal ada 1 item relevan yang berhasil direkomendasikan dalam Top-5. |
| **NDCG@05**  | Mengukur kualitas urutan/peringkat item rekomendasi (*Normalized Discounted Cumulative Gain*). |

---

## 🚀 Cara Menjalankan Proyek

### 1. Prasyarat Sistem
* Python 3.10 atau versi lebih baru.
* Neo4j Desktop / Community Edition (berjalan di port default `7687`).

### 2. Instalasi Dependensi
Klon repositori ini dan pasang pustaka Python yang dibutuhkan:
```bash
git clone https://github.com/rrralsaputra/uts-graphsainsadata.git
cd uts-graphsainsadata
pip install -r requirements.txt
```

### 3. Eksekusi Notebook
1. Pastikan layanan database Neo4j lokal Anda aktif.
2. Buka dan jalankan seluruh sel pada berkas `UTS_GrafSainsData_091.ipynb` secara berurutan.

---

## 👤 Penulis
* **Nama:** Muhammad Geralldo Agatha Saputra
* **NIM:** *24031554091*
* **Program Studi / Institusi:** S1 Sains Data / Universitas Negeri Surabaya (UNESA)
