# Panduan Desain & Sistem Desain (Design System)
## Sound Lab: Scientific Audio Analysis & Oscilloscope Dashboard

> **Ringkasan Eksekutif:**  
> Dokumen ini adalah spesifikasi desain sistem dan antarmuka pengguna untuk platform *Sound Lab*. Mengusung estetika *dark mode scientific instrument* menyerupai perangkat osiloskop dan stasiun riset akustik digital, panduan ini memuat token warna, tipografi, grid, komponen presisi, wireframe ASCII setiap halaman, serta pedoman aksesibilitas dan animasi terukur yang siap diimplementasikan langsung pada Next.js atau perangkat desain UI modern.

---

## Daftar Isi
1. [Filosofi Desain & Moodboard](#1-filosofi-desain--moodboard)
2. [Token Warna (Color Tokens)](#2-token-warna-color-tokens)
3. [Tipografi & Skala Teks](#3-tipografi--skala-teks)
4. [Sistem Spasi, Grid, & Breakpoints](#4-sistem-spasi-grid--breakpoints)
5. [Elevasi, Radius, & Batas (Borders)](#5-elevasi-radius--batas-borders)
6. [Prinsip Motion, Transisi, & Aksesibilitas Sensorik](#6-prinsip-motion-transisi--aksesibilitas-sensorik)
7. [Elemen Khas (Signature Elements)](#7-elemen-khas-signature-elements)
8. [Katalog Komponen UI](#8-katalog-komponen-ui)
9. [Spesifikasi Halaman & Wireframe ASCII](#9-spesifikasi-halaman--wireframe-ascii)
   - 9.1 [Halaman Analisis (Utama)](#91-halaman-analisis-utama)
   - 9.2 [Halaman Riwayat](#92-halaman-riwayat)
   - 9.3 [Halaman Bandingkan (Side-by-Side)](#93-halaman-bandingkan-side-by-side)
   - 9.4 [Halaman Tentang Model](#94-halaman-tentang-model)
   - 9.5 [Halaman Batasan & Etika](#95-halaman-batasan--etika)
10. [Aksesibilitas (WCAG 2.1 AA) & Validasi Kontras](#10-aksesibilitas-wcag-21-aa--validasi-kontras)
11. [Panduan Visualisasi Data Akustik](#11-panduan-visualisasi-data-akustik)

---

## 1. Filosofi Desain & Moodboard

### 1.1 Konsep: "Sound Lab"
Desain mengadopsi metafora **Laboratorium Akustik Presisi Digital**. Tidak seperti antarmuka SaaS konvensional yang datar atau terlalu korporat, *Sound Lab* memancarkan nuansa perangkat instrumen laboratorium fisika/sinyal (seperti osiloskop, spektrum analizer, dan konsol rekaman studio). 

### 1.2 Pilar Pengalaman Pengguna
1. **Presisi Ilmiah (Scientific Rigor):** Tata letak berbasis data, penggunaan grid terkalibrasi, angka monospaced berpresisi tinggi, dan grafik tanpa ornamen sia-sia.
2. **Keterbacaan Gelap yang Tenang (Subdued Dark Elegance):** Latar belakang hitam kebiruan pekat yang mengurangi ketegangan mata saat meneliti sinyal frekuensi tinggi dalam jangka panjang.
3. **Kontras Semantik Tegas:** Warna aksen fungsional hanya digunakan untuk menyampaikan makna data: oranye-amber untuk kebisingan energetik, cyan-teal untuk kesunyian tenang, dan ungu netral untuk interaktivitas instrumen.

---

## 2. Token Warna (Color Tokens)

Semua token warna terdefinisi secara terpusat dan konsisten di seluruh antarmuka.

### 2.1 Palet Utama (Core Palette)
| Token Nama | Hex Code | Deskripsi & Peruntukan Fungsional |
| :--- | :--- | :--- |
| `color-bg-base` | `#0B0F14` | Latar belakang dasar aplikasi (deep obsidian void). |
| `color-surface-card` | `#121922` | Permukaan kartu instrumen, panel kontrol, dan wadah data. |
| `color-surface-elevated`| `#182230` | Permukaan modal dialog, dropdown popover, dan panel melayang. |
| `color-border-subtle` | `#1F2A37` | Garis tepi batas kartu (1px border), pemisah tabel, dan garis grid plot. |
| `color-border-hover` | `#374151` | Garis batas komponen saat disentuh kursor (*hover state*). |
| `color-text-primary` | `#E6EDF3` | Teks judul, angka metrik utama, dan label penting (kontras 14.8:1). |
| `color-text-secondary`| `#8B98A9` | Teks isi, label sumbu grafik, dan informasi pendukung (kontras 6.2:1). |
| `color-text-muted` | `#525F71` | Teks non-aktif, placeholder form, dan petunjuk kecil. |

### 2.2 Palet Semantik & Data (Semantic & Class Colors)
| Token Nama | Hex Code | Penerapan Semantik Data |
| :--- | :--- | :--- |
| `color-class-noisy` | `#FF6B35` | **BERISIK (Amber-Oranye):** Badge aktif, kurva RMS tinggi, segmen chunk bising. |
| `color-class-noisy-glow`| `rgba(255, 107, 53, 0.25)` | Efek pendaran (*glow shadow*) untuk kelas Berisik saat aktif. |
| `color-class-quiet` | `#22D3C5` | **NON-BERISIK (Cyan-Teal):** Badge aktif, kurva audio sunyi, status stabil. |
| `color-class-quiet-glow`| `rgba(34, 211, 197, 0.25)` | Efek pendaran (*glow shadow*) untuk kelas Non-Berisik saat aktif. |
| `color-accent-purple` | `#8B5CF6` | **Interaksi Netral (Ungu):** Tombol aksi sekunder, tab aktif, penanda waktu. |
| `color-warning-border` | `#F59E0B` | Border peringatan untuk hasil *confidence* rendah (< 65%). |
| `color-warning-bg` | `rgba(245, 158, 11, 0.08)`| Latar kartu peringatan ambang rendah. |
| `color-error-base` | `#EF4444` | Penolakan berkas korup, kesalahan format, atau kegagalan peladen. |
| `color-success-base` | `#10B981` | Notifikasi penyelesaian unggah berkas dan penyimpanan Supabase sukses. |

### 2.3 Colormap Spektrogram
- Spektrogram menggunakan pemetaan warna **`inferno`** (dari hitam pekat $\rightarrow$ ungu $\rightarrow$ merah-oranye $\rightarrow$ kuning cerah) untuk menggambarkan spektrum energi desibel (dB) secara linier terhadap persepsi mata manusia.

---

## 3. Tipografi & Skala Teks

Tipografi memadukan ketegasan judul modern, kenyamanan membaca teks instruksi, dan presisi pembacaan data instrumen.

### 3.1 Keluarga Font (Font Families)
1. **Headings:** `"Space Grotesk", sans-serif` (Bobot: 600 - SemiBold). Memberikan karakter futuristik dan ilmiah.
2. **Body & Interface:** `"Inter", sans-serif` (Bobot: 400 - Regular, 500 - Medium). Optimal untuk keterbacaan antarmuka di layar padat.
3. **Data, Metrik, & Kode:** `"JetBrains Mono", monospace` (Bobot: 500 - Medium, 700 - Bold). Menjaga perataan vertikal karakter angka pada pembacaan Hz, dB, persentase, dan stempel waktu.

### 3.2 Skala Tipografi
| Token Ukuran | Font Size | Line Height | Penggunaan Utama | Font Family |
| :--- | :--- | :--- | :--- | :--- |
| `text-display` | 44 px | 52 px (1.18) | Angka persentase besar pada *result gauge*. | JetBrains Mono (Bold) |
| `text-h1` | 28 px | 36 px (1.28) | Judul halaman utama, judul instrumen besar. | Space Grotesk (600) |
| `text-h2` | 20 px | 28 px (1.40) | Judul panel instrumen, nama kartu analisis. | Space Grotesk (600) |
| `text-base` | 16 px | 24 px (1.50) | Teks paragraf, penjelasan naratif hasil. | Inter (400) |
| `text-sub` | 14 px | 20 px (1.42) | Label tombol, item tabel riwayat, teks form. | Inter (500) |
| `text-mono-sm` | 12 px | 16 px (1.33) | Satuan frekuensi (Hz), nilai fitur, koordinat waktu.| JetBrains Mono (500) |

---

## 4. Sistem Spasi, Grid, & Breakpoints

### 4.1 Skala Spasi (Spacing Scale)
Mengikuti sistem modular 4px:
- `space-1` (4px), `space-2` (8px), `space-3` (12px), `space-4` (16px), `space-6` (24px), `space-8` (32px), `space-12` (48px), `space-16` (64px).

### 4.2 Breakpoints Responsif
- **Mobile (`< 768px`, Target referensi: 360px):** Tata letak 1 kolom linier vertikal. Sidebar dialihkan menjadi navigasi bilah bawah (*bottom bar*) atau menu drawer hamburger.
- **Tablet (`768px - 1279px`, Target referensi: 768px):** Tata letak 2 kolom modular. Sidebar ikon mengecil (*icon-only rail*).
- **Desktop (`>= 1280px`, Target referensi: 1440px):** Tata letak penuh 12 kolom grid dengan lebar kontainer maksimal 1400px, didampingi sidebar navigasi statis selebar 240px.

---

## 5. Elevasi, Radius, & Batas (Borders)

### 5.1 Radius Sudut
- `radius-sm` (6px): Badge pill kecil, tag filter, dan tooltip.
- `radius-md` (12px): **Standar Kartu Instrumen**, zona upload, jendela pemutar audio, dan modal dialog.
- `radius-full` (9999px): Tombol rekam bulat, indikator status, dan badge status kelas.

### 5.2 Batas & Bayangan (Borders & Shadows)
- Seluruh kartu menggunakan **`border: 1px solid #1F2A37`**.
- Tidak menggunakan bayangan buram berat (*drop shadow*) tradisional. Sebagai gantinya, digunakan pendaran tepi halus (*subtle inner/edge glow*):
  - Kartu Normal: `box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4)`
  - Kartu Aktif Berisik: `box-shadow: 0 0 15px rgba(255, 107, 53, 0.15), inset 0 0 1px rgba(255, 107, 53, 0.4)`
  - Kartu Aktif Non-Berisik: `box-shadow: 0 0 15px rgba(34, 211, 197, 0.15), inset 0 0 1px rgba(34, 211, 197, 0.4)`

---

## 6. Prinsip Motion, Transisi, & Aksesibilitas Sensorik

### 6.1 Karakter Gerakan
- **Durasi Standar:** `200ms` (interaksi tombol/hover) hingga `300ms` (animasi membuka modal/transisi panel).
- **Fungsi Easing:** `cubic-bezier(0.16, 1, 0.3, 1)` (*ease-out swift*), menghadirkan kesan mekanis presisi.
- **Counter Confidence:** Angka persentase keyakinan berhitung cepat dari 0% ke nilai target selama 800ms dengan akselerasi halus saat hasil pertama kali dimuat.

### 6.2 Aturan `prefers-reduced-motion`
- Bila pengguna mengaktifkan mode pembatasan gerak di sistem operasi:
  - Seluruh animasi denyut (*pulse*) tombol dinonaktifkan (diganti dengan border statis warna kontras).
  - Penghitungan angka dialihkan langsung ke angka final tanpa animasi.
  - Pergeseran garis kursor waveform tetap berjalan proporsional terhadap waktu audio, namun efek pendaran dinonaktifkan.

---

## 7. Elemen Khas (Signature Elements)

1. **Oscilloscope Waveform Sweep:** Garis kursor vertikal 2px berwarna cyan bercahaya (`#22D3C5`) menyapu bidang waveform audio secara mulus dari kiri ke kanan saat audio diputar.
2. **Denyut Tombol Rekam Ilmiah:** Tombol rekam bundar dengan lampu indikator merah di tengah yang berkedip halus (*rhythmic breath pulse* 1.5s) saat sesi mikrofon berlangsung.
3. **Instrumen Semi-Circular Gauge:** Busur derajat sudut 180° dengan jarum penunjuk digital yang mengunci pada nilai *confidence score*.
4. **VU-Meter RMS Real-Time:** Bar segmen vertikal bertingkat 12 balok (hijau pada level dasar, kuning pada tengah, merah pada level saturasi kebisingan) yang bergerak aktif merespons sinyal.

---

## 8. Katalog Komponen UI

### 8.1 Tombol (Buttons)
- **Primary Action (Analisis / Rekam):** Latar `#FF6B35` (mode bising/rekam) atau `#8B5CF6` (netral). Teks putih tegas, padding `10px 20px`, radius 8px, font Inter 500.
- **Secondary Action (Sampel / Reset):** Latar transparan, `border: 1px solid #1F2A37`, teks `#E6EDF3`. Hover: border `#374151`, latar `#182230`.
- **Disabled State:** Opasitas 40%, kursor `not-allowed`, latar `#121922`, border `#1F2A37`.

### 8.2 Badge Status Kelas (Result Pill Badges)
- **Varian BERISIK:**
  - Latar: `rgba(255, 107, 53, 0.12)`
  - Border: `1px solid #FF6B35`
  - Teks: `#FF6B35` (Font JetBrains Mono, 14px, Bold, Uppercase)
  - Ikon: `Volume2` (Speaker memancarkan gelombang suara penuh)
- **Varian NON-BERISIK:**
  - Latar: `rgba(34, 211, 197, 0.12)`
  - Border: `1px solid #22D3C5`
  - Teks: `#22D3C5` (Font JetBrains Mono, 14px, Bold, Uppercase)
  - Ikon: `VolumeX` (Speaker dalam kondisi teredam/senyap)

### 8.3 Zona Unggah Berkas (Upload Dropzone)
- Wadah persegi panjang tinggi 180px, garis putus-putus (`border: 2px dashed #1F2A37`).
- Ikon awan pengunggah (`UploadCloud`) di tengah, diiringi instruksi seret file dan keterangan format yang didukung (.mp3, .wav, .ogg, .flac, .m4a maks 15MB).
- **State Drag-Over:** Border berubah menjadi garis padat cyan `#22D3C5`, latar menjadi `rgba(34, 211, 197, 0.05)`, teks instruksi memancarkan pendaran lembut.

### 8.4 Timeline Prediksi Per-Chunk (3 Detik)
- Komponen batang horisontal yang terbagi menjadi kotak-kotak kecil bernomor (Chunk 1, Chunk 2, dst.).
- Warna kotak: Oranye `#FF6B35` (jika chunk diklasifikasikan berisik) dan Cyan `#22D3C5` (jika chunk sunyi).
- Banner peringatan di atas kotak: *Tag: INDIKATIF (Model dilatih pada agregat 30 detik)*.

---

## 9. Spesifikasi Halaman & Wireframe ASCII

### 9.1 Halaman Analisis (Utama)
Halaman sentral tempat pengguna memasukkan audio dan membaca seluruh diagnosis instrumen.

#### Wireframe ASCII Desktop (1280px+)
```text
+-------------------------------------------------------------------------------------------------------+
| [LOGO] SOUND LAB  :: Environmental Acoustic Classifier                 [Status: Backend Online (Green)]|
+-------------------+-----------------------------------------------------------------------------------+
| [NAV]             | HERO SECTION: Oscilloscope Waveform Line Animation Header                        |
| - Analisis (Active| "Uji Kebisingan Akustik Lingkungan Berbasis Random Forest & DSP"                     |
| - Riwayat         +-----------------------------------------------------------------------------------+
| - Bandingkan      | SECTION 1: INPUT PANEL                                                            |
| - Tentang Model   | +-----------------------------------------------+ +-------------------------------+ |
| - Batasan & Etika | | AREA UNGGAH BERKAS (DRAG & DROP)              | | REKAM MIKROFON LANGSUNG       | |
|                   | | [Ikon Upload]                                 | | [O Record Pulse]  00:15 / 00:30| |
|                   | | Tarik file .mp3, .wav ke sini (Maks 15 MB)    | | [VU-METER: ||||||||....]     | |
|                   | +-----------------------------------------------+ +-------------------------------+ |
|                   | CONTOH CEPAT: [Contoh Berisik 1] [Contoh Berisik 2] [Contoh Sunyi] [Audio Ambigu] |
|                   +-----------------------------------------------------------------------------------+
|                   | SECTION 2: INSTRUMENT RESULT PANEL (Tampil setelah analisis selesai)              |
|                   | +---------------------------------------+ +-------------------------------------+ |
|                   | | KARTU DIAGNOSIS UTAMA                 | | PEMUTAR AUDIO & SINKRONISASI        | |
|                   | | Label: [ (speaker) BERISIK ]          | | [Play/Pause] 00:12 / 00:30          | |
|                   | | Keyakinan: [==== GAUGE 89% ====]      | | ~~~|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  | |
|                   | | Probabilitas:                         | | (Garis Kursor Berpendar Bergerak)   | |
|                   | | - Berisik     : 89.0 % [==========]   | +-------------------------------------+ |
|                   | | - Non-Berisik : 11.0 % [==]           | | TIMELINE PREDIKSI PER-CHUNK (3 DETIK)|
|                   | +---------------------------------------+ | [B1][B2][B3][B4][B5][S6][B7][B8][B9]| |
|                   | | PENJELASAN AWAM (EXPLAINABILITY)      | | (Keterangan: B=Berisik, S=Sunyi)    | |
|                   | | "Sinyal menunjukkan intensitas energi | | [Label: Indikatif Estimasi]         | |
|                   | | RMS tinggi (0.0406) dan spektrum      +-------------------------------------+ |
|                   | | centroid tajam, mencirikan bising."   |                                       |
|                   | +---------------------------------------+                                       |
|                   +-----------------------------------------------------------------------------------+
|                   | SECTION 3: VISUALISASI FREKUENSI & 10 FITUR LENGKAP                               |
|                   | +---------------------------------------+ +-------------------------------------+ |
|                   | | SPEKTROGRAM FREKUENSI (INFERNO)       | | 10 FITUR TERPILIH VS RATA-RATA KELAS| |
|                   | | [Gambar Spektrogram Resolusi Tinggi]  | | 1. rms_mean       : 0.0406 [||||]   | |
|                   | | Sumbu Y: 0 - 11025 Hz, Sumbu X: Detik | | 2. mfcc1_mean     : -180.2 [||]     | |
|                   | | [Tombol Zoom Spektrogram Modal]       | | 3. rms_std        : 0.0120 [|||]    | |
|                   | +---------------------------------------+ | ... 10. centroid  : 1781Hz [||||||] | |
|                   |                                         +-------------------------------------+ |
+-------------------+-----------------------------------------------------------------------------------+
```

#### Perilaku Responsif Mobile (360px - 767px)
- Navigasi berpindah ke bilah bawah (*bottom bar*) dengan 5 tombol ikon.
- Panel input ditumpuk vertikal: Tab pengalih antara "Unggah Berkas" dan "Rekam Suara".
- Kartu Diagnosis Utama tampil di atas, disusul Waveform, Timeline Chunk, Spektrogram, dan perbandingan fitur di bagian paling bawah.

---

### 9.2 Halaman Riwayat

#### Wireframe ASCII
```text
+-------------------------------------------------------------------------------------------------------+
| SOUND LAB  :: Riwayat Analisis Sesi                                                                   |
+-------------------------------------------------------------------------------------------------------+
| [Filter: Semua (12) | Berisik (8) | Non-Berisik (4)]                    [Tombol Unduh CSV Sesi (.csv)]|
+-------------------------------------------------------------------------------------------------------+
| WAKTU    | NAMA BERKAS / SUMBER       | DURASI | PREDIKSI     | CONFIDENCE | FITUR KUNCI  | AKSI      |
|----------+----------------------------+--------+--------------+------------+--------------+-----------|
| 14:32:05 | 24-003 berisik (1).mp3     | 27.2 s | BERISIK      | 89.0 %     | RMS: 0.0406  | [Detail]  |
| 14:30:11 | Rekaman_Mic_01.wav         | 15.0 s | NON-BERISIK  | 92.4 %     | RMS: 0.0195  | [Detail]  |
| 14:25:40 | 24-003 sunyi 2.mp3         | 30.0 s | NON-BERISIK  | 94.1 %     | RMS: 0.0209  | [Detail]  |
+-------------------------------------------------------------------------------------------------------+
| * Catatan Privasi: Berkas audio mentah tidak disimpan pada peladen. Hanya metadata & vektor fitur.     |
+-------------------------------------------------------------------------------------------------------+
```
- **Empty State:** Ilustrasi sensor kosong dengan teks: *"Belum ada riwayat analisis pada sesi ini. Mulailah menguji audio di halaman Analisis."*
- **Aksi Detail:** Membuka modal yang menampilkan ulang 10 nilai fitur dan perbandingan rata-rata kelas.

---

### 9.3 Halaman Bandingkan (Side-by-Side Comparison)

#### Wireframe ASCII
```text
+-------------------------------------------------------------------------------------------------------+
| SOUND LAB  :: Komparasi Dua Sinyal Akustik                                                            |
+---------------------------------------------------+---------------------------------------------------+
| AUDIO A (Sampel 1)                                | AUDIO B (Sampel 2)                                |
| [Unggah / Pilih Contoh Berisik 1]                 | [Unggah / Pilih Contoh Sunyi 2]                   |
| Status: BERISIK (89.0%)                           | Status: NON-BERISIK (92.0%)                       |
| RMS Mean: 0.04065                                 | RMS Mean: 0.02091                                 |
| Centroid Mean: 1781.2 Hz                          | Centroid Mean: 1729.9 Hz                          |
+---------------------------------------------------+---------------------------------------------------+
| PANEL KOMPARASI RADAR / OVERLAY:                                                                      |
| [Grafik Radar 10 Fitur Terpilih: Garis Oranye (Audio A) vs Garis Cyan (Audio B)]                     |
|                                                                                                       |
| MATRIKS SELISIH FITUR:                                                                                |
| - Delta Energi RMS: +94.4% lebih tinggi pada Audio A (Indikasi suara jauh lebih padat/keras)          |
| - Delta Spektral Centroid: +51.3 Hz lebih tinggi pada Audio A                                         |
+-------------------------------------------------------------------------------------------------------+
```

---

### 9.4 Halaman Tentang Model

#### Komponen Konten & Struktur Informasi
1. **Header Metodologi:** Penjelasan model klasifikasi *Random Forest* (100 estimators, max_depth=10, random_state=42) yang dilatih menggunakan pustaka `scikit-learn 1.8.0`.
2. **Tabel Metrik Performa Evaluasi:**
   | Metrik | Skor Pengujian | Keterangan Evaluasi |
   | :--- | :--- | :--- |
   | **Akurasi Pengujian** | **85.0 %** | Divalidasi pada data uji terpisah (40 sampel). |
   | **Presisi (Weighted)**| **0.85** | Keseimbangan deteksi positif benar. |
   | **Recall (Weighted)** | **0.85** | Sensitivitas penangkapan sinyal bising. |
   | **F1-Score** | **0.85** | Rata-rata harmonik presisi dan recall. |
   | **ROC-AUC** | **0.8688** | Kemampuan pemisahan probabilitas kedua kelas. |
   | **5-Fold Cross Validation** | **84.15% (+/- 2.94%)** | Stabilitas model lintas lipatan data latih. |

3. **Matriks Konfusi (Confusion Matrix):**
   - Data Uji (40 Sampel, Stratified 80:20):
     - Kelas Berisik (20 sampel): 17 Benar (True Positive), 3 Salah (False Negative).
     - Kelas Non-Berisik (20 sampel): 17 Benar (True Negative), 3 Salah (False Positive).

4. **Tabel Komparasi 6 Model Klasifikasi (Benchmarking):**
   | Peringkat | Algoritma Model | Akurasi Test | Status Pemilihan |
   | :---: | :--- | :---: | :--- |
   | **1** | **Random Forest** | **85.0 %** | **Terpilih (Juara Utama)** |
   | 2 | Gradient Boosting | 80.0 % | Kandidat Sekunder |
   | 3 | Support Vector Machine (RBF) | 77.5 % | Tereliminasi |
   | 4 | Support Vector Machine (Linear) | 72.5 % | Tereliminasi |
   | 5 | Logistic Regression | 70.0 % | Tereliminasi |
   | 6 | K-Nearest Neighbors (KNN) | 70.0 % | Tereliminasi |

5. **Tabel Bobot Kepentingan Fitur (Feature Importance - MDI):**
   - 10 fitur terpilih diurutkan dari bobot tertinggi:
     1. `rms_mean`: 0.169 (Tingkat energi amplitudo rata-rata)
     2. `mfcc1_mean`: 0.094 (Energi spektral pita frekuensi rendah pertama)
     3. `rms_std`: 0.092 (Variabilitas dinamika amplitudo sinyal)
     4. `mfcc3_mean`: 0.087 (Komponen harmonik ketiga)
     5. `zcr_mean`: 0.056 (Tingkat pergantian tanda gelombang / tekstur suara)
     6. `mfcc6_mean`: 0.053 (Koefisien bentuk spektral menengah)
     7. `mfcc7_mean`: 0.053 (Koefisien sebaran timbre)
     8. `mfcc13_mean`: 0.045 (Koefisien detail frekuensi tinggi)
     9. `mfcc5_mean`: 0.045 (Karakter timbre instrumen/vokal)
     10. `centroid_mean`: 0.044 (Pusat massa frekuensi suara / ketajaman audio)

---

### 9.5 Halaman Batasan & Etika

#### Poin-Poin Transparansi Sains Data
1. **Distribusi Dataset Latih:** Model dikembangkan berdasarkan ~200 sampel audio rekaman ruang fisik kampus (durasi ~30 detik, 44.1 kHz).
2. **Suara di Luar Distribusi (Out-of-Distribution / OOD):** Model belum dilatih untuk menangani musik terkompresi dengan dinamika drum beat sintetis, suara hujan deras terus menerus, atau deru mesin turbin industri berat. Pada kondisi ini, skor *confidence* dapat menurun.
3. **Komitmen Privasi Total:**
   - Tidak ada audio mentah yang disimpan di database Supabase atau media penyimpanan peladen.
   - Perekaman peramban sepenuhnya diproses di memori lokal klien sebelum diunggah sementara ke endpoint inferensi, dan file sementara dihapus seketika setelah fitur diekstrak.
4. **Pemberitahuan Lisensi & Tanggung Jawab:** Hasil klasifikasi ditujukan sebagai instrumen bantu penelitian akademis dan evaluasi kenyamanan ruang, bukan sebagai alat bukti legal yuridis baku mutu lingkungan hidup formal.

---

## 10. Aksesibilitas (WCAG 2.1 AA) & Validasi Kontras

1. **Rasio Kontras Teks:**
   - Teks `#E6EDF3` di atas latar `#0B0F14`: Rasio **14.8:1** (Lolos WCAG AAA).
   - Teks `#8B98A9` di atas latar `#121922`: Rasio **6.2:1** (Lolos WCAG AA).
   - Aksen Berisik `#FF6B35` di atas `#121922`: Rasio **5.3:1** (Lolos WCAG AA).
   - Aksen Non-Berisik `#22D3C5` di atas `#121922`: Rasio **9.8:1** (Lolos WCAG AAA).
2. **Fokus Keyboard (Keyboard Focus Indicators):** Semua tombol, slider audio, tab, dan baris tabel memiliki garis luar fokus eksplisit: `outline: 2px solid #8B5CF6; outline-offset: 2px`.
3. **Aria Labels & Peran Aksesibilitas:**
   - Pemutar audio menyematkan atribut `aria-label="Pemutar rekaman audio lingkungan"`.
   - Tombol rekam mikrofon menyematkan `role="button"` dan status dinamis `aria-pressed="true|false"`.
   - Spektrogram memiliki teks alternatif: `alt="Visualisasi spektrogram inferno frekuensi audio rentang 0 hingga 11 kHz"`.

---

## 11. Panduan Visualisasi Data Akustik

1. **Integritas Sumbu:** Sumbu waktu selalu menggunakan detik (`s`), sumbu frekuensi menggunakan Hertz (`Hz`) atau kiloHertz (`kHz`), dan sumbu energi menggunakan skala normalisasi desibel (`dBFS`).
2. **Konsistensi Visual Kelas:**
   - Kategori **BERISIK** tidak boleh ditampilkan selain menggunakan warna `#FF6B35`.
   - Kategori **NON-BERISIK** tidak boleh ditampilkan selain menggunakan warna `#22D3C5`.
3. **Grid Plot Latar:** Setiap kanvas grafik (waveform dan radar) dilengkapi garis grid tipis berjarak teratur (`#1F2A37`, ketebalan 0.5px) dengan angka pemandu monospaced pada tiap interval utama untuk memperkuat sensasi instrumen laboratorium presisi.
