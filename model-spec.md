# Spesifikasi Model & Pipeline Ekstraksi Fitur (Model Specification)
## Sound Lab: Random Forest Classifier & Digital Signal Processing Pipeline

> **Ringkasan Eksekutif:**  
> Dokumen ini adalah spesifikasi matematis, teknis, dan algoritmis untuk model Machine Learning serta pipeline Pengolahan Sinyal Digital (PSD) pada aplikasi *Sound Lab*. Memuat struktur biner artefak `joblib`, pseudocode ekstraksi fitur deterministik langkah-demi-langkah, kalkulasi penjelas (*explainability*), logika prediksi segmen per-chunk, dan keputusan penting mengenai paritas *training-vs-inference* (`split` vs `trim`).

---

## Daftar Isi
1. [Spesifikasi Artefak Model (`.pkl`)](#1-spesifikasi-artefak-model-pkl)
2. [Hyperparameter & Karakteristik Pelatihan](#2-hyperparameter--karakteristik-pelatihan)
3. [Daftar 10 Fitur Terpilih & Arti Fisiknya](#3-daftar-10-fitur-terpilih--arti-fisiknya)
4. [Pipeline Ekstraksi Fitur Lengkap (Pseudocode)](#4-pipeline-ekstraksi-fitur-lengkap-pseudocode)
5. [Keputusan Desain Paritas: `split` vs `trim`](#5-keputusan-desain-paritas-split-vs-trim)
6. [Mekanisme Penjelasan Fitur (Explainability Engine)](#6-mekanisme-penjelasan-fitur-explainability-engine)
7. [Prediksi Per-Chunk & Penandaan "Indikatif"](#7-prediksi-per-chunk--penandaan-indikatif)
8. [Kalibrasi Ambang Keyakinan Rendah (Confidence Threshold)](#8-kalibrasi-ambang-keyakinan-rendah-confidence-threshold)
9. [Keterbatasan Model & Distribusi Data Latih](#9-keterbatasan-model--distribusi-data-latih)

---

## 1. Spesifikasi Artefak Model (`.pkl`)

Artefak model disimpan dalam berkas biner serialisasi pustaka `joblib` (bukan pickle standar Python untuk efisiensi array numpy).

### 1.1 Struktur Kamus (Dictionary Payload)
Berkas `model_klasifikasi_audio.pkl` membungkus dictionary dengan struktur berikut:
```python
{
    'model': RandomForestClassifier,        # Objek estimator terlatih scikit-learn
    'selected_features': [                 # List 10 string nama fitur urutan wajib
        'rms_mean',
        'mfcc1_mean',
        'rms_std',
        'mfcc3_mean',
        'zcr_mean',
        'mfcc6_mean',
        'mfcc7_mean',
        'mfcc13_mean',
        'mfcc5_mean',
        'centroid_mean'
    ],
    'target_sr': 22050,                    # Sampling rate target DSP (Hz)
    'top_db': 25,                          # Ambang desibel pemotongan hening
    'chunk_duration': 3,                   # Durasi potongan waktu per chunk (detik)
    'model_name': 'Random Forest'          # Metadata pengenal model
}
```

### 1.2 Persyaratan Pustaka Inti (Pinned Dependencies)
Model ini dilatih menggunakan **`scikit-learn==1.8.0`**.
> **PERINGATAN KRITIS:**  
> Pemuatan objek `RandomForestClassifier` dari `joblib` sangat sensitif terhadap struktur pohon biner internal scikit-learn. Backend **WAJIB** mengunci versi `scikit-learn==1.8.0` di `requirements.txt`. Penggunaan versi di bawah atau di atasnya berisiko memicu `InconsistentVersionWarning`, kerusakan prediksi probabilitas, atau galat *unpickling*.

### 1.3 Label Kelas & Penentuan Probabilitas
Model membedakan dua kelas biner: `'berisik'` dan `'non-berisik'`.
- Aturan inferensi: **JANGAN PERNAH** melakukan *hardcode* indeks seperti `proba[0]` atau `proba[1]`.
- Pengembang **WAJIB** membaca atribut `model.classes_` secara dinamis:
```python
classes = model.classes_  # Array misalnya: ['berisik', 'non-berisik']
probs = model.predict_proba(X)[0]
prob_dict = {cls_name: float(prob) for cls_name, prob in zip(classes, probs)}
```

---

## 2. Hyperparameter & Karakteristik Pelatihan

### 2.1 Konfigurasi Hyperparameter
- **Estimator:** `RandomForestClassifier`
- `n_estimators`: `100` (100 pohon keputusan)
- `max_depth`: `10` (membatasi kompleksitas dan mencegah overfitting)
- `random_state`: `42`
- `n_features_in_`: `10`
- `criterion`: `'gini'`

### 2.2 Metrik Evaluasi Model (Data Uji 40 Sampel - Split 80:20 Stratified)
| Metrik Evaluasi | Nilai | Catatan Ilmiah |
| :--- | :--- | :--- |
| **Akurasi Pengujian (Test Accuracy)** | **85.00 %** | 34 dari 40 sampel uji terprediksi tepat |
| **Presisi (Precision)** | **0.85** | Keseimbangan antara false alarm dan deteksi |
| **Recall (Sensitivitas)** | **0.85** | 85% suara bising berhasil diidentifikasi |
| **F1-Score** | **0.85** | Rata-rata harmonik presisi dan recall |
| **ROC-AUC** | **0.8688** | Tingkat diskriminasi probabilitas tinggi |
| **5-Fold Cross Validation** | **84.15% (+/- 2.94%)** | Deviasi standar rendah menunjukkan model stabil |

### 2.3 Matriks Konfusi (Confusion Matrix Data Uji)
Tiap kelas memiliki 20 sampel uji:
- **Kelas Berisik:** 17 Terdeteksi Benar, 3 Salah Terklasifikasi Non-Berisik.
- **Kelas Non-Berisik:** 17 Terdeteksi Benar, 3 Salah Terklasifikasi Berisik.

---

## 3. Daftar 10 Fitur Terpilih & Arti Fisiknya

Awalnya diekstraksi 19 fitur kandidat: `rms` (mean, std), `centroid` (mean, std), `zcr` (mean, std), dan 13 koefisien `mfcc` (mean). Melalui teknik seleksi fitur *Random Forest Feature Importance (MDI)* dan uji *ANOVA F-score* dengan validasi 5-fold CV, ditemukan $k=10$ fitur paling optimal:

| No | Nama Fitur Wajib | Bobot Importance (MDI) | Definisi Fisik & Relevansi Akustik |
| :-: | :--- | :-: | :--- |
| **1** | **`rms_mean`** | **0.169** | **Root Mean Square Energy (Rata-rata):** Mengukur kekuatan/intensitas energi amplitudo rata-rata sinyal audio. Suara bising memiliki amplitudo energi jauh lebih tinggi daripada ruangan senyap. |
| **2** | **`mfcc1_mean`** | **0.094** | **Mel-Frequency Cepstral Coefficient 1 (Rata-rata):** Menggambarkan energi spektral keseluruhan pada pita frekuensi rendah (paling selaras dengan kenyaringan persepsi telinga manusia). |
| **3** | **`rms_std`** | **0.092** | **Standar Deviasi RMS:** Mengukur fluktuasi dinamika suara. Suara lingkungan bising (teriakan, gesekan barang) memiliki variasi energi yang melonjak-lonjak tinggi. |
| **4** | **`mfcc3_mean`** | **0.087** | **MFCC Koefisien 3:** Menangkap kurva kemiringan selubung spektrum (*spectral envelope slope*), membedakan resonansi ucapan manusia dari dengungan latar. |
| **5** | **`zcr_mean`** | **0.056** | **Zero Crossing Rate (Rata-rata):** Frekuensi pergantian tanda gelombang sinyal melewati nol. Suara mendesis atau gesekan kasar memiliki ZCR tinggi. |
| **6** | **`mfcc6_mean`** | **0.053** | **MFCC Koefisien 6:** Menangkap rincian bentuk spektrum frekuensi menengah. |
| **7** | **`mfcc7_mean`** | **0.053** | **MFCC Koefisien 7:** Menangkap variasi tekstur akustik timbre frekuensi menengah ke atas. |
| **8** | **`mfcc13_mean`**| **0.045** | **MFCC Koefisien 13:** Komponen frekuensi tinggi cepstral yang mewakili detail halus kebisingan berfrekuensi tajam. |
| **9** | **`mfcc5_mean`** | **0.045** | **MFCC Koefisien 5:** Menggambarkan komponen harmonik suara instrumen / vokal manusia. |
| **10**| **`centroid_mean`**| **0.044**| **Spectral Centroid (Rata-rata):** "Pusat massa" spektrum frekuensi (dinyatakan dalam Hz). Menunjukkan apakah suara didominasi nada rendah (bass/hangat) atau nada tinggi (tajam/menoreh telinga). |

> **PERHATIAN URUTAN MATRIKS:**  
> Scikit-learn mengevaluasi pohon berdasarkan indeks kolom array. Urutan ke-10 fitur di atas adalah **MUTLAK**. Jangan pernah mengubah posisi kolom saat membangun DataFrame inferensi.

---

## 4. Pipeline Ekstraksi Fitur Lengkap (Pseudocode)

Pipeline ini wajib dieksekusi persis sama dengan pipeline saat pelatihan data.

```python
import librosa
import numpy as np
import pandas as pd

def extract_features_pipeline(audio_file_path: str, target_sr=22050, top_db=25, chunk_duration=3):
    # 1. Pembacaan Audio (Resampling ke 22050 Hz & Konversi Mono)
    y, sr = librosa.load(audio_file_path, sr=target_sr, mono=True)
    
    # 2. Pembuangan Bagian Hening (Silence Removal via Split & Concatenate)
    # PENTING: Menggunakan split, bukan trim!
    intervals = librosa.effects.split(y, top_db=top_db)
    if len(intervals) > 0:
        y_active = np.concatenate([y[start:end] for start, end in intervals])
    else:
        y_active = y  # Fallback jika audio sangat hening atau split kosong
        
    # 3. Chunking Segmen 3 Detik (66.150 Sampel Tanpa Overlap)
    chunk_samples = chunk_duration * target_sr  # 3 * 22050 = 66150
    total_samples = len(y_active)
    
    # Penanganan jika sinyal aktif lebih pendek dari 1 chunk
    if total_samples < chunk_samples:
        pad_width = chunk_samples - total_samples
        y_active = np.pad(y_active, (0, pad_width), mode='constant')
        total_samples = len(y_active)
        
    num_chunks = total_samples // chunk_samples  # Sisa di akhir dibuang
    
    # List penampung matriks fitur per chunk
    rms_list = []
    centroid_list = []
    zcr_list = []
    mfcc_list = []  # Akan berisi array 13 koefisien
    
    # 4. Ekstraksi Fitur Per Chunk
    for i in range(num_chunks):
        start_idx = i * chunk_samples
        end_idx = start_idx + chunk_samples
        chunk = y_active[start_idx:end_idx]
        
        # Ekstraksi fitur dasar pada chunk
        c_rms = librosa.feature.rms(y=chunk)
        c_centroid = librosa.feature.spectral_centroid(y=chunk, sr=target_sr)
        c_zcr = librosa.feature.zero_crossing_rate(y=chunk)
        c_mfcc = librosa.feature.mfcc(y=chunk, sr=target_sr, n_mfcc=13)
        
        rms_list.append(c_rms)
        centroid_list.append(c_centroid)
        zcr_list.append(c_zcr)
        mfcc_list.append(c_mfcc)
        
    # 5. Penggabungan Horizontal (hstack) & Agregasi Waktu (Mean/Std)
    rms_all = np.hstack(rms_list)
    centroid_all = np.hstack(centroid_list)
    zcr_all = np.hstack(zcr_list)
    mfcc_all = np.hstack(mfcc_list)  # Shape: (13, total_frames)
    
    # Agregasi statistik
    rms_mean = float(np.mean(rms_all))
    rms_std = float(np.std(rms_all))
    centroid_mean = float(np.mean(centroid_all))
    centroid_std = float(np.std(centroid_all))
    zcr_mean = float(np.mean(zcr_all))
    zcr_std = float(np.std(zcr_all))
    mfcc_means = np.mean(mfcc_all, axis=1)  # 13 nilai rata-rata
    
    # 6. Pemetaan Tepat ke 10 Fitur Terpilih (Urutan Wajib)
    feature_dict = {
        'rms_mean': rms_mean,
        'mfcc1_mean': float(mfcc_means[0]),
        'rms_std': rms_std,
        'mfcc3_mean': float(mfcc_means[2]),
        'zcr_mean': zcr_mean,
        'mfcc6_mean': float(mfcc_means[5]),
        'mfcc7_mean': float(mfcc_means[6]),
        'mfcc13_mean': float(mfcc_means[12]),
        'mfcc5_mean': float(mfcc_means[4]),
        'centroid_mean': centroid_mean
    }
    
    # Bentuk DataFrame 1 baris
    ordered_columns = [
        'rms_mean', 'mfcc1_mean', 'rms_std', 'mfcc3_mean', 'zcr_mean',
        'mfcc6_mean', 'mfcc7_mean', 'mfcc13_mean', 'mfcc5_mean', 'centroid_mean'
    ]
    df_features = pd.DataFrame([feature_dict])[ordered_columns]
    
    return df_features, feature_dict, num_chunks
```

---

## 5. Keputusan Desain Paritas: `split` vs `trim`

### 5.1 Analisis Masalah
Saat eksperimen awal di Jupyter Notebook:
- **Notebook Training:** Menggunakan `librosa.effects.split(top_db=25)` dan menyambung semua interval aktif (`np.concatenate`). Metode ini memotong keheningan di **awal, tengah, dan akhir** rekaman.
- **Notebook Inferensi Lama:** Menggunakan fungsi `librosa.effects.trim(top_db=25)` yang **hanya memotong keheningan di awal dan akhir**, namun membiarkan jeda hening di tengah file. Akibatnya, durasi chunk dan nilai agregasi fitur menjadi sedikit berbeda.

### 5.2 Keputusan Desain Resmi
> **KEPUTUSAN:** Aplikasi Web Backend **WAJIB MENGGUNAKAN METODE `split + concatenate`**.

**Rasionalitas:**
1. Model `Random Forest` dilatih dan dioptimasi menggunakan fitur hasil `split + concatenate`. Jika inferensi memakai `trim`, kita melanggar asumsi dasar Machine Learning (*Training-Serving Skew*).
2. Dengan `split + concatenate`, segmen jeda bisu di tengah rekaman tidak akan mendilusi nilai rata-rata `rms_mean` atau mengacaukan bentuk koefisien MFCC.

**Dampak pada Pengujian Paritas:**
Sampel audio notebook (misal: `"24-003 berisik (1).mp3"`) sebelumnya dievaluasi memakai fungsi lama (`trim`), menghasilkan nilai `berisik 89%`, `rms_mean 0.04065`, dan `centroid_mean 1781.2 Hz`. Ketika dijalankan dengan `split + concatenate`, nilai numerik akhir mungkin memiliki sedikit deviasi (toleransi $\pm 2-5\%$), namun label kelas dan urutan magnitudo tetap konsisten. Hal ini dicatat sebagai toleransi resmi di `testing.md`.

---

## 6. Mekanisme Penjelasan Fitur (Explainability Engine)

Untuk membuat sistem tidak menjadi *black box*, sistem menyediakan data referensi `data/class_means.json` yang dihitung dari rata-rata data latih (`train_data.csv`).

### 6.1 Contoh Format `class_means.json`
```json
{
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
```

### 6.2 Logika Pembuatan Narasi Awam
Backend menghitung selisih persentase antara nilai sampel terhadap rata-rata kelas:
```python
def generate_plain_explanation(sample_features, prediction):
    rms = sample_features['rms_mean']
    centroid = sample_features['centroid_mean']
    
    if prediction == "berisik":
        if rms >= 0.040:
            energy_desc = "tingkat intensitas energi audio (RMS) yang tinggi dan padat"
        else:
            energy_desc = "dinamika perubahan amplitudo suara yang tajam"
            
        if centroid >= 1900:
            pitch_desc = "didominasi oleh frekuensi tinggi yang menusuk (seperti gesekan atau vokal keras)"
        else:
            pitch_desc = "spektrum frekuensi menengah yang konstan memenuhi ruangan"
            
        return f"Sistem mengklasifikasikan rekaman ini sebagai BERISIK karena terdeteksi {energy_desc}, serta {pitch_desc}."
    else:
        return "Sistem mengklasifikasikan rekaman ini sebagai NON-BERISIK karena profil energi (RMS) berada pada batas senyap dengan latar frekuensi yang stabil tanpa lonjakan kebisingan tajam."
```

---

## 7. Prediksi Per-Chunk & Penandaan "Indikatif"

Pengguna ingin melihat segmen mana dalam audio 30 detik yang paling bising.

### 7.1 Mekanisme Inferensi Chunk
Untuk setiap chunk $i$ ($i \in [0, \text{num\_chunks}-1]$):
1. Hitung 10 fitur lokal hanya dari chunk tersebut (mean dari frame di dalam chunk 3 detik).
2. Jalankan `model.predict_proba(chunk_df)`.
3. Simpan stempel waktu `start_sec = i * 3`, `end_sec = (i + 1) * 3`, label chunk, dan confidence.

### 7.2 Label "Indikatif"
Karena model dilatih pada **agregat 30 detik** (bukan dilatih khusus pada potongan 3 detik), prediksi per-chunk memiliki potensi variabilitas lebih tinggi. Pada antarmuka pengguna, komponen timeline **WAJIB** mencantumkan label keterangan:
> *"Status Per-Chunk: INDIKATIF (Estimasi segmen lokal - Model utama dioptimasi untuk sinyal keseluruhan durasi)."*

---

## 8. Kalibrasi Ambang Keyakinan Rendah (Confidence Threshold)

- **Ambang Batas Peringatan:** **`65%` (0.65)**
- Jika skor keyakinan kelas pemenang $< 0.65$:
  - Tandai atribut JSON respons `is_low_confidence = True`.
  - Berikan rekomendasi mitigasi di UI:
    *"Peringatan: Tingkat kepastian model rendah (< 65%). Karakteristik suara berada pada perbatasan atau mengandung perpaduan suara hening dan bising secara acak. Disarankan mengambil sampel ulang."*

---

## 9. Keterbatasan Model & Distribusi Data Latih

1. **Skala Dataset Latih:** Model dikembangkan dengan 158 sampel latih dan 40 sampel uji. Akibatnya, kemampuan generalisasi dibatasi oleh variasi akustik ruang kelas dan gedung perkuliahan tempat data direkam.
2. **Domain Tunggal:** Perekaman dilakukan pada ruangan tertutup dan selasar gedung kampus. Model belum terkalibrasi untuk:
   - Kebisingan lalu lintas jalan raya berat (deru knalpot motor blong terus-menerus).
   - Polusi suara alam seperti petir, hujan badai lebat pada atap seng.
   - Sinyal musik terkompresi dengan bass elektronik tinggi (*sub-bass 40-80 Hz*).
3. **Sensitivitas Perangkat Keras:** Perekaman mikrofon laptop murah dengan peredam kebisingan otomatis (*built-in hardware noise cancellation*) dapat menekan nilai RMS secara artifisial, yang berpotensi menghasilkan *False Negative* (suara bising terdeteksi non-berisik).
