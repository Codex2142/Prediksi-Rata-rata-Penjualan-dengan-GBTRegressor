# Prediksi Minat Pasar Menggunakan XGBoost Berbasis Big Data
## Latar Belakang
Perkembangan teknologi Big Data dan Artificial Intelligence memungkinkan analisis perilaku pasar dilakukan secara lebih akurat dan terukur. Dalam konteks ritel, kemampuan memprediksi minat pasar menjadi faktor penting untuk mendukung pengambilan keputusan bisnis seperti perencanaan stok, strategi harga, dan pengelolaan distribusi produk. Dataset M5 Forecasting merepresentasikan data penjualan harian dalam skala besar yang mencerminkan kondisi riil pasar ritel, sehingga sangat relevan digunakan sebagai studi kasus dalam penerapan teknik machine learning berbasis Big Data.
## Tujuan
1.  Mengolah data penjualan berskala besar secara terdistribusi.

2. Menghasilkan fitur-fitur penting yang merepresentasikan pola penjualan dan perilaku pasar.

3. Memprediksi nilai penjualan pada periode tertentu sebagai indikator minat pasar.

4. Mengevaluasi dan mengoptimalkan performa model prediksi secara sistematis.

## Dataset
Dataset yang digunakan adalah M5 Forecasting – Starter Data Exploration, yang berisi data penjualan harian produk ritel dari beberapa toko dan wilayah. Dataset ini mencakup informasi seperti:

1. Penjualan harian per produk dan toko

2. Kalender (hari, minggu, event, dan hari libur)

3. Harga jual produk dari waktu ke waktu

Dataset bersifat time series dan memiliki volume data yang besar, sehingga sesuai untuk pendekatan Big Data.

## Pednakatan
data processing dan machine learning dilakukan menggunakan PySpark

## Struktur Folder:
menjelaskan tentang susunan folder yag dikerjakan mahasiswa
```sh
UAS/
├── dataset/
│   ├── raw/
│   │   ├── calendar.csv
│   │   ├── sales_train_evaluation.csv
│   │   ├── sales_train_validation.csv
│   │   ├── sample_submission.csv
│   │   └── sell_prices.csv
│   │
│   └── cooked/
│       ├── base_table.parquet
│       ├── features.parquet
│       ├── train.parquet
│       ├── test.parquet
│       └── predictions.parquet
│
├── notebook/
│   ├── 0_PREPARATION.ipynb
│   ├── 1_DATASET.ipynb
│   ├── 2_EDA.ipynb   (opsional / Tableau)
│   ├── 3_TRANSFORM_FEATURE.ipynb
│   ├── 4_SPLIT.ipynb
│   ├── 5_MODELING.ipynb
│   ├── 6_EVALUATION.ipynb
│   ├── 7_TUNING.ipynb
│
├── xgboost/
│   ├── requirements.txt
│   └── spark_xgb_env.md
```

## Tabel Ringkasan

| Tahap           | Notebook | Input       | Output      |
| --------------- | -------- | ----------- | ----------- |
| Download        | 0        | google drive      | raw CSV     |
| Dataset         | 1        | raw CSV     | base_table  |
| EDA             | 2        | base_table  | Tableau     |
| **Feature Eng** | **3**    | base_table  | features    |
| Split           | 4        | features    | train/test  |
| Modeling        | 5        | train       | prediction |
| Evaluasi        | 6        | predictions | metric     |
| Tuning          | 7        | train       | best param |


## Tahap Pengerjaan
### ```0_PREPARATION.ipynb```
Tujuan

Menyiapkan environment dan dataset mentah

Input

- Kaggle M5 Dataset (download)

Proses

- Setup PySpark session

- Download dataset

- Validasi file (row count, schema)

Output
```sh
dataset/raw/*.csv
```

### ```1_DATASET.ipynb```
Tujuan

Membangun base table (tabel inti time series)

Input
```sh
dataset/raw/
  - calendar.csv
  - sales_train_validation.csv
  - sell_prices.csv
```

Proses (PySpark)

- Load CSV → Spark DataFrame

- Unpivot d_1 ... d_n → format long

- Join:

    - sales × calendar

    - sales × sell_prices

- Filter produk / store bila perlu

Output
```sh
dataset/cooked/base_table.parquet
```
Isi base_table

- date

- item_id

- store_id

- sales

- price

- event

- weekday

### ```2_EDA.ipynb``` (SKIP / TABLEAU)
Tujuan

Analisis visual & bisnis

Output (non-file)

- Dashboard Tableau

- Insight musiman, trend, event


### ```3_TRANSFORM_FEATURE.ipynb```
Tujuan

Feature engineering berbasis time series

Input
```sh
dataset/cooked/base_table.parquet
```

Proses (PySpark)

Feature Engineering di sini:

- Lag features (lag_7, lag_14, lag_28)

- Rolling mean / std

- Price change

- Event encoding

- Date features:

    - day, week, month

    - is_weekend

- Encoding kategori (StringIndexer)

Output
```sh
dataset/cooked/features.parquet
```
Data sudah numerik & siap ML

### ```4_SPLIT.ipynb```
Tujuan

Membagi data time-series dengan benar

Input
```sh
dataset/cooked/features.parquet
```

Proses

- Time-based split:

    - Train: sebelum T

    - Test: setelah T

- No random split

Output
```sh
dataset/cooked/train.parquet
dataset/cooked/test.parquet
```

### ```5_MODELING.ipynb```
Tujuan

Training model XGBoost berbasis Spark

Input
```sh
dataset/cooked/train.parquet
dataset/cooked/test.parquet
```

Proses

- VectorAssembler

- Train XGBoost (SparkXGBRegressor)

- Predict test set

Output
```sh
dataset/cooked/predictions.parquet
```

### ```6_EVALUATION.ipynb```
Tujuan

Evaluasi performa prediksi

Input
```sh
dataset/cooked/predictions.parquet
```

Proses

- RMSE

- MAE

- MAPE

- Analisis per store / item

Output

- Metric (print / csv)

- Insight performa model

### ```7_TUNING.ipynb```
Tujuan

Optimasi hyperparameter

Input
```sh
dataset/cooked/train.parquet
```

Proses

- Grid Search / Random Search

- Parameter:

    - max_depth

    - eta

    - subsample

    - colsample_bytree

- Cross validation berbasis waktu

Output

- Best parameter

- Model final (opsional disimpan)