# Panduan Prompt Eksekusi AI Koding (AI Coding Prompts)
## Sound Lab: Copy-Paste Prompts for Autonomous AI Developers

> **Ringkasan Eksekutif:**  
> Dokumen ini memuat kumpulan perintah (*prompt*) siap pakai (*copy-paste ready*) yang dirancang khusus untuk memandu asisten koding AI (seperti Cursor, Windsurf, Claude Code, GitHub Copilot, atau v0.dev) dalam mengeksekusi pembangunan platform *Sound Lab* per fase. Setiap prompt bersifat mandiri (*self-contained*), menyertakan referensi dokumen spesifikasi, batasan teknis ketat, kriteria penerimaan, dan prosedur verifikasi otomatis.

---

## Daftar Isi
1. [Prinsip Penggunaan Prompt](#1-prinsip-penggunaan-prompt)
2. [Prompt Khusus UI Generator / V0.dev (Berdasarkan `desain.md`)](#2-prompt-khusus-ui-generator--v0dev-berdasarkan-desainmd)
3. [Prompt 0: Inisialisasi Proyek & Penyiapan Repositori](#3-prompt-0-inisialisasi-proyek--penyiapan-repositori)
4. [Prompt 1: Backend FastAPI, Pipeline DSP Librosa, & Uji Paritas](#4-prompt-1-backend-fastapi-pipeline-dsp-librosa--uji-paritas)
5. [Prompt 2: UI Inti Next.js, Dropzone, Recorder Web Audio, & Supabase](#5-prompt-2-ui-inti-nextjs-dropzone-recorder-web-audio--supabase)
6. [Prompt 3: Visualisasi Waveform Canvas, Spektrogram Inferno, & Chunk Timeline](#6-prompt-3-visualisasi-waveform-canvas-spektrogram-inferno--chunk-timeline)
7. [Prompt 4: Halaman Riwayat Sesi (Supabase) & Komparasi Dua Audio](#7-prompt-4-halaman-riwayat-sesi-supabase--komparasi-dua-audio)
8. [Prompt 5: Halaman Dokumentasi Ilmiah, Metrik Model, & Etika AI](#8-prompt-5-halaman-dokumentasi-ilmiah-metrik-model--etika-ai)
9. [Prompt 6: Audit Kinerja, Penanganan Cold-Start, & Panduan Deployment](#9-prompt-6-audit-kinerja-penanganan-cold-start--panduan-deployment)
10. [Prompt 7: Final Code Review, Refactoring, & Verifikasi Bug](#10-prompt-7-final-code-review-refactoring--verifikasi-bug)

---

## 1. Prinsip Penggunaan Prompt

1. **Jalankan Berurutan:** Eksekusi prompt sesuai urutan nomor (mulai dari Prompt 0 hingga Prompt 7).
2. **Jangan Mengubah Parameter Kunci:** Model Machine Learning menggunakan dependensi sensitif (`scikit-learn==1.8.0`) dan urutan 10 fitur ekstraksi yang tidak boleh ditukar posisinya.
3. **Verifikasi Mandiri:** Setiap prompt menyertakan perintah terminal atau uji peramban untuk memastikan tugas selesai sebelum beralih ke prompt berikutnya.

---

## 2. Prompt Khusus UI Generator / V0.dev (Berdasarkan `desain.md`)

Gunakan prompt ini pada generator UI (seperti v0.dev, Lovable, atau Bolt) untuk menghasilkan komponen antarmuka yang presisi:

```text
Buat satu halaman web aplikasi instrumen ilmiah modern bernama "Sound Lab" (Environmental Acoustic Classifier) menggunakan Next.js (App Router), React, Tailwind CSS, dan Lucide Icons.

TEMA & ESTETIKA ("Sound Lab"):
- Gaya: Dark mode scientific dashboard menyerupai instrumen audio laboratorium / osiloskop presisi. Serius, presisi, data-first.
- Palet Warna:
  * Latar dasar: #0B0F14
  * Permukaan kartu instrumen: #121922
  * Garis border: 1px solid #1F2A37
  * Teks utama: #E6EDF3
  * Teks sekunder: #8B98A9
  * Aksen BERISIK (noisy): #FF6B35 (Amber-Oranye)
  * Aksen NON-BERISIK (quiet): #22D3C5 (Cyan-Teal)
  * Aksen Netral/Aksi: #8B5CF6 (Ungu)
- Tipografi: Headings "Space Grotesk" 600, Body "Inter" 400/500, Nilai numerik/data teknis "JetBrains Mono".

STRUKTUR TATA LETAK (12 Kolom Desktop + Sidebar Sisi Kiri):
1. SIDEBAR KIRI: Lebar 240px, Logo Sound Lab berdenyut halus, navigasi (Analisis, Riwayat, Bandingkan, Tentang Model, Batasan & Etika).
2. HEADER UTAMA: Judul "Klasifikasi Akustik Lingkungan (Berisik vs Non-Berisik)" dengan latar visual animasi gelombang halus dan indikator status backend hijau online.
3. INPUT SECTION:
   - Kolom kiri: File Dropzone (drag & drop berkas .mp3, .wav, maks 15MB) dengan efek border cyan menyala saat di-drag.
   - Kolom kanan: Perekam Mikrofon Langsung dengan tombol rekam bulat merah berdenyut halus, timer 00:15 / 00:30, dan VU-meter audio vertikal bertingkat bergerak dinamis.
   - Presets Cepat: 4 tombol contoh audio bawaan ("Contoh Berisik 1", "Contoh Berisik 2", "Contoh Sunyi", "Audio Ambigu").
4. RESULT INSTRUMENT PANEL (State Hasil Analisis):
   - Kartu Diagnosa Utama: Badge status pill besar (BERISIK dengan ikon Volume2 atau NON-BERISIK dengan ikon VolumeX), semi-circular gauge confidence score (0-100%), dan rincian probabilitas 2 kelas.
   - Penjelasan Bahasa Awam: Kotak narasi ramah yang menerangkan alasan hasil berdasarkan energi suara (RMS).
   - Audio Player terintegrasi Waveform: Kanvas gelombang suara dengan garis kursor vertikal cyan bercahaya yang menyapu saat tombol play ditekan.
   - Timeline Prediksi Per-Chunk: Baris balok 3 detik berkode warna (oranye/cyan) dengan tag tegas: "INDIKATIF: Estimasi Segmen Lokal".
5. DETAIL SPEKTRUM & FITUR:
   - Spektrogram frekuensi berskala "inferno" dengan tombol modal pembesar.
   - Grafik batang interaktif membandingkan 10 fitur terpilih (rms_mean, mfcc1-13, zcr_mean, centroid_mean) terhadap rata-rata data latih.

Pastikan kode modular, bersih, responsif dari layar ponsel (360px) hingga monitor desktop (1440px), dan seluruh teks antarmuka menggunakan Bahasa Indonesia formal.
```

---

## 3. Prompt 0: Inisialisasi Proyek & Penyiapan Repositori

```text
Kamu adalah system architect senior. Tugasmu adalah menyiapkan fondasi struktur repositori dan dependensi proyek "Sound Lab" untuk sistem klasifikasi audio lingkungan.

DOKUMEN ACUAN:
- Baca [arsitektur.md](docs/arsitektur.md) bagian Struktur Folder dan Pemilihan Teknologi.

TUGAS UTAMA:
1. Buat struktur folder proyek berikut secara bersih:
   - `backend/app/api`, `backend/app/core`, `backend/app/schemas`, `backend/tests`
   - `frontend/src/app`, `frontend/src/components/ui`, `frontend/src/lib`
   - `data/`, `samples/`, `docs/`
2. Di folder `backend/`, buat berkas `requirements.txt` dengan dependensi terkunci (PINNED):
   - fastapi>=0.111.0
   - uvicorn[standard]>=0.30.0
   - scikit-learn==1.8.0  # WAJIB TEPAT 1.8.0!
   - librosa==0.10.2
   - soundfile>=0.12.1
   - joblib>=1.4.2
   - pandas>=2.2.0
   - numpy>=1.26.0,<2.0.0
   - slowapi>=0.1.9
   - python-multipart>=0.0.9
   - pytest>=8.0.0
   - httpx>=0.27.0
3. Di folder `frontend/`, inisialisasi aplikasi Next.js (App Router, TypeScript, Tailwind CSS, Lucide React).
4. Buat berkas konfigurasi lingkungan contoh:
   - `backend/.env.example` (PORT=8000, MODEL_PATH=../data/model_klasifikasi_audio.pkl)
   - `frontend/.env.local.example` (NEXT_PUBLIC_API_URL=http://localhost:8000, NEXT_PUBLIC_SUPABASE_URL=..., NEXT_PUBLIC_SUPABASE_ANON_KEY=...)
5. Buat file SQL `data/supabase_schema.sql` untuk membuat tabel `analysis_history` sesuai spesifikasi pada arsitektur.md.

BATASAN:
- Jangan menulis kode endpoint atau antarmuka rumit dulu di tahap ini. Fokus murni pada penyiapan struktur direktori dan paket.
- Versi `scikit-learn` WAJIB `1.8.0`.

VERIFIKASI:
Jalankan di terminal:
`python -c "import sklearn; assert sklearn.__version__ == '1.8.0'"` dan pastikan tidak ada error.
```

---

## 4. Prompt 1: Backend FastAPI, Pipeline DSP Librosa, & Uji Paritas

```text
Kamu adalah AI engineer dan DSP specialist. Tugasmu adalah membangun mesin inferensi backend FastAPI dan memvalidasi paritas fitur audio terhadap data notebook eksperimen.

DOKUMEN ACUAN:
- [model-spec.md](docs/model-spec.md) (Sangat penting: Pipeline Ekstraksi Fitur & Split vs Trim)
- [api-spec.md](docs/api-spec.md) (Kontrak endpoint RESTful)
- [testing.md](docs/testing.md) (Nilai paritas data notebook)

TUGAS UTAMA:
1. Di `backend/app/core/model_loader.py`, buat pemuat model biner `data/model_klasifikasi_audio.pkl` menggunakan `joblib.load()`.
   - Simpan objek model, target_sr (22050), top_db (25), dan chunk_duration (3) ke dalam state singleton aplikasi FastAPI menggunakan lifespan context manager di `main.py`.
   - Ambil daftar kelas secara dinamis dari `model.classes_`.
2. Di `backend/app/core/audio_pipeline.py`, implementasikan fungsi ekstraksi sinyal:
   a. `librosa.load(file, sr=22050, mono=True)`
   b. Buang hening dengan `librosa.effects.split(y, top_db=25)` lalu sambungkan semua interval dengan `np.concatenate`. (JANGAN pakai effects.trim!).
   c. Segmentasi chunk 3 detik (66.150 sampel). Jika audio < 66.150 sampel, zero-pad ke 66.150 sampel.
   d. Hitung per-chunk: RMS, spectral centroid, ZCR, MFCC 1-13.
   e. Gabungkan secara horizontal (hstack), lalu agregasi statistik mean dan std.
   f. Bentuk DataFrame 1 baris dengan 10 fitur urutan wajib:
      `['rms_mean', 'mfcc1_mean', 'rms_std', 'mfcc3_mean', 'zcr_mean', 'mfcc6_mean', 'mfcc7_mean', 'mfcc13_mean', 'mfcc5_mean', 'centroid_mean']`.
   g. Jalankan `model.predict()` dan `model.predict_proba()`.
3. Di `backend/app/api/routes.py`, bangun endpoint:
   - `POST /api/predict`: Menerima multipart `file`, simpan sementara di `/tmp/`, eksekusi pipeline via `asyncio.to_thread()`, dan hapus file sementara pada blok `finally`. Kembalikan skema JSON sesuai `api-spec.md`.
   - `GET /api/model-info`: Kembalikan metrik evaluasi model (85% akurasi, dsb).
   - `GET /api/health`: Kembalikan status server dan pemakaian memori.
4. Buat unit test paritas di `backend/tests/test_pipeline_parity.py` untuk menguji file sampel:
   - `24-003 berisik (1).mp3` -> label 'berisik', RMS ~0.0406 (toleransi 6%)
   - `24-003 sunyi 2.mp3` -> label 'non-berisik', RMS ~0.0209 (toleransi 6%)

BATASAN KETAT:
- Urutan 10 fitur TIDAK BOLEH diubah sedikitpun.
- Pastikan berkas temporer selalu dihapus di blok `finally` (Kebijakan Zero Raw-Audio Retention).
- Tangani error dengan pesan Bahasa Indonesia sesuai tabel error di api-spec.md.

VERIFIKASI:
Jalankan `pytest backend/tests/test_pipeline_parity.py` dan pastikan seluruh pengujian lulus dengan status hijau.
```

---

## 5. Prompt 2: UI Inti Next.js, Dropzone, Recorder Web Audio, & Supabase

```text
Kamu adalah frontend developer senior ahli Next.js dan Tailwind CSS. Tugasmu adalah membangun antarmuka utama "Sound Lab", penanganan input audio, dan integrasi penyimpanan riwayat ke Supabase.

DOKUMEN ACUAN:
- [desain.md](docs/desain.md) (Token warna, tipografi, wireframe Section 1 & Section 2)
- [PRD.md](docs/PRD.md) (Fitur 1, 2, 3, 5, dan 11)
- [api-spec.md](docs/api-spec.md) (Format request dan response)

TUGAS UTAMA:
1. Konfigurasi `tailwind.config.ts` dengan warna Sound Lab:
   - `bg-base`: `#0B0F14`
   - `surface-card`: `#121922`
   - `border-subtle`: `#1F2A37`
   - `class-noisy`: `#FF6B35`
   - `class-quiet`: `#22D3C5`
   - `accent-purple`: `#8B5CF6`
   Pasang font Google: Space Grotesk, Inter, dan JetBrains Mono.
2. Buat komponen layout:
   - Sidebar navigasi kiri (`Sidebar.tsx`) dengan link: Analisis, Riwayat, Bandingkan, Tentang Model, Batasan & Etika.
   - Header aplikasi dengan status konektivitas backend online/offline.
3. Buat komponen input audio (`src/components/analysis/`):
   - `FileDropzone.tsx`: Area seret file (.mp3, .wav, .ogg, .flac, .m4a maks 15MB) dengan efek border pendaran cyan.
   - `AudioRecorder.tsx`: Modul perekam Web Audio API (MediaRecorder), timer 5-30 detik, tombol bundar merah berdenyut halus, dan canvas VU-meter vertikal live.
   - `QuickPresets.tsx`: Tombol pilihan cepat untuk 4 sampel audio bawaan.
4. Buat kartu hasil instrumen (`ResultCard.tsx`):
   - Badge status pill: `BERISIK` (ikon Volume2, warna #FF6B35) atau `NON-BERISIK` (ikon VolumeX, warna #22D3C5).
   - Semi-circular gauge meter untuk skor *confidence* dengan animasi counter angka (0% -> target% dalam 800ms).
   - Rincian persentase probabilitas 2 kelas.
5. Di `src/lib/supabase.ts`, inisialisasi `@supabase/supabase-js`. Buat fungsi untuk mencatat riwayat analisis ke tabel `analysis_history` setelah menerima respons sukses dari API backend.

BATASAN:
- Tampilan harus responsif (Mobile 360px, Tablet 768px, Desktop 1280px).
- Seluruh teks antarmuka dan pesan validasi wajib berbahasa Indonesia.
- Jangan simpan file rekaman biner ke Supabase; hanya simpan metadata numerik dan label!

VERIFIKASI:
Jalankan `npm run dev`, lakukan drag file audio atau klik rekam, dan pastikan request terkirim ke backend serta kartu hasil merender diagnosis dengan benar.
```

---

## 6. Prompt 3: Visualisasi Waveform Canvas, Spektrogram Inferno, & Chunk Timeline

```text
Kamu adalah UI/UX engineer spesialis visualisasi data audio di web. Tugasmu adalah membangun visualisasi interaktif pada halaman analisis Sound Lab.

DOKUMEN ACUAN:
- [desain.md](docs/desain.md) (Elemen Khas: Oscilloscope Waveform Sweep, Spektrogram Inferno, Timeline Chunk)
- [model-spec.md](docs/model-spec.md) (Spesifikasi chunk 3 detik & narasi explainability)

TUGAS UTAMA:
1. Buat pemutar audio terintegrasi waveform (`WaveformPlayer.tsx`):
   - Render bentuk gelombang audio menggunakan data `waveform_downsampled` dari backend pada elemen HTML5 `<canvas>`.
   - Tambahkan tombol Play/Pause dan label stempel waktu (contoh: `00:12 / 00:27`).
   - Implementasikan *Oscilloscope Waveform Sweep*: Kursor garis vertikal cyan bercahaya (`#22D3C5`, lebar 2px) yang menyapu gelombang audio secara sinkron saat pemutaran berlangsung.
   - Izinkan pengguna mengklik sembarang titik pada kanvas gelombang untuk berpindah posisi waktu (*seeking*).
2. Buat visualisasi timeline chunk (`ChunkTimeline.tsx`):
   - Render barisan kotak 3 detik dari array `chunk_timeline`.
   - Kotak berwarna oranye (`#FF6B35`) untuk chunk berisik dan cyan (`#22D3C5`) untuk chunk sunyi.
   - Cantumkan label peringatan tegas di atasnya: *"INDIKATIF (Estimasi segmen lokal - Model dilatih pada agregat 30 detik)"*.
   - Tambahkan tooltip interaktif saat hover yang menampilkan detik mulai-selesai dan confidence chunk.
3. Buat tampilan spektrogram (`SpectrogramViewer.tsx`):
   - Tampilkan gambar spektrogram frekuensi hasil generate backend (colormap inferno).
   - Sediakan tombol modal untuk memperbesar spektrogram ke resolusi penuh beserta skala sumbu frekuensi (Hz).
4. Buat grafik perbandingan fitur (`FeatureComparisonBar.tsx`):
   - Tampilkan 10 fitur audio yang diuji berdampingan dengan rata-rata data latih kelas berisik dan non-berisik (dari `class_comparison`).
5. Buat panel narasi awam (`ExplainabilityCard.tsx`):
   - Tampilkan teks `plain_explanation` dari backend.
   - Jika `is_low_confidence` bernilai `true` (skor < 65%), tampilkan banner peringatan bergaris tepi kuning dengan rekomendasi pengujian ulang.

BATASAN:
- Animasi kursor waveform harus berjalan mulus (60 FPS via `requestAnimationFrame`).
- Dukung aturan CSS `@media (prefers-reduced-motion)` untuk menonaktifkan efek pendaran jika diinginkan pengguna.

VERIFIKASI:
Putar berkas audio hasil analisis dan amati pergerakan kursor vertikal menyapu waveform serta verifikasi tooltip timeline per-chunk saat kursor mouse diarahkan ke tiap balok.
```

---

## 7. Prompt 4: Halaman Riwayat Sesi (Supabase) & Komparasi Dua Audio

```text
Kamu adalah full-stack developer. Tugasmu adalah membangun Halaman Riwayat Sesi dan Halaman Komparasi Dua Audio pada platform Sound Lab.

DOKUMEN ACUAN:
- [desain.md](docs/desain.md) (Wireframe Halaman 9.2 Riwayat dan 9.3 Bandingkan)
- [PRD.md](docs/PRD.md) (Fitur 10 Riwayat + Ekspor CSV, Fitur 11 Komparasi)

TUGAS UTAMA:
1. Buat halaman `/riwayat/page.tsx`:
   - Ambil data dari tabel Supabase `analysis_history` yang sesuai dengan `session_id` pengguna saat ini.
   - Sediakan filter tab: "Semua", "Berisik", dan "Non-Berisik".
   - Tampilkan tabel dengan kolom: Waktu, Nama Berkas, Durasi, Prediksi (Badge Pill), Confidence, RMS Mean, dan Tombol Aksi Detail.
   - Sediakan tombol "Ekspor CSV" yang menghasilkan dan mengunduh berkas `.csv` berisi seluruh metadata dan 10 fitur pengujian sesi.
   - Tangani empty state jika belum ada data analisis pada sesi tersebut.
2. Buat halaman `/bandingkan/page.tsx`:
   - Sediakan dua panel masukan audio berdampingan: "Audio A" dan "Audio B" (bisa upload file atau memilih dari preset).
   - Jalankan analisis untuk kedua berkas ke endpoint backend.
   - Tampilkan kartu komparasi: Status hasil, perbandingan selisih RMS (dalam persentase), dan selisih Spectral Centroid (Hz).
   - Tampilkan grafik radar atau bar overlay yang membandingkan 10 fitur Audio A vs Audio B secara visual.

BATASAN:
- Pastikan tombol Ekspor CSV memformat angka desimal dan string dengan pemisah koma/titik-koma yang kompatibel dengan Microsoft Excel dan Google Sheets.
- Desain tabel harus responsif (bisa di-scroll horizontal pada layar ponsel tanpa merusak layout).

VERIFIKASI:
Lakukan 2 kali analisis audio, buka halaman Riwayat, verifikasi kedua data muncul di tabel, lalu klik tombol "Ekspor CSV" dan periksa integritas berkas CSV yang terunduh.
```

---

## 8. Prompt 5: Halaman Dokumentasi Ilmiah, Metrik Model, & Etika AI

```text
Kamu adalah technical writer dan frontend engineer. Tugasmu adalah mengimplementasikan Halaman "Tentang Model" dan Halaman "Batasan & Etika" untuk keperluan transparansi ilmiah pengujian dosen PSD.

DOKUMEN ACUAN:
- [desain.md](docs/desain.md) (Spesifikasi Halaman 9.4 dan 9.5)
- [model-spec.md](docs/model-spec.md) (Metrik evaluasi 85%, confusion matrix 17/3, ranking 6 model, 10 bobot fitur)

TUGAS UTAMA:
1. Buat halaman `/tentang-model/page.tsx`:
   - Ambil data metodologi dari endpoint `GET /api/model-info` (atau render data statis jika server offline).
   - Tampilkan kartu arsitektur model: Random Forest Classifier (100 trees, max_depth=10, scikit-learn 1.8.0).
   - Tampilkan tabel metrik evaluasi utama:
     * Akurasi Pengujian: 85.00%
     * Presisi: 0.85, Recall: 0.85, F1-Score: 0.85
     * ROC-AUC: 0.8688
     * 5-Fold Cross Validation: 84.15% (+/- 2.94%)
   - Tampilkan visualisasi Matriks Konfusi (Confusion Matrix Data Uji 40 Sampel): 17 benar dan 3 salah untuk kelas Berisik dan Non-Berisik.
   - Tampilkan tabel pemeringkatan 6 model (Random Forest 85%, Gradient Boosting 80%, SVM RBF 77.5%, SVM Linear 72.5%, Logistic Regression 70%, KNN 70%).
   - Tampilkan tabel 10 fitur terpilih beserta bobot importance MDI-nya.
2. Buat halaman `/batasan-etika/page.tsx`:
   - Bagian 1: Distribusi Dataset (~200 sampel audio rekaman kampus durasi 30 detik).
   - Bagian 2: Suara di Luar Distribusi (OOD) - Penjelasan mengapa model dapat kurang akurat pada suara musik ber-bass keras, hujan deras terus menerus, atau turbin pabrik.
   - Bagian 3: Komitmen Privasi Total (Zero Raw-Audio Retention) - Penegasan bahwa tidak ada rekaman audio pengguna yang disimpan di server atau database cloud.
   - Bagian 4: Lisensi Akademik dan Batasan Tanggung Jawab.

BATASAN:
- Semua angka metrik ilmiah harus 100% konsisten dengan laporan proyek sains data (tidak boleh ada data fiktif).
- Gunakan tipografi yang bersih, hierarki heading jelas, dan tabel data yang terstruktur rapi.

VERIFIKASI:
Buka kedua halaman di peramban, pastikan seluruh data tabel terbaca jelas dan tidak ada tautan navigasi yang rusak.
```

---

## 9. Prompt 6: Audit Kinerja, Penanganan Cold-Start, & Panduan Deployment

```text
Kamu adalah DevOps dan performance engineer. Tugasmu adalah mengoptimasi performa produksi Sound Lab, menangani cold start backend gratis, dan menyusun konfigurasi deployment online.

DOKUMEN ACUAN:
- [arsitektur.md](docs/arsitektur.md) (Rencana Deployment, Analisis Biaya, dan Penanganan Cold-Start)
- [testing.md](docs/testing.md) (Uji Performa & Lighthouse)

TUGAS UTAMA:
1. Di frontend Next.js, buat komponen pemantau kesiapan backend (`BackendHealthMonitor.tsx`):
   - Lakukan fetch ke `GET /api/health` saat aplikasi pertama kali dimuat.
   - Jika endpoint belum merespons dalam 3 detik (indikasi cold start di platform gratis seperti Render), tampilkan banner informatif:
     *"Sedang menghubungkan ke mesin audio server (~30-45 detik untuk pemanasan awal)..."* dengan indikator loading berdenyut halus.
   - Saat server sudah merespons status healthy, ubah badge menjadi hijau dan sembunyikan banner otomatis.
2. Buat `backend/Dockerfile` yang optimal untuk deployment:
   - Base image: `python:3.10-slim`
   - Install dependensi sistem: `ffmpeg`, `libsndfile1`, dan `build-essential`.
   - Salin requirements.txt dan install paket Python.
   - Salin kode aplikasi dan berkas model.
   - Jalankan dengan `CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]`.
3. Buat berkas `README.md` di root repositori:
   - Judul, badge teknologi, dan deskripsi ringkas.
   - Panduan menjalankan secara lokal (Prasyarat: Python 3.10+, Node.js 18+, FFmpeg).
   - Langkah konfigurasi `.env` backend dan frontend.
   - Petunjuk deployment ke Vercel (frontend) dan Render/Hugging Face Spaces (backend).

BATASAN:
- Pastikan container Docker tidak menyimpan cache apt berlebih untuk menjaga ukuran image seminimal mungkin.
- Pastikan CORS di backend mengizinkan domain publik Vercel yang akan digunakan.

VERIFIKASI:
Bangun image docker secara lokal atau uji perintah run guide pada README untuk memastikan instalasi dari nol berhasil tanpa kendala.
```

---

## 10. Prompt 7: Final Code Review, Refactoring, & Verifikasi Bug

```text
Kamu adalah Lead QA & Principal Code Reviewer. Tugasmu adalah melakukan audit menyeluruh terhadap seluruh codebase Sound Lab sebelum diserahkan untuk demonstrasi penilaian mata kuliah PSD.

DOKUMEN ACUAN:
- [testing.md](docs/testing.md) (Matriks Pengujian Sistem TC-01 s.d. TC-10)
- [PRD.md](docs/PRD.md) (Acceptance Criteria)

TUGAS AUDIT:
1. Audit Integritas Fitur PSD:
   - Periksa kembali fungsi ekstraksi librosa di backend. Pastikan `librosa.effects.split` digunakan dan `trim` tidak lagi dipakai.
   - Pastikan nama dan urutan 10 kolom fitur tidak pernah berubah di seluruh file.
2. Audit Privasi & Pembersihan Memori:
   - Periksa endpoint `/api/predict`. Pastikan `os.unlink()` selalu berada di blok `finally` dan dipanggil untuk setiap file temporer.
   - Pastikan tidak ada berkas audio `.mp3` atau `.wav` milik pengguna yang tersimpan permanen di disk.
3. Audit Responsivitas & Aksesibilitas UI:
   - Periksa kontras warna elemen teks dan badge kelas (#FF6B35 dan #22D3C5).
   - Pastikan tombol audio player memiliki atribut `aria-label`.
   - Uji tampilan pada ukuran layar 360px (mobile) untuk memastikan tidak ada horizontal scrollbar yang tidak diinginkan.
4. Audit Paritas & Error Handling:
   - Uji pengiriman file kosong 0 byte -> pastikan muncul HTTP 400 dengan pesan Indonesia.
   - Uji pengiriman file non-audio -> pastikan muncul HTTP 415.
   - Uji pengiriman file audio > 15 MB -> pastikan ditolak dengan pesan yang ramah.

LAPORAN:
Tuliskan rekapitulasi audit dalam format checklist: status kelulusan tiap kriteria, daftar bug yang berhasil diperbaiki, dan konfirmasi bahwa seluruh sistem siap untuk demonstrasi dosen penguji.
```
