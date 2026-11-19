# Smart Grid Stability Assessment – Kelompok A2

Sistem **Hybrid AI** untuk memprediksi kestabilan jaringan listrik (*stable* / *unstable*):

| Komponen | Peran |
|---|---|
| **Genetic Algorithm (GA)** | Seleksi fitur |
| **Particle Swarm Optimization (PSO)** | Optimasi hyperparameter Neural Network |
| **Neural Network (MLP)** | Klasifikasi kestabilan |
| **Fuzzy Mamdani** | Mengubah probabilitas model menjadi skor & kategori risiko |
| **Streamlit** | Dashboard interaktif |

## Dataset
*Electrical Grid Stability Simulated Data* – UCI Machine Learning Repository (ID 471), 10.000 baris.

- **Target:** `stabf` (`stable` / `unstable`).
- **Fitur:** `tau1–tau4` (waktu reaksi), `p1–p4` (daya), `g1–g4` (elastisitas harga).
- Kolom **`stab` tidak dipakai sebagai fitur** karena menyebabkan *data leakage* (nilai `stab` menentukan `stabf`).
- Metadata UCI menyebut `p1` redundan terhadap `p2`, `p3`, `p4`. Baseline tetap memakai semua 12 fitur; GA kemudian memilih fitur secara otomatis.
- Pembagian data **70 : 15 : 15** (train / validation / test), `stratify`, `seed = 42`.
  GA dan PSO hanya memakai data **validation** untuk fitness; data **test** hanya untuk evaluasi akhir.

## Struktur Proyek
```
SmartGrid-Stability-A2/
├── app.py                 # Dashboard Streamlit (8 halaman)
├── eda.py                 # Exploratory Data Analysis
├── baseline_nn.py         # Model 1: Baseline NN (12 fitur)
├── genetic_algorithm.py   # GA: seleksi fitur -> artifacts/ga_selected_features.csv
├── ga_nn.py               # Model 2: GA + NN
├── pso_nn.py              # Model 3: PSO + NN
├── ga_pso_nn.py           # Model 4: GA + PSO + NN
├── fuzzy_system.py        # Sistem Fuzzy Mamdani (2 input, 9 rule)
├── integrated_system.py   # Integrasi 4 model + Fuzzy pada data test
├── requirements.txt       # Library Python
├── data/                  # Data_for_UCI_named.csv
└── artifacts/             # Model (.keras), scaler (.pkl), metrik, grafik, hasil
```

## Cara Menjalankan

**1. Instal library**:
```bash
pip install -r requirements.txt
```

**2. Jalankan dashboard** (artefak hasil training sudah tersedia di `artifacts/`):
```bash
streamlit run app.py
```

**3. (Opsional) Latih ulang dari awal** – jalankan dari folder proyek, **berurutan**:
```bash
python eda.py
python baseline_nn.py
python genetic_algorithm.py     # hasilnya dipakai ga_nn.py & ga_pso_nn.py
python ga_nn.py
python pso_nn.py
python ga_pso_nn.py
python integrated_system.py     # butuh keempat model di atas
```
Semua script memakai path relatif (`data/...`, `artifacts/...`), jadi jalankan dari folder yang berisi `app.py`.

## Hasil Sementara (data test = 1.500 baris, seed 42)

| Model | Jumlah Fitur | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|
| Baseline NN | 12 | 0,9567 | 0,9646 | 0,9676 | 0,9661 | 0,9943 |
| GA + NN | 8 | 0,9667 | 0,9680 | 0,9801 | 0,9740 | 0,9956 |
| PSO + NN | 12 | 0,9647 | 0,9650 | 0,9801 | 0,9725 | 0,9958 |
| GA + PSO + NN | 8 | 0,9753 | 0,9904 | 0,9707 | 0,9805 | 0,9974 |

Fitur terpilih GA: `tau1, tau2, tau3, tau4, g1, g2, g3, g4`.

> **Catatan:** hasil di atas baru dari satu seed. Selisih antarmodel kecil (sekitar 1–2%), sehingga klaim perbaikan perlu dibuktikan dengan beberapa seed. Fuzzy Logic berfungsi sebagai lapisan keputusan, bukan jaminan kenaikan akurasi.

## Fuzzy Mamdani
- **Input 1:** probabilitas rata-rata "tidak stabil" dari empat model.
- **Input 2:** tingkat kesepakatan antar model (proporsi pasangan model yang memprediksi kelas sama).
- **Output:** skor risiko 0–100 -> kategori *Risiko Rendah* (< 40), *Sedang* (40–70), *Tinggi* (≥ 70).

## Dashboard
Halaman: Project Overview, Dataset Explorer, Exploratory Data Analysis, Baseline Neural Network, Genetic Algorithm, PSO Optimization, Model Comparison, dan Fuzzy Decision (termasuk form prediksi data baru).

## Anggota & Pembagian Peran
| Nama | NIM | Peran |
|---|---|---|
| Aditya Raharja | 04231003 |
| Dirgantara Satria Putra | 04231024 | 
| Dwi Vera Anggreani | 04231026 |  |
| Muhammad Aji Kurniawan | 04231051 |  |
