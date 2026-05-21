# Arkavidia 10.0 — Datavidia (Penyisihan)

## 🏆 Result

| Metric | Score |
|---|---|
| Public Score | 0.68725 |
| Private Score | 0.68358 |
| Overall Ranking | **5th Place** |

## 📋 Overview

Penurunan kualitas udara di wilayah metropolitan seperti DKI Jakarta telah menjadi isu lingkungan kritis yang berdampak langsung pada kesehatan masyarakat dan stabilitas ekonomi. Kompetisi Datavidia 10.0 menantang peserta untuk memetakan pola polusi menggunakan pendekatan berbasis data, memanfaatkan dataset historis ISPU dari tahun 2010 hingga 2025 serta variabel pendukung seperti cuaca, indeks vegetasi (NDVI), dan demografi.

**Task:** Klasifikasi kategori Indeks Standar Pencemar Udara (ISPU) harian di 5 stasiun DKI Jakarta (DKI1–DKI5) untuk periode September–November 2025. Tantangan utama adalah karakteristik data yang tidak seimbang (*imbalanced dataset*), di mana kategori ekstrem seperti "Berbahaya" jarang muncul namun krusial untuk dideteksi.

**Evaluation:** Macro F1-Score — rata-rata F1-Score per kelas tanpa mempedulikan proporsi data (*support*).

## 🧠 Approach

Pendekatan utama yang dipakai adalah **regresi → klasifikasi**: memprediksi nilai numerik ISPU (`max`), kemudian mengklasifikasikan ke kategori ("BAIK", "SEDANG", "TIDAK SEHAT") menggunakan threshold yang dioptimasi per stasiun.

### Eksperimen yang Dilakukan

1. **Climatology Method**
   Menggunakan median historis per kombinasi lokasi-bulan-hari sebagai *anchor* prediksi. Sederhana namun menjadi baseline yang berguna.

2. **Recursive Forecasting (Walk-Forward)**
   Model dilatih dengan fitur lag (`lag_1`, `lag_2`, `lag_3`), rolling statistics (`rolling_mean_7d`, `rolling_max_7d`), dan *climatological anchor*. Dua model dicoba: Random Forest Regressor dan LightGBM Regressor.

3. **Cross-Station Ensemble with Time Shift (Final Submission)**
   Metode utama yang digunakan untuk submisi akhir. Memanfaatkan data historis ~340 hari sebelumnya (data 2024) sebagai proksi untuk forecast 2025, dengan transfer lintas stasiun (*cross-station*):
   - DKI1 → source: DKI5 & DKI4 (mundur 342 hari)
   - DKI2 → source: DKI4 (mundur 340 hari)
   - DKI3 → source: DKI3 (mundur 340 hari)
   - DKI4 → source: DKI4 (mundur 340 hari)
   - DKI5 → source: DKI4 & DKI5 (mundur 340 hari)

   Model: **LightGBM Regressor** (`n_estimators=1000`, `learning_rate=0.05`, `num_leaves=31`)

### Feature Engineering

- **Time features:** year, month, day_of_year, day_of_week (dengan encoding siklus sin/cos)
- **Wind features:** dekomposisi vektor angin (komponen x, y) untuk ketinggian 10m dan 100m
- **Lag features:** `lag_1`, `lag_2`, `lag_3` pada nilai ISPU max
- **Rolling statistics:** `rolling_mean_7d`, `rolling_max_7d`
- **Climatological anchor:** median historis per stasiun-bulan-hari
- **NDVI lag features:** `ndvi_lag_1`, `ndvi_lag_2`, `ndvi_lag_3`

### Threshold Tuning

Threshold klasifikasi dioptimasi per stasiun menggunakan grid search pada validation set (Sept–Nov 2024):

| Stasiun | T1 (BAIK/SEDANG) | T2 (SEDANG/TIDAK SEHAT) |
|---|---|---|
| DKI1 | 50 | 92 |
| DKI2 | 68 | 107 |
| DKI3 | 38 | 94 |
| DKI4 | 61 | 124 |
| DKI5 | 50 | 98 |

### Post-processing

Satu baris prediksi yang hilang diisi dengan modus dataset ("SEDANG") untuk memenuhi format submisi.

## 📁 Repository Structure

```
├── notebooks/
│   └── Arkavidia10_Datavidia_Three Params kata nopal.ipynb
├── reports/
│   └── Three Params kata nopal_Laporan.pdf
└── .gitignore
```

> **Catatan:** Folder `dataset/` tidak disertakan di repo ini karena dataset bersifat tidak untuk dipublikasikan.

## 👥 Team

**Nama Tim:** Three Params kata nopal

| Nama |
|---|
| Muhammad Naufal Muzaki |
| Marvel Irawan |
| Rafsanjani |
