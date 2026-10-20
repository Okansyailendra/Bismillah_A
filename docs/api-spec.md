# Spesifikasi Kontrak API (API Specification)
## Sound Lab: RESTful API Engine for Acoustic Classification

> **Ringkasan Eksekutif:**  
> Dokumen ini mendefinisikan kontrak antarmuka pemrograman aplikasi (API) RESTful untuk backend FastAPI *Sound Lab*. Dokumen ini mencakup endpoint inferensi berkas audio, informasi arsitektur model, dan pengecekan kesehatan server (*healthcheck*), lengkap dengan validasi muatan (*payload*), skema JSON terstruktur, kode status HTTP, serta pesan galat informatif berbahasa Indonesia.

---

## Daftar Isi
1. [Standar Protokol & Konvensi Global](#1-standar-protokol--konvensi-global)
2. [Ringkasan Daftar Endpoint](#2-ringkasan-daftar-endpoint)
3. [Spesifikasi Endpoint POST `/api/predict`](#3-spesifikasi-endpoint-post-apipredict)
4. [Spesifikasi Endpoint GET `/api/model-info`](#4-spesifikasi-endpoint-get-apimodel-info)
5. [Spesifikasi Endpoint GET `/api/health`](#5-spesifikasi-endpoint-get-apihealth)
6. [Format Kesalahan Baku (Standard Error Responses)](#6-format-kesalahan-baku-standard-error-responses)
7. [Toleransi Waktu Habis (Timeouts) & Batasan Beban](#7-toleransi-waktu-habis-timeouts--batasan-beban)

---

## 1. Standar Protokol & Konvensi Global

- **Format Data:** JSON (`application/json; charset=utf-8`) untuk seluruh respons dan pengiriman data terstruktur.
- **Pengunggahan File:** `multipart/form-data` dengan parameter form bernama `file`.
- **Ekstensi & Format Audio yang Diterima:** `.mp3`, `.wav`, `.ogg`, `.flac`, `.m4a`.
- **Pesan Galat:** Seluruh pesan kesalahan ditulis dalam **Bahasa Indonesia** yang jelas, mendidik, dan mengarahkan solusi.

---

## 2. Ringkasan Daftar Endpoint

| Metode HTTP | Path Endpoint | Peruntukan Utama | Tingkat Keamanan |
| :---: | :--- | :--- | :---: |
| **POST** | `/api/predict` | Menganalisis berkas audio, mengekstrak 10 fitur, dan mengembalikan klasifikasi serta visualisasi. | Terproteksi Rate Limit |
| **GET** | `/api/model-info` | Mengambil metadata ilmiah, daftar fitur, metrik evaluasi (85% akurasi), dan rata-rata kelas. | Akses Terbuka |
| **GET** | `/api/health` | Pengecekan kesiapan server (*liveness & readiness*), pemakaian RAM, dan dependensi ffmpeg. | Akses Terbuka |

---

## 3. Spesifikasi Endpoint POST `/api/predict`

### 3.1 Parameter Permintaan (Request)
- **Content-Type:** `multipart/form-data`
- **Body Form-Data:**
  - `file` (*binary*, Wajib): Berkas audio mentah (ekstensi `.mp3`, `.wav`, `.ogg`, `.flac`, atau `.m4a`). Ukuran maksimal: **15 MB**.

### 3.2 Validasi Permintaan
1. File tidak boleh kosong (ukuran $> 0$ bytes).
2. Tipe MIME harus berupa jenis audio yang valid.
3. Durasi audio diuji tidak melebihi 120 detik (audio $> 120$ detik ditolak dengan HTTP 422).

### 3.3 Skema Respons Sukses (HTTP 200 OK)

```json
{
  "status": "success",
  "prediction": "berisik",
  "confidence": 0.8902,
  "is_low_confidence": false,
  "probabilities": {
    "berisik": 0.8902,
    "non-berisik": 0.1098
  },
  "audio_metadata": {
    "filename": "24-003 berisik (1).mp3",
    "duration_sec": 27.24,
    "active_duration_sec": 26.85,
    "sample_rate": 22050,
    "channels": "mono",
    "num_chunks": 9
  },
  "features": {
    "rms_mean": 0.04065,
    "mfcc1_mean": -180.24,
    "rms_std": 0.01205,
    "mfcc3_mean": 19.32,
    "zcr_mean": 0.0612,
    "mfcc6_mean": -4.85,
    "mfcc7_mean": 11.20,
    "mfcc13_mean": -2.45,
    "mfcc5_mean": -5.90,
    "centroid_mean": 1781.20
  },
  "class_comparison": {
    "rms_mean": {
      "sample": 0.04065,
      "train_noisy_mean": 0.06120,
      "train_quiet_mean": 0.01850,
      "closest_class": "berisik"
    },
    "centroid_mean": {
      "sample": 1781.20,
      "train_noisy_mean": 2045.00,
      "train_quiet_mean": 1620.00,
      "closest_class": "berisik"
    }
  },
  "plain_explanation": "Sistem mengklasifikasikan audio ini sebagai BERISIK karena terdeteksi intensitas energi suara (RMS) yang relatif tinggi serta pola frekuensi tajam yang melampaui ambang batas kesunyian ruang belajar.",
  "chunk_timeline": [
    {
      "chunk_index": 1,
      "start_sec": 0.0,
      "end_sec": 3.0,
      "prediction": "berisik",
      "confidence": 0.8420,
      "rms": 0.0385
    },
    {
      "chunk_index": 2,
      "start_sec": 3.0,
      "end_sec": 6.0,
      "prediction": "berisik",
      "confidence": 0.9150,
      "rms": 0.0442
    },
    {
      "chunk_index": 3,
      "start_sec": 6.0,
      "end_sec": 9.0,
      "prediction": "berisik",
      "confidence": 0.8870,
      "rms": 0.0410
    }
  ],
  "visualizations": {
    "waveform_downsampled": [
      0.012, -0.015, 0.045, -0.052, 0.078, -0.082, 0.032, -0.021
    ],
    "spectrogram_image_base64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...",
    "mfcc_13_profile": [
      -180.24, 110.45, 19.32, -8.12, -5.90, -4.85, 11.20, -1.45, 3.20, -6.12, 0.85, -2.10, -2.45
    ]
  }
}
```

---

## 4. Spesifikasi Endpoint GET `/api/model-info`

Digunakan oleh frontend untuk merender halaman "Tentang Model" dan tabel metodologi ilmiah.

### 4.1 Permintaan (Request)
- Tanpa parameter.

### 4.2 Skema Respons Sukses (HTTP 200 OK)

```json
{
  "model_name": "Random Forest Classifier",
  "library": "scikit-learn 1.8.0",
  "hyperparameters": {
    "n_estimators": 100,
    "max_depth": 10,
    "random_state": 42,
    "criterion": "gini",
    "n_features_in": 10
  },
  "dataset_summary": {
    "total_samples": 198,
    "train_samples": 158,
    "test_samples": 40,
    "split_ratio": "80:20 (Stratified)",
    "duration_per_sample_sec": "~30"
  },
  "metrics": {
    "test_accuracy": 0.8500,
    "precision": 0.8500,
    "recall": 0.8500,
    "f1_score": 0.8500,
    "roc_auc": 0.8688,
    "cross_val_5fold_mean": 0.8415,
    "cross_val_5fold_std": 0.0294
  },
  "confusion_matrix_test": {
    "noisy_true_positive": 17,
    "noisy_false_negative": 3,
    "quiet_true_negative": 17,
    "quiet_false_positive": 3
  },
  "benchmark_comparison": [
    { "model": "Random Forest", "test_accuracy": 0.850, "is_selected": true },
    { "model": "Gradient Boosting", "test_accuracy": 0.800, "is_selected": false },
    { "model": "SVM (RBF Kernel)", "test_accuracy": 0.775, "is_selected": false },
    { "model": "SVM (Linear Kernel)", "test_accuracy": 0.725, "is_selected": false },
    { "model": "Logistic Regression", "test_accuracy": 0.700, "is_selected": false },
    { "model": "K-Nearest Neighbors", "test_accuracy": 0.700, "is_selected": false }
  ],
  "selected_features": [
    { "rank": 1, "name": "rms_mean", "importance_mdi": 0.169, "unit": "amplitudo linier" },
    { "rank": 2, "name": "mfcc1_mean", "importance_mdi": 0.094, "unit": "koefisien cepstral" },
    { "rank": 3, "name": "rms_std", "importance_mdi": 0.092, "unit": "deviasi standar" },
    { "rank": 4, "name": "mfcc3_mean", "importance_mdi": 0.087, "unit": "koefisien cepstral" },
    { "rank": 5, "name": "zcr_mean", "importance_mdi": 0.056, "unit": "rasio persilangan" },
    { "rank": 6, "name": "mfcc6_mean", "importance_mdi": 0.053, "unit": "koefisien cepstral" },
    { "rank": 7, "name": "mfcc7_mean", "importance_mdi": 0.053, "unit": "koefisien cepstral" },
    { "rank": 8, "name": "mfcc13_mean", "importance_mdi": 0.045, "unit": "koefisien cepstral" },
    { "rank": 9, "name": "mfcc5_mean", "importance_mdi": 0.045, "unit": "koefisien cepstral" },
    { "rank": 10, "name": "centroid_mean", "importance_mdi": 0.044, "unit": "Hertz (Hz)" }
  ],
  "class_means": {
    "berisik": {
      "rms_mean": 0.0612,
      "mfcc1_mean": -165.4,
      "rms_std": 0.0215,
      "mfcc3_mean": 18.4,
      "zcr_mean": 0.089,
      "mfcc6_mean": -4.2,
      "mfcc7_mean": 12.1,
      "mfcc13_mean": -2.8,
      "mfcc5_mean": -6.5,
      "centroid_mean": 2045.0
    },
    "non-berisik": {
      "rms_mean": 0.0185,
      "mfcc1_mean": -245.8,
      "rms_std": 0.0078,
      "mfcc3_mean": 8.1,
      "zcr_mean": 0.042,
      "mfcc6_mean": -1.1,
      "mfcc7_mean": 5.4,
      "mfcc13_mean": -0.9,
      "mfcc5_mean": -2.2,
      "centroid_mean": 1620.0
    }
  }
}
```

---

## 5. Spesifikasi Endpoint GET `/api/health`

Digunakan oleh frontend untuk memantau status server aktif, waktu bangun (*cold start*), dan alokasi memori.

### 5.1 Permintaan (Request)
- Tanpa parameter.

### 5.2 Skema Respons Sukses (HTTP 200 OK)

```json
{
  "status": "healthy",
  "model_loaded": true,
  "scikit_learn_version": "1.8.0",
  "ffmpeg_available": true,
  "memory_usage_mb": 182.4,
  "uptime_seconds": 3600
}
```

---

## 6. Format Kesalahan Baku (Standard Error Responses)

Semua galat diformat seragam menggunakan skema JSON berikut:

```json
{
  "status": "error",
  "error_code": "STRING_IDENTIFIER",
  "message": "Pesan deskriptif dalam Bahasa Indonesia yang menjelaskan akar masalah dan panduan perbaikan.",
  "details": null
}
```

### Tabel Status Galat HTTP

| Kode Status HTTP | `error_code` | Kapan Terjadi | Contoh Pesan Bahasa Indonesia |
| :---: | :--- | :--- | :--- |
| **400 Bad Request** | `EMPTY_FILE` | Berkas berukuran 0 byte diunggah. | `"Berkas audio yang diunggah kosong (0 byte). Harap pilih berkas audio yang valid."` |
| **413 Payload Too Large** | `FILE_TOO_LARGE` | Ukuran file melebihi batas 15 MB. | `"Ukuran berkas melebihi batas maksimum 15 MB. Silakan kompres atau pilih berkas lain."` |
| **415 Unsupported Media Type** | `INVALID_EXTENSION` | Format di luar daftar ekstensi yang didukung. | `"Format berkas tidak didukung. Sistem hanya menerima berkas .mp3, .wav, .ogg, .flac, dan .m4a."` |
| **422 Unprocessable Entity** | `DURATION_EXCEEDED` | Durasi audio melebihi 120 detik. | `"Durasi audio melebihi batas 120 detik. Harap potong audio sebelum mengunggah."` |
| **422 Unprocessable Entity** | `CORRUPT_AUDIO` | Stream biner bukan format audio yang dapat didecode oleh librosa/ffmpeg. | `"Berkas audio rusak atau tidak dapat didekode oleh sistem pengolah sinyal."` |
| **429 Too Many Requests** | `RATE_LIMIT_EXCEEDED` | Klien mengirim $> 10$ request per menit. | `"Batas permintaan terlampaui (maks 10 kali/menit). Mohon tunggu beberapa saat sebelum mencoba lagi."` |
| **500 Internal Server Error** | `INFERENCE_FAILURE` | Kegagalan tidak terduga pada pemrosesan model/DSP di peladen. | `"Terjadi kesalahan internal pada peladen saat memproses analisis audio. Silakan coba kembali."` |

---

## 7. Toleransi Waktu Habis (Timeouts) & Batasan Beban

- **Server Timeout:** Uvicorn disetel dengan toleransi timeout worker sebesar **30 detik**.
- **Client Fetch Timeout:** Klien Next.js (`api-client.ts`) menetapkan batas waktu pembatalan (*abort timeout*) sebesar **25 detik**. Jika respons belum diterima dalam 25 detik, klien menampilkan notifikasi kegagalan koneksi ramah pengguna.
- **Ukuran Maksimum Payload Form:** Dibatasi ketat pada konfigurasi ASGI/Uvicorn sebesar `16 * 1024 * 1024` bytes (16 MB) untuk mencegah serangan penolakan layanan (*Denial of Service* via upload tak terbatas).
