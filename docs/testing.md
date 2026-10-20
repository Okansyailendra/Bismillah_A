# Rencana & Strategi Pengujian (Testing Strategy)
## Sound Lab: Quality Assurance, Feature Parity, & Reliability Testing

> **Ringkasan Eksekutif:**  
> Dokumen ini menjabarkan seluruh strategi pengujian untuk memverifikasi keakuratan, stabilitas, dan keandalan platform *Sound Lab*. Mencakup pengujian paritas numerik Pengolahan Sinyal Digital (PSD) terhadap hasil eksperimen notebook, pengujian integrasi API, pengujian responsivitas dan aksesibilitas antarmuka pengguna, simulasi kasus tepi (*edge cases*), serta matriks uji terstruktur.

---

## Daftar Isi
1. [Strategi Pengujian Menyeluruh](#1-strategi-pengujian-menyeluruh)
2. [Uji Paritas Fitur Numerik (Ground Truth Notebook)](#2-uji-paritas-fitur-numerik-ground-truth-notebook)
3. [Uji Validitas Urutan Fitur Model](#3-uji-validitas-urutan-fitur-model)
4. [Uji Kasus Tepi (Edge Cases Testing)](#4-uji-kasus-tepi-edge-cases-testing)
5. [Uji Integrasi API Backend (FastAPI TestClient)](#5-uji-integrasi-api-backend-fastapi-testclient)
6. [Checklist Pengujian Antarmuka UI & Breakpoint](#6-checklist-pengujian-antarmuka-ui--breakpoint)
7. [Uji Aksesibilitas (WCAG 2.1 AA) & Audit Lighthouse](#7-uji-aksesibilitas-wcag-21-aa--audit-lighthouse)
8. [Uji Performa, Latensi, & Manajemen Memori](#8-uji-performa-latensi--manajemen-memori)
9. [Matriks Pengujian Sistem (Test Matrix)](#9-matriks-pengujian-sistem-test-matrix)

---

## 1. Strategi Pengujian Menyeluruh

Pengujian dilakukan melalui piramida uji bertingkat:
1. **Unit Test (Python):** Memverifikasi fungsi matematika librosa, pemotongan hening (*split*), segmentasi chunk, dan agregasi 10 fitur.
2. **Parity Test:** Membandingkan output inferensi backend terhadap hasil rekaman sampel notebook referensi.
3. **Integration Test:** Memvalidasi pertukaran data antara Next.js, FastAPI, dan Supabase.
4. **End-to-End (E2E) & Manual Verification:** Memastikan pengalaman pengguna mulus di berbagai resolusi layar dan peramban.

---

## 2. Uji Paritas Fitur Numerik (Ground Truth Notebook)

### 2.1 Data Sampel Referensi Notebook
Terdapat 3 berkas audio acuan yang telah diuji pada notebook eksperimen:

| Berkas Sampel Uji | Prediksi Notebook | Keyakinan (*Confidence*) | Nilai RMS Mean | Centroid Mean (Hz) | Jumlah Chunk |
| :--- | :---: | :---: | :---: | :---: | :---: |
| `24-003 berisik (1).mp3` | **berisik** | 89.0 % | 0.04065 | 1781.2 Hz | 9 chunk |
| `24-003 berisik (2).mp3` | **berisik** | 98.0 % | 0.07744 | 2132.0 Hz | 10 chunk |
| `24-003 sunyi 2.mp3` | **non-berisik**| 92.0 % | 0.02091 | 1729.9 Hz | 10 chunk |

### 2.2 Batas Toleransi Paritas & Rasionalitas
> **CATATAN REKAYASA KRITIS:**  
> Fungsi inferensi lama pada notebook menggunakan `librosa.effects.trim` (hanya memotong jeda hening di ujung depan/belakang). Sedangkan pipeline backend resmi menggunakan `librosa.effects.split(top_db=25)` + `np.concatenate` (memotong seluruh jeda hening termasuk di tengah rekaman) agar identik 100% dengan data pelatihan.

Oleh karena perbedaan perlakuan hening di tengah file:
- **Toleransi Label Prediksi:** **Wajib 100% Cocok (0% toleransi)**. Kelas harus tetap `'berisik'` atau `'non-berisik'` sesuai acuan.
- **Toleransi Confidence:** Toleransi $\pm 5\%$ (misal: 89% dapat bernilai antara 84% - 94%).
- **Toleransi RMS Mean:** Toleransi deviasi relatif $\le 6\%$ ($|\text{RMS}_{\text{web}} - \text{RMS}_{\text{nb}}| / \text{RMS}_{\text{nb}} \le 0.06$).
- **Toleransi Spectral Centroid:** Toleransi deviasi relatif $\le 4\%$ ($|\text{Centroid}_{\text{web}} - \text{Centroid}_{\text{nb}}| / \text{Centroid}_{\text{nb}} \le 0.04$).
- **Toleransi Jumlah Chunk:** $\pm 1$ chunk bergantung pada jumlah detik jeda hening yang dibuang.

---

## 3. Uji Validitas Urutan Fitur Model

Model `RandomForestClassifier` dari scikit-learn tidak memvalidasi nama kolom saat diberikan matriks numpy murni. Oleh karena itu, pengujian otomatis wajib memverifikasi bahwa DataFrame yang dioper ke `model.predict()` memiliki tepat 10 kolom dengan nama dan urutan persis:

```python
EXPECTED_COLUMNS = [
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
]

def test_feature_order_integrity(extracted_dataframe):
    assert list(extracted_dataframe.columns) == EXPECTED_COLUMNS
    assert extracted_dataframe.shape == (1, 10)
    assert not extracted_dataframe.isnull().values.any()
```

---

## 4. Uji Kasus Tepi (Edge Cases Testing)

Sistem harus teruji tahan banting terhadap berbagai anomali berkas dan sinyal:

| Skenario Kasus Tepi | Karakteristik Audio Input | Perilaku Sistem yang Diharapkan |
| :--- | :--- | :--- |
| **Berkas Nol Byte** | Berkas berukuran 0 KB. | Backend menolak langsung dengan HTTP 400 Bad Request (`EMPTY_FILE`). |
| **Audio Sangat Pendek (< 3 detik)** | Audio durasi 1.2 detik (26.460 sampel). | Pipeline melakukan *zero-padding* pada sinyal hingga tepat 66.150 sampel (1 chunk) tanpa gagal. |
| **Audio Stereo (2 Channel)** | Berkas rekaman ganda kiri-kanan. | `librosa.load(..., mono=True)` otomatis meratakan channel menjadi mono rata-rata tanpa galat bentuk array. |
| **Senyap Total (Digital Zero)** | Amplitudo audio datar $y = 0.0$. | `librosa.effects.split` mengembalikan array kosong; pipeline fallback ke sinyal asli; RMS bernilai 0.0; terklasifikasi `non-berisik`. |
| **Audio Sangat Keras (Clipping)** | Audio distorsi dengan amplitudo saturasi penuh $1.0$. | Terproses normal, RMS terdeteksi sangat tinggi ($> 0.3$), terklasifikasi `berisik` dengan confidence mendekati 100%. |
| **Berkas Non-Audio / Ekstensi Palsu** | Berkas teks `.txt` yang diubah namanya menjadi `.mp3`. | Decoder `soundfile/audioread` gagal membaca stream biner; sistem menangkap exception dan mengembalikan HTTP 422 (`CORRUPT_AUDIO`). |
| **Audio Melebihi Durasi Maksimal** | Berkas audio durasi 180 detik (> 120 detik). | Backend menolak dengan HTTP 422 (`DURATION_EXCEEDED`) sebelum ekstraksi fitur dimulai. |

---

## 5. Uji Integrasi API Backend (FastAPI TestClient)

Contoh skrip pengujian otomatis menggunakan `pytest` dan `httpx.AsyncClient`:

```python
import pytest
from httpx import AsyncClient
from app.main import app

@pytest.mark.asyncio
async def test_health_endpoint():
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.get("/api/health")
        assert response.status_code == 200
        data = response.json()
        assert data["status"] == "healthy"
        assert data["model_loaded"] is True
        assert data["scikit_learn_version"] == "1.8.0"

@pytest.mark.asyncio
async def test_predict_success():
    async with AsyncClient(app=app, base_url="http://test") as client:
        with open("samples/24-003 berisik (1).mp3", "rb") as f:
            files = {"file": ("sample.mp3", f, "audio/mpeg")}
            response = await client.post("/api/predict", files=files)
            
        assert response.status_code == 200
        data = response.json()
        assert data["status"] == "success"
        assert data["prediction"] in ["berisik", "non-berisik"]
        assert 0.0 <= data["confidence"] <= 1.0
        assert len(data["chunk_timeline"]) >= 1
```

---

## 6. Checklist Pengujian Antarmuka UI & Breakpoint

### 6.1 Uji Breakpoint Responsif
- [ ] **Mobile (360px):**
  - Dropzone dan perekam suara tertata vertikal rapi tanpa *horizontal scrolling*.
  - Gauge confidence tidak terpotong tepi layar.
  - Tabel riwayat dapat digeser (*scrollable*) dengan label kolom tetap terbaca.
- [ ] **Tablet (768px):**
  - Grid 2 kolom berimbang antara panel hasil dan pemutar audio.
- [ ] **Desktop (1280px+):**
  - Tata letak 12 kolom penuh dengan sidebar statis 240px di sisi kiri.
  - Garis kursor vertikal menyapu gelombang waveform dengan mulus (*60 FPS*).

### 6.2 Checklist Halaman Spesifik
1. **Halaman Analisis:**
   - [ ] Drag-and-drop file menampilkan pendaran cyan aktif.
   - [ ] Perekaman mikrofon menampilkan VU-meter dinamis bergerak saat ada suara.
   - [ ] Mengklik sampel audio bawaan langsung memulai analisis tanpa jeda.
   - [ ] Banner peringatan kuning muncul jika confidence $< 65\%$.
2. **Halaman Riwayat:**
   - [ ] Data sesi baru langsung muncul di tabel setelah inferensi selesai.
   - [ ] Filter tab (Semua, Berisik, Non-Berisik) menyaring baris secara akurat.
   - [ ] Tombol "Ekspor CSV" mengunduh file `.csv` dengan header kolom lengkap.
3. **Halaman Bandingkan:**
   - [ ] Mengunggah 2 file menampilkan perbandingan side-by-side dan selisih persentase RMS.
4. **Halaman Informasi (Tentang Model & Batasan):**
   - [ ] Seluruh metrik ilmiah (85% akurasi, F1 0.85, 17/3 confusion matrix) tercantum tanpa salah ketik.

---

## 7. Uji Aksesibilitas (WCAG 2.1 AA) & Audit Lighthouse

1. **Audit Kontras Warna:**
   - Seluruh teks `#E6EDF3` di atas `#0B0F14` dan `#121922` memenuhi standar minimal 4.5:1 untuk teks normal (hasil audit: > 6:1).
2. **Navigasi Tab Keyboard:**
   - Pengguna dapat mengoperasikan seluruh fitur utama (memilih file, menekan tombol rekam, memutar audio) murni menggunakan tombol `Tab`, `Enter`, dan `Space`.
   - Cincin fokus ungu (`#8B5CF6`) terlihat jelas saat elemen aktif difokuskan.
3. **Skor Lighthouse Target:**
   - **Performance:** $\ge 90$
   - **Accessibility:** $\ge 95$
   - **Best Practices:** $\ge 95$
   - **SEO:** $\ge 90$

---

## 8. Uji Performa, Latensi, & Manajemen Memori

- **Tolak Ukur Latensi Inferensi:**
  - Audio durasi 10 detik: Waktu pemrosesan $\le 3.5$ detik.
  - Audio durasi 30 detik: Waktu pemrosesan $\le 7.0$ detik (Batas toleransi maksimal: 10 detik).
- **Penggunaan Memori RAM Server:**
  - Baseline saat startup (Model loaded): $\sim 180$ MB.
  - Peak RAM saat ekstraksi librosa berlangsung: $\le 450$ MB.
  - Garbage collection berhasil membebaskan memori kembali ke baseline setelah request selesai.
- **Uji Beban Konkurensi:**
  - Menjalankan 5 request inferensi simultan; FastAPI threadpool berhasil menyelesaikan seluruh permintaan tanpa kegagalan koneksi (*zero dropped requests*).

---

## 9. Matriks Pengujian Sistem (Test Matrix)

| ID Uji | Modul yang Diuji | Deskripsi & Prosedur Pengujian | Hasil yang Diharapkan | Status Kelulusan |
| :---: | :--- | :--- | :--- | :---: |
| **TC-01** | Backend DSP | Unggah sampel `24-003 berisik (1).mp3` | Prediksi `'berisik'`, confidence $\approx 89\% \pm 5\%$, RMS $\approx 0.0406 \pm 6\%$. | WAJIB LOLOS |
| **TC-02** | Backend DSP | Unggah sampel `24-003 sunyi 2.mp3` | Prediksi `'non-berisik'`, confidence $\approx 92\% \pm 5\%$, RMS $\approx 0.0209 \pm 6\%$. | WAJIB LOLOS |
| **TC-03** | Model RF | Validasi urutan 10 kolom fitur | Urutan kolom persis identik dengan `selected_features` pada artefak pkl. | WAJIB LOLOS |
| **TC-04** | Validasi API | Unggah file dummy ukuran 16 MB | HTTP 413 Payload Too Large dengan pesan Bahasa Indonesia. | WAJIB LOLOS |
| **TC-05** | Validasi API | Unggah file audio durasi 130 detik | HTTP 422 Durasi melebihi 120 detik. | WAJIB LOLOS |
| **TC-06** | Validasi API | Unggah file ekstensi `.exe` | HTTP 415 Format file tidak didukung. | WAJIB LOLOS |
| **TC-07** | Privasi Disk | Periksa direktori `/tmp/` setelah inferensi | File temporer telah dihapus bersih (`os.path.exists == False`). | WAJIB LOLOS |
| **TC-08** | Web Audio | Rekam mikrofon selama 7 detik | Terkirim ke API, inferensi berhasil, VU-meter reaktif. | WAJIB LOLOS |
| **TC-09** | Database | Verifikasi catatan baru di Supabase | Row baru tersimpan di `analysis_history` dengan session_id valid. | WAJIB LOLOS |
| **TC-10** | UI Responsif | Buka aplikasi pada viewport 360x640px | Tampilan teradaptasi satu kolom tanpa elemen bertumpuk/pecah. | WAJIB LOLOS |
