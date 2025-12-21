# Prediksi Minat Pasar Menggunakan XGBoost Berbasis Big Data
## Latar Belakang
Perkembangan teknologi Big Data dan Artificial Intelligence memungkinkan analisis perilaku pasar dilakukan secara lebih akurat dan terukur. Dalam konteks ritel, kemampuan memprediksi minat pasar menjadi faktor penting untuk mendukung pengambilan keputusan bisnis seperti perencanaan stok, strategi harga, dan pengelolaan distribusi produk. Dataset M5 Forecasting merepresentasikan data penjualan harian dalam skala besar yang mencerminkan kondisi riil pasar ritel, sehingga sangat relevan digunakan sebagai studi kasus dalam penerapan teknik machine learning berbasis Big Data.
## Tujuan
1.  Mengolah data penjualan berskala besar secara terdistribusi.

2. Menghasilkan fitur-fitur penting yang merepresentasikan pola penjualan dan perilaku pasar.

3. Memprediksi nilai penjualan pada periode tertentu sebagai indikator minat pasar.

4. Mengevaluasi dan mengoptimalkan performa model prediksi secara sistematis.

## Dataset
Dataset yang digunakan adalah M5 Forecasting – Starter Data Exploration, yang terdiri dari:

- Penjualan harian per produk dan toko

- Kalender (hari, minggu, event, dan hari libur)

- Harga jual produk dari waktu ke waktu

Karakteristik dataset:

- Bertipe time series

- Multilevel (item, store, state)

- Volume data besar

- Cocok untuk pendekatan Big Data dan distributed machine learning

## Pednakatan
Data Processing & Feature Engineering: PySpark

Modeling: XGBoost berbasis Spark (SparkXGBRegressor)

Evaluasi: Time-based evaluation (RMSE, MAE, MAPE)

Optimasi: Hyperparameter tuning berbasis time series split

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
│       ├── predictions.parquet
│       └── metrics.csv
│
├── notebook/
│   ├── 0_PREPARATION.ipynb
│   ├── 1_DATASET.ipynb
│   ├── 2_EDA.ipynb        (opsional / Tableau)
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

| Tahap       | Notebook | Input          | Output         |
| ----------- | -------- | -------------- | -------------- |
| Preparation | 0        | Kaggle Dataset | Raw CSV        |
| Dataset     | 1        | Raw CSV        | Base Table     |
| EDA         | 2        | Base Table     | Dashboard      |
| Feature Eng | 3        | Base Table     | Features       |
| Split       | 4        | Features       | Train/Test     |
| Modeling    | 5        | Train/Test     | Predictions    |
| Evaluation  | 6        | Predictions    | Metrics        |
| Tuning      | 7        | Train          | Best Parameter |



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

Membagi data berdasarkan waktu (time-based split).

Input

- features.parquet

Proses

- Train: sebelum waktu T

- Test: setelah waktu T

- Tanpa random split

Output
```sh
train.parquet
test.parquet
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
skema prediksi:
```sh
date
item_id
store_id
y_true
y_pred

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
```sh
Metric (print / csv)
```

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

    - ```max_depth```

    - ```eta```

    - ```subsample```

    - ```colsample_bytree```

- Cross validation berbasis waktu

Output

- Best parameter

- Model final (opsional disimpan)