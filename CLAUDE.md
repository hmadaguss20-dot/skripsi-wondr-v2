# PROMPT — Program Skripsi v3 (bertahap, sesuai proposal revisi 21 Oktober 2026)

> **Cara pakai:** simpan berkas ini sebagai `CLAUDE.md` di folder proyek Claude Code, lalu salin PDF proposal final ke `naskah/Revisi_Proposal_Aas_Bu_Anggi.pdf` di folder yang sama. Setelah itu ketik: **"Mulai Tahap 0.1."**

---

## 1. Konteks

Saya Ahmad Agus Subhan, mahasiswa S1 Matematika, Universitas Brawijaya. Judul skripsi saya:

**"Optimasi Hyperparameter Model IndoBERT Menggunakan Particle Swarm Optimization Disertai Uji Signifikansi Statistik pada Analisis Sentimen Ulasan Aplikasi Wondr by BNI"**

Tugasmu adalah membangun program penelitian sebagai notebook Google Colab yang **bertahap, bersih, dan sesuai naskah**. Metodologi sudah final di proposal: Bab III berisi 16 tahap, dan Bab II memuat persamaan beserta rujukannya. **Kamu menerjemahkan spesifikasi menjadi kode, bukan merancang ulang metode.** Kalau prompt ini dan naskah bertentangan, berhenti dan tanyakan ke saya.

Satu tahap naskah boleh dipecah menjadi beberapa **subtahap** (misalnya 8.1, 8.2, 8.3) supaya setiap sel pendek dan keluarannya mudah dipahami. Nomor tahap utama selalu sama dengan nomor tahap di Bab III.

**Lingkungan**
- Google Colab dengan GPU **A100** (keputusan pengguna 21 Oktober 2026, menggantikan T4; Bab III perlu disesuaikan). Seluruh pelatihan dijalankan di GPU yang sama.
- Folder kerja baru di Google Drive: `MyDrive/skripsi_wondr_v3/`. Folder lama tidak boleh ditimpa.
- Data mentah `data_wonder_bybni_3.xlsx` (7.204 baris), saya salin ke `skripsi_wondr_v3/data/`.
- Kamu menulis kodenya, saya menjalankannya di Colab, lalu menempelkan output ke sini.
- Setiap subtahap juga ditambahkan ke `skripsi_wondr_v3.ipynb` di folder Claude Code: 1 sel markdown (judul, dasar naskah, package) ditambah 1–2 sel kode. Kodenya juga ditampilkan di balasan supaya bisa langsung saya salin.

---

## 2. Aturan kerja (wajib)

1. **Satu subtahap per balasan**, maksimal 2 sel kode. Setelah itu berhenti dan tunggu output saya.
2. **Sel kode biasa paling banyak sekitar 40 baris**, dengan satu tujuan per sel. Sel `%%writefile lib/...` boleh lebih panjang, tetapi isinya hanya fungsi.
3. Sebelum memberi subtahap berikutnya, **analisis output** subtahap sebelumnya terhadap target di Bagian 6.
4. Jika ada error, *warning*, atau angka yang menyimpang dari target, **jangan lanjut**. Pakai format A–E.
5. Jangan mengubah metodologi, nama kolom, atau angka konfigurasi. Hal teknis yang tidak diatur di sini ditulis sebagai **"Keputusan teknis"** beserta alasannya, dan harus saya setujui dulu. Untuk subtahap yang memakai GPU lama (8.2, 11, 12, 13.1, 14), sebutkan perkiraan waktunya dan minta persetujuan sebelum dijalankan.
6. **Larangan:**
   - menghapus data tanpa menampilkan jumlah sebelum dan sesudah;
   - membagi data sebelum *pre-processing* dan deduplikasi selesai;
   - menjalankan *random search* atau PSO sebelum Tahap 11 selesai;
   - memakai label atau metrik data uji sebelum Tahap 15, kecuali untuk mengekspor lembar anotasi di Tahap 9.1.
7. Bahasa balasan: Indonesia yang mudah dipahami. Istilah teknis diberi arti di dalam kurung saat pertama muncul. Istilah seperti *random search*, *fine-tuning*, *learning rate*, *weight decay*, dan *confident learning* tetap dalam bahasa Inggris.
8. Setiap angka yang kamu sebut diberi label asal: (Rujukan), (Data), (Hitungan), (Keputusan), (Bawaan), atau (Contoh).
9. Penjelasan cukup 2–4 kalimat. Saya sudah memahami teorinya. Yang saya butuhkan adalah program yang benar dan keluaran yang bisa dipertanggungjawabkan.

### Format balasan setiap subtahap

```
## Tahap X.Y — [Nama]
Dasar naskah: Bab III tahap X; Bab II subbab ..., persamaan (...); rujukan ...

### 1. Tujuan
### 2. Penjelasan (alasan dan dampak, 2–4 kalimat)
### 3. Program            ← sel kode, diawali daftar package di baris komentar pertama
### 4. Output yang saya harapkan   ← sebut angka target dari Bagian 6
### 5. Interpretasi       ← tanda normal vs tanda bermasalah

Status: Menunggu output Tahap X.Y dari pengguna.
```

### Format jika hasil berbeda dari harapan

- **A. Identifikasi masalah**
- **B. Analisis penyebab**: data, *pre-processing*, label, pembagian data, tokenisasi, parameter, atau *bug*; urutkan dari yang paling mungkin
- **C. Verifikasi** dengan kode kecil
- **D. Perbaikan** hanya pada sel yang bermasalah
- **E. Keputusan**: ✅ valid / ⚠️ perlu revisi / ❌ kembali ke tahap sebelumnya

---

## 3. Prinsip program bersih

### 3.1 Struktur folder

```
skripsi_wondr_v3/
├── data/          data_wonder_bybni_3.xlsx
├── lib/           wondr_teks.py (1.2), wondr_latih.py (8.1), wondr_label.py (8.3)
├── hasil/         berkas antar-tahap: tahapXX_nama.parquet / .json / .csv
├── anotasi/       lembar anotator, kunci anotasi, berkas selisih (Tahap 9)
├── bab4/tabel/    CSV untuk Bab IV
├── bab4/gambar/   PNG 300 dpi untuk Bab IV
└── log/           versi_package.json, state lipatan CL, rs_state.json, pso_state.json
```

### 3.2 Bentuk sel

- Setiap sel subtahap **berdiri sendiri**. Setelah *restart*, cukup jalankan sel Tahap 0.1 dan 0.2, lalu sel mana pun bisa dijalankan ulang.
- Antar-tahap berkomunikasi lewat berkas di `hasil/`. Tidak ada variabel yang dibawa antar-sel, kecuali `PATH`, `CFG`, `TARGET`, dan fungsi bantu dari Tahap 0.2.
- Fungsi yang dipakai lebih dari satu tahap ditulis **sekali** ke `lib/` dengan `%%writefile`, lalu di-*import*.
- Baris pertama setiap sel: `# ══ Tahap X.Y — Nama ══` diikuti `# Package: ...`. Urutan isinya: baca masukan → proses → simpan → cetak ringkasan dan cek.
- Komentar singkat berbahasa Indonesia. Setiap angka konfigurasi diambil dari `CFG`, tidak ditulis ulang di sel.

### 3.3 Keluaran hanya yang dibutuhkan

- Di Tahap 0.1, matikan semua kebisingan: `warnings.filterwarnings("ignore")`, `transformers.logging.set_verbosity_error()`, *progress bar* `tqdm`/`datasets`/HF Hub, `TOKENIZERS_PARALLELISM=false`, dan pesan "Some weights … newly initialized".
- **Jangan mencetak** `df.head()`, `df.info()`, daftar kolom, isi berkas, atau log per *step* pelatihan.
- Setiap subtahap mencetak **satu blok ringkasan** dengan bentuk tetap:

```
══════════ TAHAP 4 — PRE-PROCESSING ══════════
Masukan : hasil/tahap03_label.parquet (7.204 baris)
Keluaran: hasil/tahap04_bersih.parquet (7.141 baris)
────────────────────────────────────────────────
[1–2 tabel ringkas yang memang dibutuhkan]
────────────────────────────────────────────────
Cek otomatis:
  ✅ jumlah setelah pre-processing = 7.141 (target 7.141)
Waktu: 3,2 detik
```

- Angka di ringkasan memakai format Indonesia (7.204; 0,8360). Berkas CSV memakai angka mentah.
- Pelatihan dicetak satu baris per pelatihan: konfigurasi, *seed*, *epoch* terbaik, *Macro F1* validasi terbaik, dan durasi.
- Gambar hanya yang tercantum di Bagian 6, disimpan ke `bab4/gambar/` dan ditampilkan satu kali.
- Cek otomatis tidak memakai `assert`. Cetak ✅/⚠️ beserta nilai aktual dan target, memakai fungsi `cek()` dan kamus `TARGET`.

### 3.4 Keterulangan (*reproducibility*)

- Setiap sumber keacakan memakai *seed* sendiri (Bagian 4).
- `atur_seed(s)` menjalankan `random.seed`, `np.random.seed`, `torch.manual_seed`, `torch.cuda.manual_seed_all`, dan `transformers.set_seed`, serta mengatur `cudnn.deterministic = True` dan `cudnn.benchmark = False`.
- `CUBLAS_WORKSPACE_CONFIG=":4096:8"` diatur di Tahap 0.1, sebelum torch memakai GPU.
- Subtahap 8.2, 12, dan 13.1 menyimpan *state* setelah setiap pelatihan atau evaluasi. Kalau Colab terputus, menjalankan ulang sel yang sama melanjutkan dari titik terakhir.

---

## 4. Konfigurasi tetap (`CFG` di Tahap 0.2)

Nomor persamaan, tabel, dan subbab mengikuti PDF proposal final.

| Nama | Nilai | Asal |
|---|---|---|
| *Checkpoint* | `indobenchmark/indobert-base-p1` | Bab III tahap 7; Wilie dkk. (2020) |
| Label awal | `rating` ≥ 4 → 1 (Positif); ≤ 3 → 0 (Non-Positif) | pers. (2.1)–(2.2), τ = 4; Morgan & Rego (2006) |
| Proporsi pembagian | latih 0,70 · validasi 0,15 · uji 0,15 | Bab III tahap 6 |
| *Seed* pembagian | 42 | Bab III tahap 6 |
| Kelompok panjang | 1–2, 3–10, >10 kata | Bab III tahap 6 |
| L_max | b·⌈P_q / b⌉, q = 100, b = 32, **dari data latih saja**; panjang = token subkata + [CLS] + [SEP] | pers. (2.9); Bab III tahap 7 |
| *Confident learning* | 5 lipatan berbasis grup pada data latih; skema *default*; 4 *epoch* tanpa *early stopping*; validasi = rata-rata peluang 5 model; uji tidak ikut; ambang dan *confident joint* terpisah untuk latih dan validasi | pers. (2.3)–(2.4); Bab III tahap 8; Northcutt dkk. (2021) |
| *Seed* CL | `StratifiedGroupKFold(5, shuffle=True, random_state=42)`; *seed* pelatihan 42 | Keputusan teknis (pemilihan kelas pemecah lipatan) |
| Anotasi | 2 anotator, buta terhadap *rating*/label awal/prediksi; 1 = Positif, 0 = Non-Positif; urutan lembar diacak `default_rng(2026)`; κ Cohen + tafsiran Landis & Koch; selisih diselesaikan lewat diskusi | pers. (2.5), Tabel 2.10; Bab III tahap 9 |
| *Batch size* `B` | 16 | Tabel 3.1; Devlin dkk. (2019) |
| *Epoch* maksimum `E` | 4 | Tabel 3.1; Sun dkk. (2019) |
| *Patience* `p` | 2 | Tabel 3.1 (Keputusan) |
| Batas norma gradien `v` | 1,0 | Tabel 3.1, pers. (2.19) (Keputusan) |
| *Optimizer* | AdamW, β = (0,9; 0,999), ε = 1e-8 | pers. (2.18); bawaan PyTorch |
| Jadwal *learning rate* | naik linear lalu turun linear ke 0; T = ⌈n_latih / B⌉ × E; T_w = ⌊ρ·T⌋ | pers. (2.20)–(2.22) |
| *Loss* | *cross-entropy* tanpa bobot kelas | pers. (2.15); Batasan masalah 4 |
| Skema *default* | lr 2e-5 · *warmup ratio* 0,1 · *weight decay* 0,01 | Tabel 3.1; Devlin dkk. (2019); Sun dkk. (2019) |
| Ruang pencarian | lr [1e-5; 5e-5] skala log · wr [0; 0,2] · wd [0; 0,3] | Tabel 3.1; pers. (2.27); Liu & Wang (2021) |
| Dekode f ∈ [0,1]³ | lr = 1e-5 · 5^f₁ · wr = 0,2·f₂ · wd = 0,3·f₃ | Hitungan |
| *Fitness* | rata-rata *Macro F1* validasi terbaik dari *seed* 42 dan 43 | pers. (2.25); Bab III tahap 11 |
| *Random search* | M = 50; `default_rng(2024)`; `F = rng.random((50, 3))` | Bab III tahap 12 |
| PSO | s = 10; G_max = 4 → 10 × (1 + 4) = 50 evaluasi; ω = 0,729; c₁ = c₂ = 1,49445; V_max = 1,0; posisi awal U(0,1); kecepatan awal U(−1,1); *seed* 7; pembaruan serentak; dinding penyerap | Tabel 3.2; pers. (2.28)–(2.35); Zeng dkk. (2025); Lorenzo dkk. (2017); Clerc & Kennedy (2002); Eberhart & Shi (2000); Robinson & Rahmat-Samii (2004) |
| Model akhir | 3 skema × *seed* 42, 43, 44 | Bab III tahap 14 |
| Ansambel | suara terbanyak dari 3 model | pers. (2.23) |
| *Bootstrap* | R = 10.000; `default_rng(42)`; persentil; berpasangan (satu matriks indeks untuk semua skema) | pers. (2.41)–(2.43); Efron & Tibshirani (1993); Dror dkk. (2018) |
| Koreksi banyak-uji | Bonferroni, m = 3, α = 0,05 → α' = 0,05/3; selang 98,33% | pers. (2.44)–(2.45); Dror dkk. (2017) |
| Kembaran-dekat di data uji | `review_id` 339, 595, 2854, 3946, 4128 | Data (temuan pembagian) |

**Penamaan:** jumlah sampel ulang *bootstrap* ditulis `R`, bukan `B`, karena `B` sudah dipakai untuk *batch size*.

---

## 5. Peta tahap naskah → subtahap program

| Bab III | Subtahap program | GPU |
|---|---|---|
| — | 0.1 Lingkungan · 0.2 Konfigurasi dan fungsi bantu | – |
| 1 Pengumpulan Data | 1.1 Muat data mentah · 1.2 Tulis `lib/wondr_teks.py` + audit kualitas data *(audit: program saja)* | – |
| 2 Seleksi Atribut | 2 | – |
| 3 Pelabelan Awal | 3 | – |
| 4 *Pre-processing* | 4 (diawali uji contoh Bab II) | – |
| 5 Deduplikasi | 5 | – |
| 6 Pembagian Data | 6.1 Bagi data · 6.2 Cek kebocoran dan keseimbangan | – |
| 7 Tokenisasi | 7 | – |
| 8 Deteksi Kandidat Label Salah | 8.1 Tulis `lib/wondr_latih.py` + uji cepat · 8.2 Pelatihan 5 lipatan · 8.3 Ambang, *confident joint*, kandidat | 8.2 |
| 9 Anotasi Manual | 9.1 Ekspor lembar · *(jeda: anotasi)* · 9.2 Impor + κ · *(jeda: diskusi)* · 9.3 Terapkan label akhir | – |
| 10 Eksplorasi Data | 10 (+ acuan minimum validasi, *program saja*) | – |
| 11 *Fine-tuning* Skema *Default* | 11 | ✔ |
| 12 *Random Search* | 12 | ✔ |
| 13 PSO | 13.1 Uji fungsi + jalankan PSO · 13.2 Ringkasan dan konvergensi · 13.3 Ragam antar-*seed* *(program saja)* | 13.1 |
| 14 Pelatihan Model Akhir | 14 | ✔ |
| 15 Evaluasi Model | 15.1 Per model dan per skema · 15.2 Ansambel · 15.3 Kesesuaian label *rating* vs label manual (data uji) | – |
| 16 Uji Signifikansi Statistik | 16 | – |

Yang ditandai *(program saja)* tetap dijalankan dan disimpan, tetapi tidak dibahas di naskah.

---

## 6. Spesifikasi per subtahap

**Angka target Tahap 1–7 sudah diverifikasi** dengan menjalankan ulang pipeline di luar Colab. Kalau output berbeda, itu tanda ada *bug*. Mulai Tahap 8, label berubah karena *confident learning* dan anotasi, sehingga sebagian besar target berupa cek logika, bukan angka.

### Tahap 0.1 — Lingkungan

- **Proses:**
  - *mount* Google Drive dan buat struktur folder (3.1);
  - cek berkas data ada;
  - cek GPU (nama dan memori), dan beri ⚠️ kalau bukan A100;
  - atur variabel lingkungan untuk determinisme dan matikan kebisingan (3.3–3.4);
  - instal hanya package yang belum ada (`pip install -q`).
- **Tampilkan:** tabel package dan versinya, nama GPU, dan path folder kerja.
- **Simpan:** `log/versi_package.json`.

### Tahap 0.2 — Konfigurasi dan fungsi bantu

- **Proses:** buat `PATH`, `CFG` (Bagian 4), `TARGET` (angka target Bagian 6), serta fungsi `atur_seed()`, `fmt_id()` (format angka Indonesia), `judul()`, `cek(nama, aktual, target, toleransi)`, dan `simpan_tabel()`.
- **Tampilkan:** jumlah entri `CFG` dan `TARGET`, cukup satu baris.

### Tahap 1.1 — Muat data mentah

- **Proses:**
  - baca .xlsx dan buang kolom `Unnamed*`;
  - tambahkan `review_id` = 1 sampai 7.204 sesuai urutan baris;
  - isi data mentah tidak diubah.
- **Target:**
  - 7.204 baris;
  - kolom `nama, rating, ulasan, tanggal, likes, balasan_dev, versi`;
  - tanggal 19 Juli 2026 sampai 11 September 2026;
  - versi hanya `1.6.0`;
  - per bulan: Juli 1.747 · Agustus 3.899 · September 1.558.
- **Simpan:** `hasil/tahap01_mentah.parquet`.

### Tahap 1.2 — Tulis `lib/wondr_teks.py` dan audit kualitas data *(audit: program saja; data tidak diubah)*

- **Sel 1:** tulis kode acuan Bagian 7 **persis** ke `lib/wondr_teks.py`. Pola teks pertama kali dibutuhkan di sini, dan audit harus memakai pola yang sama dengan Tahap 4.
- **Sel 2:** audit memakai `EMO`, `SIMBOL`, `HURUF`, dan `BACA` dari `lib.wondr_teks`. Label untuk baris "teks berlabel ganda" dihitung sementara sebagai `rating >= 4`.
- **Target:**

| Pemeriksaan | Definisi | Target |
|---|---|---|
| Nilai kosong | `isna()` per kolom | hanya `balasan_dev`: 4.834 |
| Balasan pengembang | persentase dibalas per kelompok *rating* | *rating* 1–4: 95,0% · *rating* 5: 1,0% |
| *Likes* | persentase nol; Spearman *likes*–*rating* | 95,1%; ρ = −0,2483 |
| *Rating* 3 yang panjang | *rating* 3 dengan ≥ 8 kata | 237 dari 366 |
| Median kata per *rating* | `ulasan` dipisah spasi | 15, 15, 11, 3, 2 |
| Emoji atau simbol | cocok `EMO` atau `SIMBOL` | 404 |
| Huruf berulang ≥ 3 | `HURUF` pada teks huruf kecil | 227 (3,2%) |
| Tanda baca berulang | `BACA` | 634 |
| Duplikat persis | `ulasan.duplicated()` | 34,8% |
| Teks berlabel ganda | teks (huruf kecil, tanpa spasi tepi) dengan dua label | 33 teks, 1.968 baris |
| Nama unik | `nama.nunique()` | 7.153 |

- **Tampilkan:** satu tabel "temuan → angka → tahap yang memakainya".
- **Simpan:** `bab4/tabel/tahap01_audit.csv`.

### Tahap 2 — Seleksi atribut

- **Dasar:** Bab III tahap 2; Bab II 2.4.2.
- **Proses:** buang `nama`, `balasan_dev`, dan `likes`; pertahankan `review_id, ulasan, rating, tanggal, versi`.
- **Tampilkan:** tabel atribut | dipakai/dibuang | alasan, sesuai kalimat Bab III tahap 2.
- **Target:** 7.204 baris, 5 kolom.
- **Simpan:** `hasil/tahap02_atribut.parquet`.

### Tahap 3 — Pelabelan awal (*top-2-box*)

- **Dasar:** pers. (2.1)–(2.2); Bab III tahap 3.
- **Proses:** `label = (rating >= 4).astype(int)`. Kolom bernama `label` supaya kode acuan Bagian 7 tidak diubah; namanya diganti `label_awal` di Tahap 9.3. Tidak ada baris yang dihapus.
- **Target:**
  - *rating* 1: 1.305 · 2: 283 · 3: 366 · 4: 492 · 5: 4.758;
  - Positif 5.250 (72,88%), Non-Positif 1.954 (27,12%).
- **Gambar:** `bab4/gambar/tahap03_distribusi_rating.png`, grafik batang per *rating* yang diwarnai per kelas.
- **Simpan:** `hasil/tahap03_label.parquet`.

### Tahap 4 — *Pre-processing*

- **Dasar:** Bab III tahap 4; Bab II 2.2.
- **Uji dulu** `bersihkan()` dengan contoh Bab II (Tabel 2.1 dan Tabel 2.2–2.7). Tulis masukan uji sebagai *escape* Unicode:
  - `"\U0001D401\U0001D400\U0001D406\U0001D414\U0001D412\U0001D412\U0001D412 bgt!!! \u2605\u2605 tp kadang\u00b2 error, nunggu lamaaa"` → `"baguss bgt! tp kadang2 error, nunggu lamaa"`;
  - `"\uff4d\uff41\uff4e\uff54\uff41\uff50"` → `"mantap"`.
- **Proses:** jalankan 6 langkah `LANGKAH` secara berurutan, lalu buang teks kosong. Tanpa *stemming*, tanpa *stopword removal*, dan tanpa normalisasi kamus.
- **Tampilkan:**
  - jumlah ulasan yang berubah di setiap langkah;
  - jumlah teks kosong yang dibuang beserta *rating*-nya;
  - 3 contoh sebelum → sesudah untuk langkah 4.
- **Target:**
  - perubahan per langkah: 55 · 404 · 1.518 · 227 · 634 · 403;
  - teks kosong 63 (*rating* 5: 57, *rating* 4: 6);
  - hasil **7.141**.
- **Simpan:** `hasil/tahap04_bersih.parquet`, dengan tambahan kolom `text_clean`.

### Tahap 5 — Deduplikasi

- **Dasar:** Bab III tahap 5; Tabel 2.11.
- **Proses:** `deduplikasi()` dari `lib.wondr_teks`.
- **Tampilkan:**
  - alur jumlah 7.141 → jumlah kunci → jumlah dokumen;
  - rincian yang dibuang: salinan berlabel sama, anggota kalah suara, anggota kunci seri;
  - proporsi kelas sebelum dan sesudah.
- **Target:**
  - 4.372 kunci, termasuk 8 kunci seri (16 ulasan);
  - **4.364 dokumen**;
  - dibuang 2.777 = 2.688 + 73 + 16;
  - Positif 72,6% → 57,3%.
- **Simpan:** `hasil/tahap05_dedup.parquet` (dengan `dedup_key`) dan `bab4/tabel/tahap05_alur.csv`.

### Tahap 6.1 — Pembagian data

- **Dasar:** Bab III tahap 6; pers. (2.6); Tabel 2.12–2.13.
- **Proses:** `bagi_data(seed=42)` dari `lib.wondr_teks`.
- **Target:**
  - 4.293 grup;
  - latih 3.051 (Non-Positif 1.303 / Positif 1.748);
  - validasi 657 (282 / 375);
  - uji 656 (280 / 376).
- **Simpan:** `hasil/tahap06_latih.parquet`, `hasil/tahap06_validasi.parquet`, `hasil/tahap06_uji.parquet`, dengan kolom `review_id, ulasan, text_clean, rating, label, group_key, dedup_key, kelompok_panjang`.

### Tahap 6.2 — Cek kebocoran dan keseimbangan

- **Proses:**
  - `group_key`, `text_clean`, dan `dedup_key` tidak boleh muncul di lebih dari satu bagian;
  - kelima `review_id` kembaran-dekat berada di data uji;
  - uji khi-kuadrat bagian × label dan bagian × kelompok panjang.
- **Target:**
  - 0 kebocoran;
  - Non-Positif 42,7% / 42,9% / 42,7%;
  - kelompok panjang 1–2 / 3–10 / >10 kata: latih 14,0 / 47,1 / 38,9%; validasi 14,3 / 46,9 / 38,8%; uji 14,0 / 47,1 / 38,9%;
  - p = 0,9945 (label) dan 0,9998 (panjang).
- **Simpan:** `bab4/tabel/tahap06_strata.csv`.
- **Mulai di sini data uji dikunci.**

### Tahap 7 — Tokenisasi

- **Dasar:** Bab III tahap 7; pers. (2.7)–(2.9); Tabel 2.14.
- **Proses** (statistik **hanya dari data latih**):
  - *fertility* F = jumlah token subkata ÷ jumlah kata;
  - r_UNK = jumlah `[UNK]` ÷ jumlah token subkata;
  - panjang token per dokumen = token subkata + 2;
  - L_max = 32·⌈maks_latih / 32⌉;
  - sebagai deskripsi saja: jumlah dokumen validasi dan uji yang terpotong pada L_max.
- **Tampilkan:** tabel per bagian (n, median, P95, maksimum, jumlah terpotong), serta F, r_UNK, dan L_max.
- **Target:**
  - L_max = **128**; kalau berbeda, berhenti dan laporkan, karena naskah menulis 128;
  - dokumen terpotong: latih 0, validasi 0, uji 1 (`review_id` 2741).
- **Gambar:** `bab4/gambar/tahap07_panjang_token.png`, histogram panjang token data latih dengan garis L_max.
- **Simpan:** `hasil/tahap07_tokenisasi.json`.

### Tahap 8.1 — Tulis `lib/wondr_latih.py` dan uji cepat

- **Dasar:** Bab II 2.6.2 dan 2.8; pers. (2.14), (2.15), (2.18)–(2.22).
- **Isi `lib/wondr_latih.py`:**
  - `latih(df_latih, lr, wr, wd, seed, df_val=None, early_stopping=True, prediksi=None)` → dict berisi `f1_terbaik`, `epoch_terbaik`, `f1_per_epoch`, `detik`, dan `prob` (peluang Positif untuk setiap DataFrame di `prediksi`).
    - `atur_seed(seed)` dipanggil **sebelum** model dibuat;
    - tokenisasi dengan `truncation=True, max_length=L_max` dan *padding* dinamis per *batch*, di-*cache* per sesi;
    - DataLoader latih `shuffle=True` dengan `torch.Generator().manual_seed(seed)`;
    - AdamW; *weight decay* tidak dikenakan pada bias dan `LayerNorm.weight`. **Ini Keputusan teknis: konfirmasikan dulu ke saya**;
    - T = ⌈len(df_latih) / 16⌉ × 4 dan T_w = ⌊ρ·T⌋; jadwal tetap memakai T ini walaupun pelatihan berhenti lebih awal;
    - *loss* *cross-entropy* tanpa bobot kelas, lalu `clip_grad_norm_(…, 1.0)`;
    - jika `early_stopping=True`: hitung *Macro F1* validasi (`average="macro", zero_division=0`) setiap *epoch*. Skor terbaik diganti hanya bila **lebih besar**. Peluang `prediksi` diambil pada *epoch* terbaik (dihitung saat skor membaik, tanpa menyimpan bobot). Berhenti setelah 2 *epoch* berturut-turut tidak membaik;
    - jika `early_stopping=False`: latih penuh 4 *epoch*, lalu peluang `prediksi` diambil dari bobot *epoch* terakhir;
    - bebaskan memori GPU setelah selesai.
  - `fitness(lr, wr, wd)` → rata-rata `f1_terbaik` *seed* 42 dan 43 pada `hasil/tahap09_latih` dan `hasil/tahap09_validasi` (kolom `label_akhir`), beserta kedua skornya.
- **Uji cepat:**
  - T dan T_w untuk n = 3.051 dan ρ = 0,1 harus 764 dan 76;
  - 1 *epoch* pada 64 dokumen latih berjalan tanpa error, dan hasilnya tidak disimpan.

### Tahap 8.2 — *Confident learning*: pelatihan 5 lipatan

- **Dasar:** Bab III tahap 8; Bab II 2.3.4; Northcutt dkk. (2021).
- **Proses:**
  - bentuk lipatan pada data latih dengan `StratifiedGroupKFold(5, shuffle=True, random_state=42)`, `groups=group_key`, `y=label`;
  - untuk setiap lipatan k: `latih(4 lipatan lain, 2e-5, 0.1, 0.01, seed=42, early_stopping=False, prediksi={lipatan k, data validasi})`;
  - simpan peluang ke `log/cl_lipatan_k.parquet`. Lipatan yang sudah tersimpan dilewati, sehingga proses bisa dilanjutkan;
  - data uji tidak disentuh.
- **Tampilkan:** satu baris per lipatan: k, n latih, n diprediksi, *Macro F1* lipatan terhadap label awal (deskriptif), dan durasi.
- **Cek:**
  - setiap dokumen latih mendapat tepat satu peluang di luar sampel;
  - tidak ada `group_key` yang terbelah antarlipatan;
  - setiap dokumen validasi mendapat 5 peluang.

### Tahap 8.3 — Ambang, *confident joint*, dan kandidat

- **Tulis `lib/wondr_label.py`** dengan fungsi `confident_learning(p1, y)`, dengan p₀ = 1 − p₁:
  - t_j = rata-rata p_j pada dokumen berlabel j (pers. 2.3);
  - kelas taksiran = argmax di antara kelas yang p_l ≥ t_l; dokumen yang tidak memenuhi ambang mana pun tidak dihitung;
  - C_ij = banyaknya dokumen berlabel i dengan kelas taksiran j (pers. 2.4);
  - kandidat = dokumen dengan kelas taksiran ≠ label.
  - **Rumus ditulis sendiri sesuai persamaan Bab II.** Pustaka `cleanlab` tidak dipakai, karena kalibrasi bawaannya berbeda dengan naskah.
- **Uji fungsi** dengan contoh Tabel 2.9: p₁ = [0,95; 0,90; 0,15; 0,10; 0,20; 0,85] dan label = [1, 1, 1, 0, 0, 0] harus menghasilkan t₀ = 0,6167, t₁ = 0,6667, C = [[2, 1], [1, 2]], dan kandidat d₃ serta d₆.
- **Proses:** terapkan secara terpisah pada data latih (peluang di luar sampel) dan data validasi (rata-rata 5 lipatan).
- **Tampilkan:** untuk setiap bagian: t₀, t₁, matriks C, jumlah kandidat per arah (Positif → Non-Positif dan sebaliknya), persentase kandidat, dan jumlah dokumen yang tidak dihitung.
- **Cek:** tidak ada angka target. Tanda bermasalah: kandidat 0, kandidat > 40%, atau diagonal C tidak dominan.
- **Simpan:** `hasil/tahap08_kandidat.parquet` (`review_id, bagian, label, p1, kelas_taksiran, kandidat`) dan `bab4/tabel/tahap08_cj.csv`.

### Tahap 9.1 — Ekspor lembar anotasi

- **Dasar:** Bab III tahap 9; Bab II 2.3.5.
- **Proses:**
  - gabungkan kandidat latih, kandidat validasi, dan **seluruh** data uji (656);
  - acak urutannya dengan `default_rng(2026)` dan beri `id_anotasi` D0001, D0002, dan seterusnya;
  - teks yang ditampilkan adalah `text_clean`, yaitu masukan yang sama dengan masukan model. **Keputusan teknis, konfirmasikan dulu**;
  - buat `anotasi/lembar_anotator_A.xlsx` dan `anotasi/lembar_anotator_B.xlsx` yang identik. Sheet "Petunjuk" berisi definisi label dari Bab III tahap 9. Sheet "Anotasi" berisi kolom `id_anotasi, teks, label` (kosong; isi 1 = Positif, 0 = Non-Positif);
  - simpan `anotasi/kunci_anotasi.csv` (`id_anotasi, review_id, bagian, kandidat`). Berkas ini **tidak** diberikan ke anotator;
  - lembar anotasi tidak boleh memuat *rating*, label awal, prediksi, atau asal bagian.
- **Tampilkan:** jumlah dokumen per sumber (kandidat latih, kandidat validasi, uji) dan totalnya.
- **Setelah ini berhenti.** Anotasi dikerjakan manual oleh dua anotator secara terpisah. Lanjut ke 9.2 setelah saya mengunggah kedua lembar.

### Tahap 9.2 — Impor anotasi dan kesepakatan

- **Proses:**
  - validasi kedua lembar: id lengkap dan sama, semua label bernilai 0/1, tidak ada sel kosong;
  - hitung κ dengan `sklearn.metrics.cohen_kappa_score` dan dengan rumus pers. (2.5); keduanya harus sama sampai 1e-12;
  - uji rumus dengan contoh Bab II 2.3.5 (100 ulasan; 40 sepakat Positif; 45 sepakat Non-Positif; A memberi Positif 45, B memberi Positif 50) → κ = 0,70;
  - tafsirkan κ dengan Tabel 2.10 (Landis & Koch);
  - ekspor `anotasi/selisih.xlsx` (`id_anotasi, teks, label_A, label_B, label_final` kosong).
- **Tampilkan:** p_o, p_e, κ, dan tafsirannya, secara keseluruhan dan per sumber; serta jumlah selisih.
- **Setelah ini berhenti.** Selisih diselesaikan lewat diskusi kedua anotator dan diisi di kolom `label_final`.

### Tahap 9.3 — Terapkan label akhir

- **Proses:**
  - label manual = label A jika A dan B sepakat, atau `label_final` jika berbeda. Cek: semua `label_final` terisi 0/1;
  - kolom `label` diganti nama menjadi `label_awal`;
  - latih dan validasi: `label_akhir` = label manual untuk kandidat, label awal untuk dokumen lain;
  - uji: `label_akhir` = label manual untuk seluruh dokumen;
  - tambahkan kolom `kandidat` dan `berubah`.
- **Tampilkan:**
  - latih dan validasi: jumlah kandidat, jumlah label yang berubah per arah, dan proporsi kelas sebelum → sesudah;
  - uji: cukup n = 656 dokumen berlabel manual. **Sebaran label uji belum ditampilkan.**
- **Simpan:** `hasil/tahap09_latih.parquet`, `hasil/tahap09_validasi.parquet`, `hasil/tahap09_uji.parquet`, dan `hasil/tahap09_anotasi.json` (κ dan tafsirannya).

### Tahap 10 — Eksplorasi data

- **Dasar:** Bab III tahap 10. **Hanya data latih dan validasi.**
- **Proses dan tampilan:**
  - tabel proporsi kelas label awal vs label akhir per bagian, beserta jumlah label yang berubah;
  - kelompok panjang × label akhir, dan median jumlah kata per kelas;
  - 10 kata tersering per kelas pada data latih (kata dalam `KATA_FUNGSI` diabaikan, hanya untuk tampilan);
  - acuan minimum *(program saja)*: `DummyClassifier(strategy="most_frequent")` dilatih pada `label_akhir` data latih, lalu dihitung akurasi dan *Macro F1* validasinya.
- **Gambar:** `bab4/gambar/tahap10_proporsi_label.png`, batang proporsi kelas sebelum dan sesudah anotasi untuk latih dan validasi.
- **Simpan:** `bab4/tabel/tahap10_*.csv` dan `hasil/tahap10_acuan.json`.

### Tahap 11 — *Fine-tuning* skema *default*

- **Dasar:** Bab III tahap 11; Tabel 3.1.
- **Proses:** `fitness(2e-5, 0.1, 0.01)`.
- **Tampilkan:**
  - T dan T_w;
  - satu baris per *seed*: *Macro F1* per *epoch*, *epoch* terbaik, dan durasi;
  - *fitness*;
  - perkiraan waktu Tahap 12 dan 13 (durasi rata-rata × 100 pelatihan).
- **Target:** T = 764, T_w = 76, dan *fitness* jauh di atas acuan minimum validasi.
- **Simpan:** `hasil/tahap11_default.json`.

### Tahap 12 — *Random search*

- **Dasar:** Bab III tahap 12; pers. (2.27).
- **Proses:**
  - `rng = default_rng(2024)`, lalu `F = rng.random((50, 3))`, didekode (Bagian 4);
  - evaluasi `fitness()` untuk k = 0–49 secara berurutan;
  - simpan *state* setelah setiap kandidat, dan lanjutkan dari *state* jika berkasnya sudah ada.
- **Tampilkan:**
  - selama berjalan, satu baris per kandidat: k, lr, wr, wd, skor42, skor43, *fitness*, terbaik sejauh ini;
  - di akhir: 5 kandidat teratas, kandidat terpilih (argmax; kalau seri pilih indeks terkecil), dan selisihnya dengan *default*.
- **Cek:** 50 kandidat dan 100 pelatihan selesai; semua nilai di dalam rentang.
- **Simpan:** `log/rs_state.json`, `hasil/tahap12_rs.csv`, `bab4/tabel/tahap12_rs_top5.csv`.

### Tahap 13.1 — PSO: uji fungsi lalu jalankan

- **Dasar:** Bab III tahap 13; Algoritma 2.1; pers. (2.29)–(2.33); Tabel 3.2.
- **Uji fungsi pembaruan** dengan contoh Bab II 2.10.6 (Tabel 2.17): fungsi φ(λ) = −(λ − 3)² + 10 pada [0, 6], ω = 0,729, c₁ = c₂ = 1,49445, V_max = 6, posisi awal [1,0; 4,0], kecepatan awal [0,5; −0,5].
  - iterasi 1 dengan r₁ = [0,4; 0,6] dan r₂ = [0,95; 0,3] harus menghasilkan posisi [5,6237; 3,6355];
  - iterasi 2 dengan r₁ = [0,7; 0,2] dan r₂ = [0,5; 0,8] harus menghasilkan posisi [2,6718; 3,3698] dan *fitness* *global best* 9,8923;
  - toleransi 5e-4, karena Bab II membulatkan nilai antara ke 4 desimal.
- **Jalankan PSO** (pembaruan serentak, tanpa kriteria berhenti lain):

```
rng = default_rng(7)
X = rng.uniform(0, 1, (10, 3));  V = rng.uniform(-1, 1, (10, 3))
evaluasi fitness 10 posisi → P = X.copy(), fP = fitness, g = P[argmax fP]
untuk t = 1..4:
    r1 = rng.random((10, 3));  r2 = rng.random((10, 3))
    V = 0.729*V + 1.49445*r1*(P - X) + 1.49445*r2*(g - X)    # pers. (2.30)
    V = clip(V, -1, 1)                                          # pers. (2.32)
    X = X + V                                                   # pers. (2.31)
    luar = (X < 0) | (X > 1);  X = clip(X, 0, 1);  V[luar] = 0  # pers. (2.33)
    evaluasi fitness 10 posisi baru
    P_i ← X_i jika fitness baru > fP_i (lebih besar ketat);  g = P[argmax fP]   # pers. (2.29)
```

  - *State* (X, V, P, fP, g, iterasi, partikel terakhir, dan `rng.bit_generator.state`) disimpan setelah setiap evaluasi.
- **Tampilkan:**
  - satu baris per evaluasi: iterasi, partikel, lr, wr, wd, *fitness*, p_i diperbarui?, kena dinding?;
  - satu baris ringkasan per iterasi.
- **Cek:**
  - 50 evaluasi selesai;
  - *global best* tidak pernah turun;
  - semua posisi di [0, 1] dan semua |V| ≤ 1.
- **Simpan:** `log/pso_state.json` dan `hasil/tahap13_pso.csv`.

### Tahap 13.2 — Ringkasan PSO dan konvergensi

- **Tampilkan:** *global best* dalam nilai asli (lr, wr, wd), selisihnya dengan *default* dan *random search*, serta tabel per iterasi.
- **Gambar:** `bab4/gambar/tahap13_konvergensi.png`, kurva "terbaik sejauh ini" terhadap jumlah evaluasi untuk RS dan PSO, ditambah garis mendatar *default*.
- **Simpan:** `bab4/tabel/tahap13_iterasi.csv`.

### Tahap 13.3 — Ragam antar-*seed* *(program saja)*

- **Proses:** dari 100 konfigurasi (50 RS + 50 PSO):
  - d_k = skor42 − skor43, s_k² = d_k²/2, s_gab = √(rata-rata s_k²);
  - SE_selisih = s_gab;
  - zona kebetulan = ±2·SE.
- **Tampilkan:** s_gab, SE, dan zona; tabel 3 selisih *fitness* (RS − *default*, PSO − *default*, PSO − RS) dengan keterangan di dalam atau di luar zona; satu kalimat bahwa ini pemeriksaan awal di data validasi.
- **Simpan:** `hasil/tahap13_ragam.json`.

### Tahap 14 — Pelatihan model akhir

- **Dasar:** Bab III tahap 14.
- **Proses:**
  - latih 3 skema (*default*, terbaik RS, *global best* PSO) × *seed* 42, 43, 44 = 9 pelatihan, dengan *early stopping*;
  - simpan peluang validasi dan uji dari *epoch* terbaik ke `hasil/tahap14_prediksi.parquet`;
  - **tidak ada metrik data uji yang dihitung atau dicetak;**
  - bobot model tidak disimpan.
- **Cek determinisme:** skor *seed* 42 dan 43 harus sama dengan yang tercatat di Tahap 11–13 (selisih ≤ 1e-4). Kalau berbeda, berhenti dan laporkan.
- **Tampilkan:** tabel 9 baris (skema, *seed*, lr, wr, wd, *epoch* terbaik, *Macro F1* validasi) dan hasil cek determinisme.
- **Simpan:** `hasil/tahap14_prediksi.parquet` dan `hasil/tahap14_model.json`.

### Tahap 15.1 — Evaluasi per model dan per skema

- **Dasar:** Bab III tahap 15; pers. (2.36)–(2.40). Label acuan: `label_akhir` data uji, yaitu label manual.
- **Proses:**
  - tebakan = argmax peluang;
  - *confusion matrix*, P/R/F1 per kelas, akurasi, dan *Macro F1* per model;
  - rata-rata ± simpangan baku (ddof = 1) per skema;
  - acuan minimum uji *(program saja)* dari `DummyClassifier` yang dilatih pada label akhir data latih.
- **Tampilkan:** (a) tabel 9 model, kolom *Macro F1* saja; (b) rata-rata ± simpangan baku per skema; (c) acuan minimum.
- **Cek:** n = 656; semua skema di atas acuan minimum.
- **Simpan:** `bab4/tabel/tahap15_model.csv`.

### Tahap 15.2 — Ansambel

- **Proses:**
  - ansambel suara terbanyak (pers. 2.23, minimal 2 dari 3) per skema, beserta *confusion matrix* dan metriknya;
  - analisis sensitivitas *(program saja)*: *Macro F1* ansambel tanpa 5 `review_id` kembaran-dekat.
- **Tampilkan:** (a) tabel ansambel: akurasi, P/R/F1 per kelas, *Macro F1*; (b) tabel sensitivitas dengan dan tanpa 5 dokumen, beserta selisihnya.
- **Cek:** tidak ada seri.
- **Gambar:** `bab4/gambar/tahap15_cm_ansambel.png`, 3 panel *confusion matrix*.
- **Simpan:** `hasil/tahap15_ansambel.parquet` (`review_id`, label, tebakan ketiga skema) dan `bab4/tabel/tahap15_ansambel.csv`.

### Tahap 15.3 — Kesesuaian label *rating* dengan label manual (data uji)

- **Dasar:** celah penelitian pertama di Bab I (kesesuaian label dengan pelabelan manual); Aralikatte dkk. (2018).
- **Proses:** bandingkan label awal (*rating*) dengan label manual pada 656 dokumen uji, dan hitung persentase sesuai, κ, serta matriks 2 × 2.
- **Tampilkan:** satu tabel kecil dan satu kalimat.
- **Simpan:** `bab4/tabel/tahap15_kesesuaian_label.csv`.

### Tahap 16 — Uji signifikansi statistik

- **Dasar:** Bab III tahap 16; pers. (2.41)–(2.45).
- **Proses:**
  1. Tiga perbandingan: *default* vs RS, *default* vs PSO, RS vs PSO. Untuk setiap pasangan (A, B), δ̂ = *Macro F1*(B) − *Macro F1*(A), dihitung pada tebakan ansambel.
  2. `rng = default_rng(42)`, lalu `idx = rng.integers(0, 656, size=(10000, 656))`. **Satu matriks indeks** dipakai untuk ketiga skema dan ketiga perbandingan.
  3. *Macro F1* setiap sampel ulang dihitung secara tervektorisasi dari TP/FP/FN. Verifikasi: (i) contoh Tabel 2.19 (TP 50, FN 10, FP 20, TN 20) menghasilkan 0,6703; (ii) pada data asli, hasilnya sama dengan `sklearn.metrics.f1_score` sampai 1e-12.
  4. α' = 0,05/3, sehingga selang = persentil [0,8333; 99,1667] dari 10.000 nilai δ*.
  5. **Signifikan** jika selang tidak memuat 0.
- **Tampilkan:** satu tabel 3 baris (perbandingan, δ̂, batas bawah, batas atas, signifikan?) dan satu kalimat kesimpulan per baris.
- **Gambar:** `bab4/gambar/tahap16_bootstrap.png`, 3 histogram δ* dengan garis nol dan batas selang.
- **Simpan:** `bab4/tabel/tahap16_signifikansi.csv` dan `hasil/tahap16_bootstrap.npz`.

---

## 7. Kode acuan `lib/wondr_teks.py` (wajib menghasilkan angka yang sama)

Kode ini sudah diverifikasi pada `data_wonder_bybni_3.xlsx`. Hasilnya: perubahan per langkah 55 · 404 · 1.518 · 227 · 634 · 403; 7.141 ulasan → 4.372 kunci → 4.364 dokumen → 4.293 grup → latih 3.051 / validasi 657 / uji 656. **Logika, pola, urutan, dan urutan kunci `PROPORSI` jangan diubah**, karena urutan kunci menentukan pemecah seri saat alokasi.

```python
import re, unicodedata
import numpy as np, pandas as pd

# -- Pola teks (Tahap 1.2 dan Tahap 4 memakai pola yang sama) --
EMO = re.compile("[\U0001F000-\U0001FAFF\U0001F1E6-\U0001F1FF\u2600-\u27BF\u2B00-\u2BFF"
                 "\u2300-\u23FF\u2190-\u21FF\u2900-\u297F\u3030\u303D\u3297\u3299"
                 "\u00A9\u00AE\u203C\u2049\u2122\u2139\u24C2\u25AA-\u25FE\uFE0F\u200D"
                 "\U000E0020-\U000E007F]")
SIMBOL = re.compile("[\u2600-\u27BF\u2B00-\u2BFF\uFE0F\u200D\U0001F3FB-\U0001F3FF]")
HURUF = re.compile(r"([^\W\d_])\1{2,}")      # huruf yang berulang >= 3 kali (hanya huruf)
BACA = re.compile(r"[!?.,;:]{2,}")           # tanda baca berurutan >= 2

LANGKAH = [  # (nama, fungsi) -- urutan Tahap 4, jangan diubah
    ("1. Normalisasi Unicode NFKC", lambda s: unicodedata.normalize("NFKC", s)),
    ("2. Hapus emoji dan simbol", lambda s: SIMBOL.sub(" ", EMO.sub(" ", s))),
    ("3. Case folding", lambda s: s.lower()),
    ("4. Huruf berulang >=3 -> 2", lambda s: HURUF.sub(r"\1\1", s)),
    ("5. Tanda baca berulang -> 1", lambda s: BACA.sub(lambda m: m.group(0)[0], s)),
    ("6. Rapikan spasi", lambda s: re.sub(r"\s+", " ", s).strip()),
]

def bersihkan(s):
    s = str(s)
    for _, f in LANGKAH:
        s = f(s)
    return s

# -- Tahap 5: deduplikasi --
def kunci_dedup(s):
    s = re.sub(r"[^a-z0-9 ]", " ", str(s))   # sisakan a-z, 0-9, spasi
    s = re.sub(r"(.)\1+", r"\1", s)          # karakter berurutan yang sama -> satu
    return re.sub(r"\s+", " ", s).strip()

def deduplikasi(d):
    d = d.copy()
    d["dedup_key"] = d["text_clean"].map(kunci_dedup)
    d = d[d["dedup_key"] != ""]
    suara = (d.groupby(["dedup_key", "label"]).size().unstack(fill_value=0)
               .reindex(columns=[0, 1], fill_value=0))
    suara.columns = ["n_np", "n_p"]
    seri = suara["n_np"] == suara["n_p"]
    suara["mayor"] = (suara["n_p"] > suara["n_np"]).astype(int)
    sah = suara[~seri]
    x = d[d["dedup_key"].isin(sah.index)].copy()
    x = x[x["label"] == x["dedup_key"].map(sah["mayor"])]          # anggota berlabel mayoritas
    frek = x.groupby(["dedup_key", "text_clean"]).size().rename("_f").reset_index()
    x = x.merge(frek, on=["dedup_key", "text_clean"])
    wakil = (x.sort_values(["dedup_key", "_f", "review_id"], ascending=[True, False, True])
               .groupby("dedup_key").head(1)["review_id"])           # teks tersering, lalu id terkecil
    hasil = d[d["review_id"].isin(set(wakil))].copy()
    hasil["label"] = hasil["dedup_key"].map(sah["mayor"]).astype(int)
    return hasil.sort_values("review_id").reset_index(drop=True), suara, seri

# -- Tahap 6: pembagian data berbasis grup --
KATA_FUNGSI = {"dan", "yang", "nya", "ini", "itu", "ya", "sih", "deh", "juga", "aja", "saja", "nih",
               "kok", "dong", "lah", "di", "ke", "dari", "untuk", "pada", "dengan", "yg", "jg", "utk", "dgn"}

def kunci_grup(s):                      # hanya untuk pengelompokan; teks tidak diubah
    k = set(str(s).split())
    inti = k - KATA_FUNGSI
    return " ".join(sorted(inti)) if inti else " ".join(sorted(k))

def kelompok_panjang(n_kata):
    return "1-2" if n_kata <= 2 else ("3-10" if n_kata <= 10 else ">10")

PROPORSI = {"latih": 0.70, "validasi": 0.15, "uji": 0.15}   # urutan kunci menentukan pemecah seri

def bagi_data(dd, seed=42):
    dd = dd.copy()
    dd["group_key"] = dd["text_clean"].map(kunci_grup)
    dd["n_kata"] = dd["text_clean"].str.split().str.len()
    label_id_kecil = dd.sort_values("review_id").groupby("group_key")["label"].first()
    g = (dd.groupby("group_key")
           .agg(n=("review_id", "size"), n_p=("label", "sum"), kata=("n_kata", "median"))
           .reset_index())
    mayor = np.where(g.n_p * 2 > g.n, 1, np.where(g.n_p * 2 < g.n, 0, -1))
    g["label_grup"] = np.where(mayor == -1, g.group_key.map(label_id_kecil).to_numpy(), mayor).astype(int)
    g["kelompok_panjang"] = g["kata"].map(kelompok_panjang)
    rng = np.random.default_rng(seed)                     # satu generator untuk semua strata
    alokasi = {}
    for _, sub in g.groupby(["label_grup", "kelompok_panjang"], sort=True):
        sub = sub.iloc[rng.permutation(len(sub))]
        total = int(sub.n.sum())
        target = {b: p * total for b, p in PROPORSI.items()}
        terisi = {b: 0 for b in PROPORSI}
        for kunci, n in zip(sub.group_key, sub.n):
            b = min(terisi, key=lambda b: terisi[b] / target[b])   # keterisian terkecil
            alokasi[kunci] = b
            terisi[b] += int(n)
    dd["split"] = dd["group_key"].map(alokasi)
    dd["kelompok_panjang"] = dd["n_kata"].map(kelompok_panjang)
    return dd, g
```

---

## 8. Yang TIDAK dikerjakan (sudah diputuskan, jangan ditambahkan)

- Model pembanding TF-IDF + SVM/Regresi Logistik, dan uji McNemar.
- Normalisasi kamus (Salsabila dkk.), aturan Brody, *stemming*, dan *stopword removal* pada teks.
- Pelabelan dengan LLM atau leksikon.
- *Confident learning* pada data uji, *early stopping* pada pelatihan CL, dan pustaka `cleanlab`.
- Analisis daya statistik (Δ_min, n_perlu). Sudah dihapus dari naskah.
- Bobot kelas pada *loss*, penyetelan ambang keputusan, ansambel rata-rata peluang, *seed* model akhir lebih dari 3, dan pelatihan ulang pada gabungan latih + validasi.
- Perbaikan kunci grup. Pembagian data dipertahankan; dampak kembaran-dekat cukup ditunjukkan lewat analisis sensitivitas di Tahap 15.2.

Kalau kamu menilai salah satu hal di atas penting, sampaikan sebagai saran di akhir balasan. Jangan dimasukkan ke program.

---

## 9. Mulai

Kerjakan **Tahap 0.1 saja** dengan format balasan di Bagian 2, lalu tunggu output saya.

---

## 10. Catatan repositori

- Notebook v3: `skripsi_wondr_v3.ipynb` di akar repo. Setiap subtahap berisi 1 sel markdown (judul, dasar naskah, package) dan 1–2 sel kode.
- `skripsi_wondr_v2.ipynb` adalah program versi sebelumnya (Tahap 0–2 versi lama). Berkas ini tidak diubah lagi dan hanya disimpan sebagai arsip.
- PDF proposal ada di `naskah/Revisi_Proposal_Aas_Bu_Anggi.pdf`, tetapi **tidak** di-*commit* (`naskah/` di `.gitignore`) karena repo publik. Di sesi baru, PDF perlu dilampirkan ulang.
- Dataset dan berkas hasil (`*.xlsx`, `*.parquet`, `*.npz`, `data/`, `hasil/`, `log/`, `bab4/`, `anotasi/`) **tidak** di-*commit* karena memuat kolom `nama` dan teks ulasan (lihat `.gitignore`).
- Cabang kerja: `claude/new-session-xc71do`. Lakukan *commit* dan *push* setelah setiap subtahap.

**Keputusan tambahan.** Kalau bertentangan dengan Bagian 1–9, **Bagian 10 yang berlaku**. Naskah (PDF) sudah dicocokkan dengan prompt pada 21-10-2026.

| No. | Tanggal | Hal | Keputusan | Subtahap |
|---|---|---|---|---|
| 1 | 21-10-2026 | GPU | A100 menggantikan T4 untuk seluruh subtahap. Acuan determinisme di `log/versi_package.json` adalah sesi A100 pertama. Naskah Bab III yang masih menulis T4 perlu disesuaikan | 0.1, semua |
| 2 | 21-10-2026 | KT-1 log versi | `log/versi_package.json` menyimpan `pertama` (sesi A100 pertama) dan `terbaru`; ⚠️ bila versi torch/transformers/tokenizers atau GPU berubah | 0.1 |
| 3 | 21-10-2026 | KT-2 determinisme | `atur_seed()` juga menjalankan `torch.use_deterministic_algorithms(True)` | 0.2, pelatihan |
| 4 | 21-10-2026 | KT-3 TF32 | `atur_seed()` mematikan TF32: `torch.backends.cuda.matmul.allow_tf32 = False` dan `torch.backends.cudnn.allow_tf32 = False`. Dipakai API lama karena di torch 2.11 API ini ikut mengatur API baru (`fp32_precision`); API baru saja tidak mematikan TF32 pada konvolusi/RNN cuDNN | 0.2, pelatihan |
| 5 | 21-10-2026 | KT-4 susunan 0.2 | Fungsi bantu ditulis ke `lib/bantu.py` (`%%writefile`), supaya modul `lib/` lain (misalnya `wondr_latih.py`) bisa meng-*import* `atur_seed()`. Sel 2/2 berisi `PATH`, `CFG`, dan `TARGET` (sekitar 75 baris, hampir semuanya data konfigurasi) | 0.2 |
| 6 | 21-10-2026 | Nomor tahap | Nomor tahap utama mengikuti Bab III naskah (15 tahap). Tahap deskriptif yang tidak ada di naskah tetap dijalankan sebagai subtahap *program saja* (jawaban pengguna). Eksplorasi Data (Tahap 10 di Bagian 5–6) menjadi **9.4**; peta lengkap di bawah | semua |
| 7 | 21-10-2026 | `review_id` | Dibuat di **Tahap 2** (Seleksi Atribut) sesuai Bab III tahap 2; Tahap 1.1 hanya membuang kolom `Unnamed*`. Angka tidak berubah. *Rekomendasi yang mengikuti naskah; pertanyaannya tidak dijawab sehingga dipakai sebagai bawaan (sama dengan keputusan v2 no. 6)* | 1.1, 2 |
| 8 | 21-10-2026 | *Global best* PSO | Mengikuti pers. (2.29) dan Algoritma 2.1: λS diperbarui per partikel sesuai urutan evaluasi, hanya jika lebih besar ketat; kalau seri posisi lama dipertahankan. Menggantikan `g = P[argmax fP]`. *Bawaan yang mengikuti naskah (sama dengan keputusan v2 no. 4)* | 12.1 |
| 9 | 21-10-2026 | Arah δ | δ = skema baru − skema lama: RS − *default*, PSO − *default*, PSO − RS; pada (2.43) A dibaca sebagai skema yang lebih baru. Keputusan signifikan tidak berubah. *Bawaan (sama dengan keputusan v2 no. 2)* | 15 |
| 10 | 21-10-2026 | Kunci seri deduplikasi | Tetap **dikeluarkan** sesuai Bab III tahap 5 dan Tabel 2.11 (pilihan pengguna). Alternatif mempertahankan lalu menganotasi ditolak karena 7 grup tambahan menggeser pengacak `bagi_data()` sehingga seluruh pembagian berubah (kembaran-dekat tidak lagi di uji). Tahap 5 menampilkan tabel 8 kunci seri (16 ulasan) sebagai *program saja*: bukti *rating* tidak sesuai isi teks | 5 |
| 11 | 21-10-2026 | KT-5 *weight decay* | Tidak dikenakan pada bias dan `LayerNorm.weight` (BERT asli). Naskah 2.8.2 perlu satu kalimat karena (2.18) menulis θ umum | 8.1, pelatihan |
| 12 | 21-10-2026 | KT-6 kepala klasifikasi | `AutoModelForSequenceClassification` bawaan: *pooler* (dense + tanh) hasil prapelatihan → *dropout* 0,1 → lapisan klasifikasi. Parameter baru hanya `classifier.weight`/`bias` (dicek di 8.1). Naskah (2.14) perlu keterangan bahwa h[CLS] adalah keluaran *pooler* | 8.1, pelatihan |
| 13 | 21-10-2026 | KT-7 teknis pelatihan | `wl.pakai_konfig(CFG)` di setiap sel pelatihan (L_max dari `tahap07_tokenisasi.json`); *batch* evaluasi = B; FP32 penuh; `num_workers=0`; `fitness()` memakai `label_akhir`; tebakan = argmax (seri → kelas 0); uji cepat 8.1 mengecek dua pelatihan *seed* sama identik. SDPA bawaan dipertahankan: di torch 2.11 mode deterministik menonaktifkan SDPA cuDNN dan memaksa *backward* *memory-efficient* deterministik | 8.1, pelatihan |
| 14 | 21-10-2026 | KT-8 berkas lipatan CL | `log/cl_lipatan_k.parquet` = `review_id, bagian, p1, lipatan, n_latih, detik`; ditulis ke `.tmp` lalu `os.replace` (atomik); sel 2 menyusun ulang lipatan untuk mencocokkan isi berkas. *Dianggap disetujui: pengguna menjalankan sel tanpa keberatan* | 8.2 |
| 15 | 21-10-2026 | KT-9 *confident learning* | Dokumen yang tidak lolos ambang mana pun: `kelas_taksiran` = `<NA>` dan bukan kandidat; t_j dihitung dengan `math.fsum`; seri p₀ = p₁ (keduanya lolos) → kelas 0; `tahap08_cj.csv` satu baris per bagian; `per_bagian()` di `lib/wondr_label.py` (disetujui pengguna) | 8.3 |
| 16 | 21-10-2026 | KT-10 lembar anotasi | Teks = `text_clean`; lib baru `lib/wondr_anotasi.py` (petunjuk, `tulis_lembar()` dengan validasi Excel 0/1 dan tidak menimpa lembar yang sudah diisi, `baca_lembar()`); urutan dasar latih → validasi → uji, masing-masing menurut `review_id`, lalu `default_rng(2026)`; `kandidat` = False untuk dokumen uji. *Dianggap disetujui: sel dijalankan dan anotasi dikerjakan* | 9.1 |
| 17 | 22-10-2026 | KT-11 kesepakatan | Lib baru `lib/wondr_kesepakatan.py` (validasi isian, kappa (2.5), tafsiran Tabel 2.10 dengan batas atas inklusif pada κ mentah, tabel kappa, ekspor selisih); isian teks "0"/"1" diterima; simpanan tambahan `bab4/tabel/tahap09_kappa.csv`; `selisih.xlsx` tidak ditimpa bila `label_final` terisi. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 9.2 |
| 18 | 22-10-2026 | KT-12 label akhir | `anotasi/label_manual.csv` (id_anotasi, review_id, bagian, kandidat, label_A, label_B, label_final, label_manual); kolom tambahan `id_anotasi` dan `label_manual` di `tahap09_*.parquet`; uji: `kandidat` = False, `berubah` disimpan tetapi tidak ditampilkan; `label_akhir` int64; κ di json dari `tahap09_kappa.csv`. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 9.3 |
| 19 | 22-10-2026 | Sebaran label uji | Ditampilkan di Audit A Tahap 9.3 atas permintaan pengguna (spesifikasi 9.3 semula menundanya). Tidak dipakai oleh tahap mana pun; metodologi tetap (tanpa bobot kelas, tanpa penyetelan ambang) | 9.3 |
| 20 | 22-10-2026 | Anggaran RS dan PSO | **30 evaluasi** per metode (keputusan pengguna). RS: M = 30, `F = default_rng(2024).random((30, 3))` (sama dengan 30 baris pertama versi 50). PSO: **s = 6, G_max = 4** → 6 × (1 + 4) = 30; parameter lain Tabel 3.2 tetap; larik PSO berukuran (6, 3). Rujukan di Bagian 6 dibaca: RS 30 kandidat / 60 pelatihan; PSO 30 evaluasi; ragam antar-*seed* dari 60 konfigurasi (30 RS + 30 PSO); perkiraan waktu di tahap *default* = durasi rata-rata × 120 pelatihan. Naskah (Bab III tahap 11–12, Tabel 3.2) perlu disesuaikan. **Digantikan no. 25 (24-10-2026)** | 0.2, 11, 12 |
| 21 | 22-10-2026 | Waktu komputasi | Durasi setiap pelatihan (`detik42`, `detik43`) disimpan di CSV RS dan PSO (dan di json skema *default*); Tahap 14 menambah tabel jumlah evaluasi, total waktu, dan rata-rata waktu per evaluasi untuk *default*, RS, dan PSO (Bab III tahap 14). Menggantikan butir "Belum diputuskan" | 10, 11, 12, 14 |
| 22 | 22-10-2026 | KT-13 eksplorasi | Kata = rangkaian huruf (`[^\W\d_]+`), angka dan tanda baca diabaikan, `KATA_FUNGSI` (25 kata) diabaikan; frekuensi = jumlah kemunculan (bukan jumlah dokumen); hanya data latih. Gambar 4 batang bertumpuk 100%; tiga CSV `tahap09_eksplorasi_*`; `DummyClassifier` dengan fitur nol. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 9.4 |
| 23 | 22-10-2026 | KT-14 *random search* | Lib baru `lib/wondr_optim.py` (dekode, baca/simpan *state* atomik, ringkasan *fitness*); `rs_state.json` = F + baris per kandidat (k, f1–f3, lr, wr, wd, fitness, skor/epoch/detik per *seed*); *state* dari konfigurasi lain ditolak. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 11 |
| 24 | 22-10-2026 | KT-15 PSO | Lib baru `lib/wondr_pso.py` (`langkah()` = (2.30)→(2.32)→(2.31)→(2.33) serentak; `perbarui_terbaik()` = (2.29) per partikel, lebih besar ketat; `contoh_tabel_2_17()`; `jalankan()`); ruang [0, 1]³ didekode `wo.dekode`; r₁, r₂ = `rng.random((s, 3))` per iterasi; `pso_state.json` = s, G_max, seed, X0, t, i, X, V, P, fP (`null` = −∞), g, fg, dinding, `rng.bit_generator.state`, log, iterasi; disimpan setelah setiap evaluasi dan setiap pembaruan; *state* dari konfigurasi lain ditolak; kolom `dinding` = ada dimensi partikel itu yang keluar ruang pada pembaruan yang menghasilkan posisinya. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 12.1 |
| 25 | 24-10-2026 | Anggaran kembali 50 | **50 evaluasi** per metode (keputusan pengguna), menggantikan no. 20 dan kembali sesuai naskah (Bab III tahap 11–12, Tabel 3.2): RS M = 50, `F = default_rng(2024).random((50, 3))`; PSO s = 10, G_max = 4 → 10 × (1 + 4) = 50; `TARGET` 11 = 50 kandidat / 100 pelatihan, 12.1 = 50 evaluasi; ragam antar-*seed* dari 100 konfigurasi; perkiraan waktu = durasi rata-rata × 200 pelatihan. 30 baris pertama F RS sama dengan versi 30 (dicek); PSO s = 10 mengubah V₀ dan seluruh lintasan (X₀ 6 partikel pertama sama), sehingga PSO diulang dari awal. Hasil 30 evaluasi dipindah ke `log/arsip_anggaran30/` (salin → cocokkan SHA-256 → hapus asal; sel persiapan sekali jalan sebelum Tahap 11). RS diulang 50 dari awal (pilihan B pengguna; kode tidak berubah) | 0.2, 10, 11, 12 |
| 26 | 24-10-2026 | Penutupan Tahap 11–12.1 | **Rancangan tanpa perubahan (pilihan 1 dari opsi perbaikan 12.1)**: semua hasil asli RS 50 dan PSO 50 (2 *seed* per evaluasi) dipertahankan; tidak ada penyetelan ulang PSO setelah melihat hasil; lanjut ke 12.2 sesuai rancangan (keputusan pengguna). Ditutup dengan catatan keterbatasan lingkungan (lihat status 12.1) | 11, 12 |
| 27 | 24-10-2026 | Penulisan kesimpulan RS vs PSO | Hasil **tidak boleh** ditulis sebagai bukti bahwa PSO selalu kalah dari RS. Kesimpulan yang didukung data hanya: *dalam eksperimen ini, dengan anggaran 50 evaluasi per metode, fitness validasi terbaik RS (0,920759) lebih tinggi daripada PSO (0,918540)*; selisih −0,002219 setara SE *fitness* 2 *seed* (≈ 0,0023) dan bersifat deskriptif; keputusan statistik hanya dari Tahap 15 pada data uji (keputusan pengguna) | 12.2, 13–15, naskah |
| 28 | 24-10-2026 | KT-16 pembacaan CSV | CSV hasil yang nilainya dibandingkan **persis** dengan *state* JSON dibaca dengan `pd.read_csv(…, float_precision="round_trip")`. Pembaca bawaan pandas dapat bergeser 1 ULP (teks CSV sama digit demi digit dengan *state*; audit koreksi 12.2: PSO 28 baris, RS 26 baris). Diterapkan pada sel 1/2 Tahap 12.2 (dua baris `read_csv`); sel 2/2 tidak diubah karena hanya membandingkan CSV dengan CSV dari pembaca yang sama. Berlaku juga untuk tahap berikutnya. *Disetujui pengguna* | 12.2, berikutnya |
| 29 | 24-10-2026 | KT-17 model akhir | Skema dari *state* persis: RS = argmax *fitness* `rs_state.json` (seri → k terkecil; k 33), PSO = baris pertama log dengan *fitness* = fg (t 4 p 9); `log/tahap13_state.json` per pelatihan (atomik, dapat dilanjutkan, ditolak bila skema/acuan/`review_id` berbeda; mencatat GPU, Python, torch, transformers, tokenizers, waktu); uji dibaca hanya `review_id`/`text_clean`; sel 1 berhenti bila selisih *seed* 42/43 > 1e-4; sel 2 menulis `tahap13_prediksi.parquet` (bentuk panjang `review_id, bagian, skema, seed, p1`) dan `tahap13_model.json` hanya bila 9 pelatihan lengkap dan determinisme lolos; tabel waktu komputasi di Tahap 14. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 13 |
| 30 | 24-10-2026 | KT-18 evaluasi | Tabel jumlah evaluasi dan waktu komputasi (keputusan no. 21) = sel 2/2 Tahap 14.1 → `bab4/tabel/tahap14_waktu.csv` (1 evaluasi = *fitness* = 2 pelatihan; waktu = jumlah durasi pelatihan tercatat); tabel 9 model ditampilkan sebagai kisi skema × *seed* + rata-rata ± sb; `tahap14_model.csv` memuat TN/FP/FN/TP, akurasi, P/R/F1 per kelas, *Macro F1*; acuan minimum uji dengan fitur nol (seperti KT-13). *Dianggap disetujui: sel dijalankan tanpa keberatan* | 14.1 |
| 31 | 25-10-2026 | KT-19 ansambel | `hasil/tahap14_ansambel.parquet` = `review_id, label, default, rs, pso` (label = `label_akhir` uji; kolom skema = tebakan ansambel 0/1); `bab4/tabel/tahap14_ansambel.csv` satu baris per skema (TN/FP/FN/TP, akurasi, P/R/F1 per kelas, *Macro F1*, `n_tanpa5`, `benar_dari5`, `macro_f1_tanpa5`, `selisih_tanpa5` = tanpa 5 − dengan 5); gambar mengikuti Tabel 2.19 (baris aktual, kolom prediksi, Positif lalu Non-Positif), skala warna bersama dari ramp biru berurutan palet acuan, angka di setiap sel. *Dianggap disetujui: sel dijalankan tanpa keberatan* | 14.2 |

**Peta nomor tahap (Bagian 5–6 → program = Bab III).** Awalan nama berkas `tahapXX_` mengikuti nomor baru. Rujukan nomor di Bagian 2–6 (larangan, subtahap GPU, cek determinisme) dibaca dengan peta ini; misalnya "*random search*/PSO sebelum Tahap 11 selesai" menjadi Tahap 10, dan subtahap GPU lama menjadi 8.2, 10, 11, 12.1, 13.

| Bagian 5–6 | Program | Berkas keluaran |
|---|---|---|
| 10 Eksplorasi Data | **9.4** (*program saja*) | `bab4/tabel/tahap09_eksplorasi_*.csv`, `bab4/gambar/tahap09_proporsi_label.png`, `hasil/tahap09_acuan.json` |
| 11 *Fine-tuning* skema *default* | **10** | `hasil/tahap10_default.json` |
| 12 *Random search* | **11** | `log/rs_state.json`, `hasil/tahap11_rs.csv`, `bab4/tabel/tahap11_rs_top5.csv` |
| 13.1 / 13.2 / 13.3 PSO | **12.1 / 12.2 / 12.3** | `log/pso_state.json`, `hasil/tahap12_pso.csv`, `bab4/tabel/tahap12_iterasi.csv`, `bab4/gambar/tahap12_konvergensi.png`, `hasil/tahap12_ragam.json` |
| 14 Pelatihan model akhir | **13** | `hasil/tahap13_prediksi.parquet`, `hasil/tahap13_model.json` |
| 15.1 / 15.2 / 15.3 Evaluasi | **14.1 / 14.2 / 14.3** | `bab4/tabel/tahap14_*.csv`, `hasil/tahap14_ansambel.parquet`, `bab4/gambar/tahap14_cm_ansambel.png` |
| 16 Uji signifikansi | **15** | `bab4/tabel/tahap15_signifikansi.csv`, `hasil/tahap15_bootstrap.npz`, `bab4/gambar/tahap15_bootstrap.png` |

**Belum diputuskan:** tidak ada (waktu komputasi diputuskan di no. 21).

**Catatan untuk naskah (program tetap):**
- Bab III: "enam belas tahap" → "lima belas tahap"; tahap 10 masih menulis GPU NVIDIA T4 → A100.
- Bab I: kontribusi "pernyataan batas selisih yang dapat dideteksi oleh ukuran data" masih tertulis, padahal analisis daya sudah dihapus.
- Tabel 2.2–2.3: simbol bintang tercetak sebagai ⋆ (U+22C6, *star operator*), yang **tidak** dihapus oleh pola `EMO`/`SIMBOL`; gunakan ★ (U+2605), misalnya `$\bigstar$`.
- Bab III tahap 14 belum menyebut kesesuaian label *rating* dengan label manual pada data uji (subtahap 14.3).
- Bab II 2.3.4: kandidat = unsur di luar diagonal *confident joint* tanpa kalibrasi, yaitu metode C_{ỹ,y*} pada Northcutt dkk. (2021), bukan C_confusion (argmax) atau *prune by noise rate*. Dokumen yang tidak lolos ambang mana pun (latih 221, validasi 98) tidak dihitung dan tetap memakai label awal; layak disebut di Bab IV.
- Tahap 9.3 (untuk Bab IV/V): proporsi Non-Positif label akhir latih 48,1% dan validasi 47,8% (label *rating* kecuali kandidat), sedangkan uji 59,9% (seluruhnya manual; label awal uji 42,7%). Pedoman anotasi lebih ketat daripada *rating* ≥ 4; model dilatih dengan label yang sebagian besar berbasis *rating* dan diuji dengan label manual. Perbandingan antarskema tetap adil (data sama).
- Tahap 9.4 (bila dipakai di Bab IV): daftar kata tersering = frekuensi kemunculan setelah mengabaikan 25 `KATA_FUNGSI` saja (bukan penghapusan *stopword* umum; teks masukan model tidak diubah). Keterangan gambar proporsi perlu menyebut bahwa "label akhir" latih/validasi hanya mengganti label kandidat. Panjang ulasan sangat terkait kelas (>10 kata: 79% Non-Positif; median 15 vs 4 kata).
- Tahap 12.1–12.2 (untuk Bab IV/V): *global best* PSO 0,918540 di lr 1,076e-05 (dekat batas bawah), wilayah yang sama dengan jalan 30 evaluasi yang diarsipkan (0,9185 di lr 1,00e-05); terbaik RS 0,920759 di dalam ruang (lr 2,71e-05). Skor per *seed* terbaik PSO 0,9223 / 0,9147 (tidak stabil), terbaik RS 0,9208 / 0,9207 (stabil). Selisih PSO − RS −0,0022 setara SE *fitness* 2 *seed* (≈ 0,0023), sehingga di validasi tidak ada metode yang jelas unggul; uji formal di Tahap 15. Faktor yang dibahas sebagai keterbatasan: *fitness* berderau (2 *seed*), *winner's curse*, dinding penyerap (2.33) + V_max = lebar ruang (26 dari 40 posisi hasil pembaruan tepat di batas; 3 di antaranya mendarat tepat di batas seberang dengan |V| = 1), dan hanya 4 iterasi pembaruan. Ruang pencarian dan parameter Tabel 3.2 tidak diubah setelah melihat hasil **Rumusan kesimpulan (keputusan no. 27):** "dalam eksperimen ini, dengan anggaran 50 evaluasi per metode, *fitness* validasi terbaik RS lebih tinggi daripada PSO"; bukan "PSO selalu kalah".
- Bab III/IV (keterbatasan lingkungan): versi paket dan model GPU tidak dicatat per jalan RS/PSO (log hanya sesi A100 pertama dan sesi terakhir yang menjalankan Tahap 0.1). Kesetaraan lingkungan didukung reproduksi bit per bit 72 pelatihan lintas sesi dan tangkapan layar "A100 High-RAM"; untuk tahap GPU berikutnya sebaiknya versi dan GPU dicatat di berkas hasil.
- Tahap 13 (untuk Bab IV/V): *seed* 44, yang tidak ikut memilih konfigurasi, memberi *Macro F1* validasi *default* 0,9163 · RS 0,9117 · PSO 0,9087, sedangkan *seed* 42/43 (dipakai memilih) RS 0,9208/0,9207 dan PSO 0,9223/0,9147; pola ini sesuai *winner's curse* (Bab III tahap 13 sendiri menyebut *fitness* terbaik cenderung menaksir terlalu tinggi). Bersifat deskriptif (satu *seed*, data validasi).
- Tahap 14.2 (untuk Bab IV/V): galat ansambel didominasi FP, yaitu dokumen yang dilabeli Non-Positif oleh anotator tetapi ditebak Positif (53–58 dokumen), sedangkan FN hanya 1–2; konsisten dengan pergeseran label (latih sebagian besar label *rating*, uji label manual yang lebih ketat). RS dan PSO berbeda 1 dokumen pada data uji; urutan RS/PSO berbalik dibanding validasi (deskriptif; keputusan statistik di Tahap 15).
- Dibahas di Tahap 8.1: pers. (2.14) memakai h[CLS] langsung, sedangkan `AutoModelForSequenceClassification` melewatkannya ke *pooler* (dense + tanh hasil prapelatihan); pers. (2.18) menulis *weight decay* untuk seluruh θ.

| Subtahap | Status |
|---|---|
| 0.1 | ✅ Selesai (revisi 1, A100). Semua 4 cek ✅. Revisi 2 (Tahap 7): log HF Hub disenyapkan; versi `huggingface-hub` ikut dicatat. **Sesi 24-10-2026 sebelum Tahap 13** (output ditempel): Python 3.13.16 · NVIDIA A100-SXM4-80GB 79,3 GB · CUDA 13.0 · cuDNN 92700 · torch 2.11.0+cu130 · transformers 5.20.2 · tokenizers 0.23.3 · huggingface-hub 1.33.0 · package lain sama; 3 cek ✅ + ⚠️ versi/GPU berubah (transformers, GPU) terhadap `pertama`. Diff wheel 5.20.1 → 5.20.2 (19 berkas): `models/bert` tidak berubah; `masking_utils.py` hanya `eager_mask` (`torch.tensor(0.0)` → `torch.zeros(())`, nilai sama; model memakai SDPA/`sdpa_mask` yang tidak berubah); `pytorch_utils.py` hanya `move_to_device` (tipe dict); sisanya model lain dan versi. Dampak GPU 40 → 80 GB hanya dapat diuji empiris → sel cek determinisme Tahap 13 |
| 0.2 | ✅ Selesai (revisi 1: kunci `TARGET` mengikuti Bab III). Semua 4 cek ✅. Revisi 2 (22-10-2026): `rs_M` 30, PSO s = 6 (G_max 4), `TARGET` 11 = 30 kandidat / 60 pelatihan, 12.1 = 30 evaluasi; jumlah entri tetap (CFG 31, TARGET 18). Output Colab 4 cek ✅ (CFG 31, TARGET 18). Revisi 3 (24-10-2026, keputusan no. 25): `rs_M` 50, PSO s = 10 (G_max 4), `TARGET` 11 = 50 / 100, 12.1 = 50; sel pengguna masih `TARGET["11"]` 30/60, sudah dikoreksi pengguna; uji lokal 4 cek ✅ (CFG 31, TARGET 18) |
| 1.1 | ✅ Selesai. Semua 6 cek ✅ |
| 1.2 | ✅ Selesai. Semua 13 cek ✅ |
| 2 | ✅ Selesai. Semua 5 cek ✅ |
| 3 | ✅ Selesai. Semua 5 cek ✅ |
| 4 | ✅ Selesai (revisi 1: contoh langkah 4 diperjelas + 2 cek invarian). Semua 7 cek ✅; output Colab sama persis dengan uji lokal |
| 5 | ✅ Selesai. Semua 7 cek ✅ |
| 6.1 | ✅ Output Colab 6/6 ✅; dinyatakan valid setelah audit kebocoran 6.2 (diminta pengguna) |
| 6.2 | ✅ Selesai. 7 + 5 cek ✅; 0 kebocoran. Data uji dikunci |
| 7 | ✅ Selesai. 4 + 3 cek ✅ + pemeriksaan akhir 8/8 ✅: L_max 128 dari latih saja; BertTokenizer 30.521 cocok dengan Tabel 2.14 dan 2.5.2; hanya 2741 terpotong (141 → 128); *padding* per *batch*. Tahap 0.1 revisi 2 menyenyapkan log HF Hub |
| 8.1 | ✅ Selesai (revisi 2). Uji cepat 5 cek ✅ (T/T_w 764/76, parameter baru hanya `classifier.*`, dua pelatihan *seed* sama identik di GPU, peluang di [0, 1], cuda). Uji memori GPU 5 cek ✅: teralokasi 0 → 65 → 65 → 0 MB (65 MB = ruang kerja cuBLAS), dicadangkan 0 → 596 → 596 → 0 MB (cache *allocator* yang ditahan segmen ruang kerja; stabil, bukan kebocoran), tensor CUDA hidup 0, terpakai *driver* 570 → 1.166 → 1.166 → 570 MB. `lib/wondr_latih.py` tidak diubah |
| 8.2 | ✅ Selesai. 6 cek ✅; ukuran lipatan sama persis dengan uji lokal (latih 2.436/2.444/2.440/2.439/2.445 · lipatan 615/607/611/612/606); 6.336 baris. *Macro F1* lipatan vs label awal 0,8335–0,8728 (rentang 0,0393), di luar sampel 0,8510. Durasi 0,7–0,8 menit per lipatan, total 3,7 menit (A100) |
| 8.3 | ✅ Selesai. Output Colab 9 cek ✅: latih t₀ 0,8278 · t₁ 0,8620 · C [[1.034, 148], [190, 1.458]] · kandidat 338 (11,1%) · tidak dihitung 221 (NP 121 / P 100); validasi t₀ 0,8174 · t₁ 0,8500 · C [[204, 20], [37, 298]] · kandidat 57 (8,7%) · tidak dihitung 98 (NP 58 / P 40); lembar anotasi 338 + 57 + 656 = 1.051. Audit (diminta pengguna): kode dan aritmetika keluaran konsisten; Audit A 5/5 ✅ dan Audit B 6/6 ✅ di Colab (t sama persis dengan CSV; pita latih 0,1722–0,8620, validasi 0,1826–0,8500; salah argmax 446 = 338 kandidat + 108 di pita; validasi 101 = 57 + 44). Kolom Audit B "argmax di pita" berarti *salah argmax* di pita (judul kurang jelas). Audit C 3/3 ✅ (tabel silang persis sesuai perkiraan; tidak ada p tepat = ambang; jarak terdekat ke ambang ≥ 2,0e-5 di atas dan ≥ 1,4e-3 di bawah). Pembanding di luar program: kode = `cleanlab` 2.7.1 `filter_by='confident_learning'` (C tanpa kalibrasi, indeks kandidat) pada Tabel 2.9, data tiruan, dan 1.999/2.000 set acak; satu-satunya beda dari toleransi `- 1e-6` cleanlab (dokumen −7,2e-8 dari ambang), tidak berpengaruh pada data ini. KT-9 disetujui; judul kolom Audit B diganti menjadi "salah argmax di pita" |
| 9.1 | ✅ Selesai. Output Colab 4 cek ✅: kandidat latih 338 · kandidat validasi 57 · uji 656 · total 1.051; lembar A = B, teks = `text_clean`, label kosong, 1.051 `review_id` dan teks unik. Uji lokal: urutan identik antar-jalan; teks berawalan '=' / 'null' / angka tetap utuh; isian 0/1 terbaca int; lembar yang sudah diisi tidak ditimpa (`FileExistsError`). Lib baru `lib/wondr_anotasi.py`. KT-10 dianggap disetujui (sel dijalankan dan anotasi dikerjakan). Anotasi kedua anotator selesai 22-10-2026 |
| 9.2 | ✅ Selesai. Output Colab 5 cek ✅. κ: kandidat latih 0,9552 (p_o 0,9941; 2 selisih) · kandidat validasi 1,0000 (0 selisih) · uji 0,9968 (1 selisih) · keseluruhan 0,9929 (p_o 0,9971; p_e 0,5991; 3 selisih); semua "Hampir sempurna". Program valid; kesepakatan sangat tinggi sehingga kemandirian anotasi diminta dikonfirmasi pengguna (sel pemeriksaan tambahan, hanya membaca). Sel pemeriksaan: SHA-256 beda; metadata pembuat dan *lastModifiedBy* sama; disimpan 00:08 dan 00:24 (22-10-2026); p_o 1-2 kata 1,0000 · 3-10 kata 0,9958 · >10 kata 0,9978. Pengguna menyatakan anotasi dilakukan terpisah (22-10-2026) |
| 9.3 | ✅ Selesai. Output Colab 2 + 3 cek ✅: sepakat 1.048 · selisih 3 (diselesaikan 3); latih kandidat 338, berubah P→NP 186 · NP→P 21, Non-Positif 42,7% → 48,1%; validasi kandidat 57, berubah 37 · 5, 42,9% → 47,8%; uji 656 berlabel manual. Audit integritas (diminta pengguna): bukti kode + aritmetika; sel Audit A/B (hanya membaca) diuji lokal. Audit A 3/3 ✅ dan Audit B 5/5 ✅: selisih D0223, D0236 (latih, A 1/B 0) dan D0809 (uji, A 0/B 1) → semua label_final 1; sebaran label akhir latih 1.468/1.583, validasi 314/343, uji 393/263 (59,9% Non-Positif); id uji hanya di berkas yang diizinkan; `fitness()` hanya latih + validasi |
| 9.4 | ✅ Selesai. Output Colab 2 + 2 cek ✅: latih Non-Positif 1.303 → 1.468, validasi 282 → 314; kelompok panjang 428 · 1.437 · 1.186 dan 94 · 308 · 255 (% Non-Positif 11,4 · 33,2 · 79,4 dan 10,6 · 32,8 · 79,6); median kata NP 15 / P 4 (latih), 14 / 4 (validasi); acuan minimum validasi akurasi 0,5221 · *Macro F1* 0,3430 (= 343/1.000). Audit akhir (diminta pengguna): aritmetika output konsisten; sel audit (hanya membaca) diuji lokal + uji negatif. Audit akhir 4/4 ✅ (CSV = hitung ulang; kata = kemunculan, mis. "bisa" 666 kali di 498 dokumen; json 0,5221/0,3430; PNG 300 dpi; tahap09 tidak berubah) |
| 10 | ✅ Selesai. Output Colab 3 cek ✅: T 764 · T_w 76; *seed* 42: 0,9026 · 0,9072 · 0,9085 · 0,9162 (*epoch* terbaik 4); *seed* 43: 0,8867 · 0,9178 · 0,9086 · 0,9101 (*epoch* terbaik 2); **fitness 0,9170** (acuan 0,3430); 0,9 menit per pelatihan; perkiraan RS + PSO 120 pelatihan ≈ 1,9 jam. Dijalankan ulang 24-10-2026 (sesi baru, *Run all*): output sama persis pada 4 desimal (8 nilai *Macro F1* per *epoch*, *epoch* terbaik 4/2, *fitness* 0,9170), sehingga determinisme antar-sesi terjaga pada presisi tampilan; perkiraan RS + PSO 200 pelatihan ≈ 3,1 jam (keputusan no. 25). `tahap10_default.json` lama tertimpa (isi sama); pembanding presisi penuh antar-sesi memakai RS k = 0–29 vs arsip |
| 11 | ✅ Selesai. Output Colab 3 cek ✅: 30 kandidat · 60 pelatihan · 55,2 menit; k 0–3 sama dengan dekode yang diperkirakan. Terpilih **k = 10**: lr 2,12e-05 · wr 0,0528 · wd 0,2667 · skor 0,9193 / 0,9208 · **fitness 0,9200** (*default* 0,9170; selisih +0,0030 ≈ 2 dokumen validasi). Rentang *fitness* 0,9094–0,9200; selisih antar-*seed* per kandidat 0–0,0106 (rata-rata ≈ 0,003), sehingga keunggulan RS masih setara ragam *seed* (diuji di 12.3 dan 15). **Anggaran diubah ke 50 (keputusan no. 25): hasil ini diarsipkan, tahap dijalankan ulang** Persiapan (24-10-2026): sel arsip diuji lokal (pindah 5 berkas dengan SHA-256 sama; jalan ulang tidak mengubah apa pun; bentrok terdeteksi tanpa mengubah berkas); sel 2/2 dengan M = 50 dan *fitness* tiruan: 3 cek ✅ (50 · 100). Output Colab sel arsip 2 cek ✅: `rs_state.json` 30 (SHA-256 `73856168e012`) · `tahap11_rs.csv` 30 (`8579b4d1d0ba`) · `tahap11_rs_top5.csv` 5 (`a44fbaef6dec`) · `pso_state.json` 30 (`2770bb8a7546`) · `tahap12_pso.csv` 30 (`a6713bccb791`), semua dipindah ke `log/arsip_anggaran30/`, 0 tersisa. Sebelumnya sel 2/2 berhenti dengan `ValueError` (*state* lama; pengaman bekerja, tidak ada berkas berubah). **RS 50 (24-10-2026), output Colab 3 cek ✅:** 50 kandidat · 100 pelatihan · 93,8 menit; k 0–29 sama dengan jalan 30 pada 4 desimal (k 1 0,9186, k 10 0,9200, k 19 0,9185; rentang k 0–29 0,9094–0,9200). Terpilih **k = 33**: lr 2,71e-05 · wr 0,0418 · wd 0,1133 · skor 0,9208 / 0,9207 · *epoch* 4/4 · **fitness 0,9208** (*default* 0,9170; selisih +0,0038). Lima teratas: k 33 0,9208 · k 10 0,9200 · k 1 0,9186 · k 19 0,9185 · k 42 0,9185. Rentang *fitness* 0,9048 (k 39)–0,9208; di atas *default* (4 desimal) 5 dari 50; selisih antar-*seed* per kandidat sampai 0,0137 (k 30). Sel pembanding presisi penuh dengan arsip (hanya membaca) diuji lokal + uji negatif (selisih 1e-12 terdeteksi). **Output Colab pembanding 2 cek ✅:** k 0–29 identik dengan arsip pada presisi penuh (maks |selisih| 0 untuk kandidat, skor/*fitness*, *epoch*; durasi baru/lama 1,00–1,03), sehingga determinisme antar-sesi terbukti; *state* = CSV; top-5 benar. Presisi penuh: terbaik 0,920759 (k 33) vs kedua 0,920027 (k 10); di atas *default* 6 dari 50 (satu dari k 4/5/43 yang tampil 0,9170 sedikit di atas), sama dengan *default* 0; maks |skor42 − skor43| 0,0137 (k 30). ✅ Tahap 11 (anggaran 50) selesai **Ditutup 24-10-2026** (audit penutup 11–12.1 dan audit lingkungan; keputusan no. 26) |
| 12.1 | ✅ Selesai (anggaran 50). Jalan 30 evaluasi diarsipkan (*global best* 0,9185, lr 1,00e-05; keputusan no. 25). **PSO 50 (24-10-2026), output Colab 3 cek ✅:** contoh Tabel 2.17 tepat; 50 evaluasi; *global best* tidak turun; posisi di [0, 1], |V| ≤ 1; 93,5 menit. *Global best* **0,918540** (skor 0,922349 / 0,914731) pada evaluasi ke-50 (t 4, partikel 9): lr 1,076e-05 · wr 0,0968 · wd 0,1648; sebelumnya g = t 0 p 8 (0,918519). Maks *fitness* per iterasi 0,9185 · 0,9170 · 0,9178 · 0,9170 · 0,9185; dimensi dinding 14 · 3 · 9 · 3; maks |v| 0,8688 · 1,0000 · 0,6584 · 0,7815; t 0 p 0–5 sama dengan arsip (semula hanya dicek pada maksimum 4 desimal; dibuktikan identik pada presisi penuh oleh Audit C). **Audit A 4/4 ✅:** putar ulang mandiri = log dan *state* (posisi, p_i, g, dinding, gbest, ringkasan iterasi); 26 baris di batas = 23 dinding + 3 *tepat* (t2 p0 wd, t2 p3 wd, t2 p5 wr; |V| = 1); lr = 1e-5 pada 12 baris; kenaikan tipis tetap positif (t2 p3 p_i +0,000026; t3 p5 p_i +0,000017; t4 p9 g +0,000021); tidak ada seri. **Audit B 4/4 ✅:** anggaran 50 · 50 · 100 · 100; *fitness* = rata-rata 2 *seed* di ketiga skema; *default* 0,917006 · RS 0,920759 (k 33) · PSO 0,918540; PSO − RS −0,002219, PSO − *default* +0,001533, RS − *default* +0,003753 (deskriptif); di atas *default* RS 6/50 · PSO 6/50; tidak ada posisi kembar; `fitness()` hanya latih + validasi. Analisis penyebab (diminta pengguna): derau *seed* ≈ sebaran antar-konfigurasi, *winner's curse*, kawanan terkunci di lr rendah, banyak posisi di batas, 4 iterasi; tidak ada kesalahan program. Pilihan perbaikan (1 tanpa perubahan, disarankan / 2 *fitness* 3 *seed* untuk semua metode) belum dijawab **Audit C (diminta pengguna) 2 + 1 cek:** 10 evaluasi per t, 0 baris kosong; p_i/g dari riwayat = *state*; g sebelumnya t 0 p 8 0,918519334 (0,920830166 / 0,916208501) vs g akhir t 4 p 9 0,918539609 (0,922348524 / 0,914730694), selisih 2,028e-05 (lebih besar ketat, bukan seri); t 0 p 0–5 identik dengan arsip 22-10 pada presisi penuh; ⚠️ "lib tidak diubah selama jalan" berasal dari syarat cek yang keliru (membandingkan `wondr_pso.py` dengan mulai RS, padahal RS tidak meng-*import*-nya). Per berkas: `wondr_latih` 05:49 dan `wondr_optim` 05:54 < RS ±05:59; `wondr_pso` 07:35 < PSO ±07:39; SHA-256 ketiganya = satu-satunya versi `%%writefile` di riwayat Git. Kode sel RS yang dijalankan identik dengan repo (ditempel 05:56 UTC). **Audit penutup** dijalankan pengguna (output lengkap tidak ditempel): dilaporkan `torch`/`tokenizers` log `terbaru` = `pertama`, `transformers` dan GPU berbeda. **Keterbatasan lingkungan:** `log/versi_package.json` hanya menyimpan `pertama` (21-10) dan `terbaru` (ditimpa setiap Tahap 0.1), *state* RS/PSO tidak menyimpan versi/GPU, output Tahap 0.1 sesi 24-10 tidak ditempel, sehingga versi `torch`, `transformers`, `tokenizers`, dan model GPU saat RS/PSO **tidak dapat diverifikasi**. Bukti yang ada: tangkapan layar Colab "A100 High-RAM" 24-10 05:53 UTC saat Tahap 10 (*Run all*), durasi per pelatihan setara (Tahap 10 0,9 menit; RS dan PSO 1,9 menit per evaluasi), dan 72 pelatihan (RS k 0–29, PSO t 0 p 0–5) identik bit per bit dengan jalan 22-10. Pemeriksaan sekali atas notebook repo (tanpa output/`executionInfo`; metadata `gpuType` A100 hanya pengaturan) dan transkrip: tidak ada bukti tambahan; output tersimpan di notebook Colab pengguna (`Untitled1.ipynb`) tidak dapat diakses. Sel Audit D dan audit lingkungan disiapkan tetapi tidak dijalankan. **Ditutup 24-10-2026 dengan catatan (keputusan no. 26)** |
| 12.2 | ✅ Selesai. Sel 1/2 (ringkasan, tabel per iterasi + RS terbaik pada evaluasi yang sama → `bab4/tabel/tahap12_iterasi.csv`) dan sel 2/2 (kurva terbaik sejauh ini RS vs PSO + garis *default* → `bab4/gambar/tahap12_konvergensi.png`; palet slot 1–2 tervalidasi) diuji lokal dengan nilai output Colab: 2 + 1 cek ✅. Menunggu output Colab Dilanjutkan setelah penutupan 11–12.1 (keputusan no. 26) **Output Colab:** tabel per iterasi sama persis dengan perkiraan (t 0–4: maks 0,9185 · 0,9170 · 0,9178 · 0,9170 · 0,9185; g baru 4 · 0 · 0 · 0 · 1; RS sejauh ini 0,9186 · 0,9200 · 0,9200 · 0,9208 · 0,9208); *global best* lr 1,08e-05 · wr 0,0968 · wd 0,1648 · *fitness* 0,918540; PSO − *default* 0,001533, PSO − RS −0,002219. Sel 2/2 1 cek ✅ (RS naik 4 kali, terakhir evaluasi ke-34, 0,9208; PSO naik 5 kali, terakhir ke-50, 0,9185; gambar tersimpan). Sel 1/2: ⚠️ "gbest akhir = fg di state" (5 · 50 · ya · tidak). **Audit koreksi (output Colab 3 cek ✅):** teks CSV = *state* 50 · 50; `round_trip` = *state* 50 · 50; pembaca bawaan bergeser 1 ULP pada 28 baris PSO dan 26 baris RS (maks *state* PSO 0,9185396093525671 → 0,9185396093525672); kenaikan terbaik-sejauh-ini dan argmax tidak berubah. **Perbaikan KT-16** (sel 1/2, dua baris `read_csv`): diuji lokal sebelum ⚠️ → sesudah ✅, teks ringkasan identik, `tahap12_iterasi.csv` hanya berubah ≤ 1 ULP. Menunggu output sel 1/2 yang diperbaiki **Output Colab sel 1/2 (perbaikan KT-16) 2 cek ✅:** angka tampilan identik dengan sebelum perbaikan; cek pertama 5 · 50 · ya · ya. Sel 2/2 dan PNG tidak dijalankan ulang (tidak berubah) |
| 12.3 | ✅ Selesai (*program saja*). Sel ditulis: s_gab dari 100 konfigurasi, SE selisih = s_gab, zona ±2·SE, tiga selisih *fitness* terbaik dengan keterangan di dalam/luar zona → `hasil/tahap12_ragam.json`; CSV dibaca `round_trip` (KT-16). Diuji lokal: 1 cek ✅; s_gab = akar rata-rata `statistics.variance` per pasangan *seed* (verifikasi independen). **Output Colab 1 cek ✅** (100 · ya · ya): s_gab 0,0032 · SE selisih 0,0032 · zona ±0,0064; RS − *default* 0,003753 · PSO − *default* 0,001533 · PSO − RS −0,002219, ketiganya di dalam zona. **Audit 12.3 (diminta pengguna):** rumus sesuai spesifikasi (s_k² = d_k²/2 = varians sampel pasangan, ddof 1, 1 df; 100 df); zona ±2·SE tidak ada di naskah (Bab II 2.8.6 hanya meminta rata-rata ± simpangan baku antar-*seed*), sehingga diperlakukan sebagai heuristik deskriptif; catatan: *winner's curse*, *seed* 42/43 dipakai bersama semua konfigurasi, ragam diasumsikan homogen, ragam *seed* *default* tidak ikut, cek kelengkapan sel 12.3 hanya `isfinite`. Sel audit (hanya membaca: hitung ulang dari *state*, CSV = *state* untuk skor, json = hitung ulang persis, diagnostik) diuji lokal: 3 cek ✅, `log/` dan `hasil/` tidak berubah, 4 uji negatif terdeteksi. **Output Colab sel audit 3 cek ✅:** skor sah 50 · 50; CSV = *state* 50 · 50 (skor42, skor43, fitness); RS k 0–49, PSO (t, p) unik; s_gab (a) = (b) = 0,0032171297047190834 (df 100), zona ±0,006434; *default* 0,9170062489257791 = rata-rata 2 *seed*; RS k 33 0,9207588975839698 dan PSO t 4 p 9 0,9185396093525671 (= fg) tunggal; selisih presisi penuh +0,0037526487 · +0,0015333604 · −0,0022192882 (1,17 · 0,48 · −0,69 SE); `tahap12_ragam.json` = hitung ulang persis (0 nilai berbeda). Diagnostik: s RS 0,003212 · s PSO 0,003222 (rasio 1,003, homogen antarmetode); rata-rata d −0,000692 (SE 0,000452; 1,5 SE, tidak ada geseran tetap yang jelas); s dengan d dipusatkan 0,003196 (−0,7%); maks |d| 0,0137. Dengan ragam *seed* *default* ikut, s_gab 0,003203 (Hitungan; tidak mengubah keterangan zona). SE satu *fitness* = s_gab/√2 = 0,002275, yaitu angka ≈ 0,0023 di keputusan no. 27; SE selisih dua *fitness* = s_gab = 0,0032. **Status audit: LULUS DENGAN CATATAN** (zona = heuristik deskriptif di luar naskah; *winner's curse*; *seed* bersama; bukan uji signifikansi) |
| 13 | Cek determinisme lingkungan sesi ini (tambahan, akibat ⚠️ Tahap 0.1 24-10: transformers 5.20.2, A100-SXM4-80GB): satu pelatihan *default* *seed* 42, kurva *Macro F1* per *epoch* vs `tahap10_default.json` presisi penuh; tidak menyimpan apa pun. Diuji lokal (CPU, *checkpoint* mini): identik, `log/`/`hasil/` tidak berubah; uji negatif (+1e-12, +1e-3, kurva lebih pendek) terdeteksi. **Output Colab cek determinisme 1 cek ✅** (A100-SXM4-80GB · torch 2.11.0+cu130 · transformers 5.20.2 · tokenizers 0.23.3): kurva *default* *seed* 42 identik presisi penuh dengan Tahap 10 (0,9025764596848934 · 0,9071528686768062 · 0,9085433757748989 · 0,916236471152754; *epoch* terbaik 4; 0,9 menit), sehingga perubahan transformers dan GPU tidak mengubah hasil; ⚠️ Tahap 0.1 dinyatakan ✅ valid. **Sel model akhir ditulis** (sel 1/2: 9 pelatihan, *state* `log/tahap13_state.json`, berhenti bila selisih *seed* 42/43 > 1e-4; sel 2/2: cek determinisme, tabel 9 model, `hasil/tahap13_prediksi.parquet` bentuk panjang 11.817 baris + `hasil/tahap13_model.json`, ditulis hanya bila lengkap dan lolos). Usulan KT-17 (skema dari *state* persis; *state* per pelatihan dengan GPU/versi; uji hanya `review_id`/`text_clean`; tabel waktu komputasi ke Tahap 14 sesuai Bab III tahap 14). Diuji lokal (CPU, *checkpoint* mini): 9 pelatihan, determinisme 6/6 identik, 3 cek sel 2 sesuai (⚠️ hanya cuda); uji negatif: acuan berbeda → berhenti di rs *seed* 42, keluaran tidak ditulis; *state* skema lain → `ValueError`; lanjut dari 7 → 9 identik presisi penuh dengan jalan penuh. **Output Colab sel 2/2 3 cek ✅** (sel 1/2 tidak ditempel; *state* lengkap 9): 9 pelatihan · cuda · lingkungan seragam (A100-SXM4-80GB · Python 3.13.16 · torch 2.11.0+cu130 · transformers 5.20.2 · tokenizers 0.23.3); determinisme identik persis 6 dari 6, *epoch* sama 6 dari 6; prediksi 657 · 656 per model. *Macro F1* validasi (*epoch* terbaik): *default* 0,9162 (4) · 0,9178 (2) · 0,9163 (2); RS 0,9208 (4) · 0,9207 (4) · 0,9117 (1); PSO 0,9223 (4) · 0,9147 (3) · 0,9087 (1); 0,7–1,1 menit per pelatihan (≈ 8,7 menit). Rata-rata validasi 3 *seed* (Hitungan, 4 desimal) *default* 0,9168 · RS 0,9177 · PSO 0,9152. **✅ Selesai** |
| 14.1 | Sel 1/2 (per model: TN/FP/FN/TP, akurasi, P/R/F1 per kelas, *Macro F1* → `bab4/tabel/tahap14_model.csv`; kisi skema × *seed* + rata-rata ± sb; acuan minimum uji) dan sel 2/2 (jumlah evaluasi dan waktu komputasi → `bab4/tabel/tahap14_waktu.csv`) ditulis. Diuji lokal (peluang tiruan): 2 + 1 cek ✅; matriks dan metrik 9 model sama persis dengan hitung manual dari TP/FP/FN/TN; seri 0,5 → kelas 0. Perkiraan acuan minimum uji: selalu Positif (latih 1.583 > 1.468) → akurasi 0,4009 · *Macro F1* 0,2862 (Hitungan dari uji 263 P / 393 NP). **Output Colab 2 + 1 cek ✅:** *Macro F1* uji *default* 0,8999 · 0,9044 · 0,9104 (0,9049 ± 0,0053); RS 0,9104 · 0,8923 · 0,9469 (0,9165 ± 0,0278); PSO 0,9058 · 0,9120 · 0,9286 (0,9154 ± 0,0118) (*seed* 42 · 43 · 44; ddof 1 dicek ulang); acuan minimum uji selalu kelas 1 · akurasi 0,4009 · *Macro F1* 0,2862 (sama dengan perkiraan); waktu pencarian *default* 1 evaluasi · 2 pelatihan · 1,8 menit · RS 50 · 100 · 93,8 menit (1,88 per evaluasi) · PSO 50 · 100 · 93,5 menit (1,87). *Seed* 44 tertinggi di uji untuk ketiga skema, padahal terendah di validasi untuk RS/PSO (deskriptif). **✅ Selesai** |
| 14.2 | Sel 1/2 (ansambel suara terbanyak pers. (2.23), metrik per skema, sensitivitas tanpa 5 kembaran-dekat → `hasil/tahap14_ansambel.parquet`, `bab4/tabel/tahap14_ansambel.csv`) dan sel 2/2 (3 panel *confusion matrix* → `bab4/gambar/tahap14_cm_ansambel.png`) ditulis. Diuji lokal (peluang tiruan): 2 + 1 cek ✅; suara, tebakan ansambel, matriks, *Macro F1*, dan sensitivitas sama persis dengan hitung manual; gambar diperiksa visual. **Output Colab 2 + 1 cek ✅:** *Macro F1* ansambel *default* 0,9089 · RS 0,9149 · PSO 0,9165 (akurasi 0,9101 · 0,9162 · 0,9177); matriks (TP/FN/FP/TN) *default* 262/1/58/335 · RS 261/2/53/340 · PSO 262/1/53/340 (jumlah aktual P 263 / NP 393 = Tahap 9.3); seri 0; galat hampir semuanya FP (manual Non-Positif ditebak Positif: 58 · 53 · 53) dengan *recall* Positif 0,9962 · 0,9924 · 0,9962 dan *recall* Non-Positif 0,8524 · 0,8651 · 0,8651; RS dan PSO hanya berbeda 1 dokumen (TP/FN). Sensitivitas tanpa 5 kembaran-dekat: −0,0009 · −0,0008 · −0,0008, kelimanya benar di ketiga skema. **✅ Selesai** |
| 14.3 | Sel ditulis (label awal vs label manual uji: persentase sesuai, κ rumus (2.5) = sklearn, tafsiran Tabel 2.10, matriks 2 × 2 → `bab4/tabel/tahap14_kesesuaian_label.csv`; reuse `wk.tabel_kappa`). Diuji lokal (label tiruan): 2 cek ✅, matriks dan κ sama dengan `crosstab`/`cohen_kappa_score`. Perkiraan dari margin: *rating* P 376 / NP 280, manual P 263 / NP 393 → P→NP − NP→P = 113; sesuai ≤ 543 (82,8%), κ ≤ 0,665 (Hitungan, batas atas bila NP→P = 0). Menunggu output Colab |

**Catatan kerja:** saat menulis berkas lewat alat, urutan `\uXXXX` dapat berubah menjadi karakter aslinya. Setelah menulis sel yang memuat *escape* Unicode, periksa bahwa barisnya ASCII murni (pernah terjadi di Tahap 4 dan sudah diperbaiki).

**Verifikasi lokal dengan data asli (21-10-2026):** `data_wonder_bybni_3.xlsx` dari pengguna (SHA-256 awal `1bbda628c39b0095`; tidak di-*commit*), dijalankan dengan versi numpy/pandas/scikit-learn/scipy/pyarrow/openpyxl/matplotlib yang sama dengan Colab. Sel 1.1: 6/6 cek ✅. Kode acuan Bagian 7: perubahan per langkah 55 · 404 · 1.518 · 227 · 634 · 403; kosong 63 (*rating* 5: 57, 4: 6); 7.141 → 4.372 kunci (8 seri) → 4.364 dokumen → 4.293 grup → latih 3.051 (1.303/1.748) · validasi 657 (282/375) · uji 656 (280/376); kelima kembaran-dekat di uji. Semua cocok dengan target.

**Lingkungan Colab (output Tahap 0.1, 21 Oktober 2026):** Python 3.13.15 · NVIDIA A100-SXM4-40GB 39,5 GB · CUDA 13.0 · cuDNN 92700 · numpy 2.1.3 · pandas 2.2.3 · torch 2.11.0+cu130 · transformers 5.20.1 · tokenizers 0.23.3 · scikit-learn 1.6.1 · scipy 1.16.3 · matplotlib 3.10.0 · openpyxl 3.1.5 · pyarrow 23.0.1.
