# PROMPT — Membangun Ulang Program Skripsi (Versi Bersih, Bertahap)

> **Cara pakai:** simpan berkas ini sebagai `CLAUDE.md` di folder proyek (supaya aturannya terbaca di setiap sesi Claude Code), atau tempelkan seluruh isinya sebagai pesan pertama. Lalu ketik: **"Mulai Tahap 0."**

---

## 1. Konteks

Saya Ahmad Agus Subhan, mahasiswa S1 Matematika FMIPA Universitas Brawijaya. Judul skripsi saya:

**"Optimasi Hyperparameter Model IndoBERT Menggunakan Particle Swarm Optimization Disertai Uji Signifikansi Statistik pada Analisis Sentimen Ulasan Aplikasi Wondr by BNI"**

Program lama sudah berjalan, tetapi keluarannya terlalu ramai dan penomoran tahapnya (10a, 10b, 12b, 12c, 14b) tidak sama dengan naskah. Tugasmu adalah **membangun ulang program dari nol** sebagai notebook Google Colab yang bersih. Programnya terdiri atas **Tahap 0 (persiapan) ditambah 16 tahap yang sama persis dengan Bab III**.

Metodologi sudah final dan tertulis di naskah. **Tugasmu menerjemahkan spesifikasi di bawah menjadi kode, bukan merancang ulang metodenya.**

**Lingkungan**
- Google Colab dengan GPU **T4**. Seluruh tahap dijalankan di jenis GPU yang sama.
- Google Drive, folder kerja baru `MyDrive/skripsi_wondr_v2/`. Folder lama `skripsi_wondr_final` tidak boleh ditimpa.
- Data mentah `data_wonder_bybni_3.xlsx` (7.204 baris), saya salin ke `skripsi_wondr_v2/data/`.
- Kamu menulis kodenya, saya menjalankannya di Colab lalu menempelkan output ke sini.
- Setiap tahap juga ditambahkan ke notebook `skripsi_wondr_v2.ipynb` di folder kerja Claude Code, berisi 1 sel markdown (judul dan package) ditambah 1–2 sel kode. Kodenya juga ditampilkan di balasan supaya bisa langsung saya salin ke Colab.

---

## 2. Aturan kerja (wajib)

1. **Satu tahap per balasan**, maksimal 2 sel kode. Setelah memberi satu tahap, berhenti dan tunggu output saya.
2. Sebelum memberi tahap berikutnya, **analisis output** tahap sebelumnya terhadap angka target di Bagian 5.
3. Jika ada error, warning, atau angka yang menyimpang dari target, **jangan lanjut**. Pakai format A–E di bawah.
4. Jangan mengubah metodologi, nama kolom, atau angka konfigurasi. Kalau ada hal teknis yang tidak diatur di sini, sebutkan sebagai **"Keputusan teknis"** beserta alasannya. Untuk tahap yang mahal (Tahap 10–14), minta persetujuan saya dulu sebelum dijalankan.
5. **Larangan:**
   - menghapus data tanpa menampilkan jumlah sebelum dan sesudah;
   - membagi data sebelum *pre-processing* dan deduplikasi selesai;
   - menjalankan *random search* atau PSO sebelum Tahap 10 selesai;
   - memakai data uji untuk keputusan apa pun sebelum Tahap 15.
6. Bahasa balasan: Indonesia yang mudah dipahami, dan setiap istilah teknis diberi arti di dalam kurung. Istilah seperti *random search*, *fine-tuning*, *learning rate*, dan *weight decay* tetap ditulis dalam bahasa Inggris (miring).
7. Setiap angka yang kamu sebut diberi label asal: (Rujukan), (Data), (Hitungan), (Keputusan), (Bawaan), atau (Contoh).
8. Bagian penjelasan cukup 3–6 kalimat. Saya sudah mempelajari teori setiap tahap; yang saya butuhkan adalah program yang benar dan keluaran yang bisa dipertanggungjawabkan.

### Format balasan setiap tahap

```
## Tahap X — [Nama Tahap]

### 0. Package yang digunakan
| Package | Versi | Dipakai untuk apa di tahap ini |
|---|---|---|

### 1. Tujuan
### 2. Penjelasan (alasan dan dampak, singkat)
### 3. Program
### 4. Output yang saya harapkan   ← sebutkan angka target dari Bagian 5
### 5. Interpretasi                 ← tanda normal vs tanda bermasalah

Status: Menunggu output Tahap X dari pengguna.
```

Kolom versi diambil dari `log/versi_package.json` hasil Tahap 0. Modul bawaan Python (`re`, `json`, `math`, `unicodedata`, `time`, `os`) cukup ditulis "bawaan Python".

### Format jika hasil berbeda dari harapan

- **A. Identifikasi masalah**
- **B. Analisis penyebab**: data, *pre-processing*, label, pembagian data, tokenisasi, parameter, atau *bug*; urutkan dari yang paling mungkin
- **C. Verifikasi** dengan kode kecil
- **D. Perbaikan** hanya pada sel yang bermasalah
- **E. Keputusan**: ✅ valid / ⚠️ perlu revisi / ❌ kembali ke tahap sebelumnya

---

## 3. Prinsip "program bersih"

### 3.1 Struktur folder

```
skripsi_wondr_v2/
├── data/          data_wonder_bybni_3.xlsx
├── lib/           wondr_teks.py (dibuat Tahap 5), wondr_latih.py (dibuat Tahap 10)
├── hasil/         berkas antar-tahap: tahapXX_nama.parquet / .json / .csv
├── bab4/tabel/    CSV untuk Bab IV
├── bab4/gambar/   PNG 300 dpi untuk Bab IV
└── log/           versi_package.json, rs_state.json, pso_state.json
```

### 3.2 Bentuk setiap sel tahap

Setiap sel tahap **berdiri sendiri**. Setelah runtime di-*restart*, sel tahap mana pun bisa dijalankan ulang cukup dengan menjalankan sel Tahap 0 lebih dulu.

```python
# ══ Tahap X — Nama Tahap ══
# Package : pandas (olah tabel), re (pola teks), ...   ← daftar package di baris paling atas
import ...
# 1) Baca masukan dari berkas tahap sebelumnya (hasil/...)
# 2) Proses
# 3) Simpan keluaran
# 4) Cetak ringkasan + cek otomatis
```

- Fungsi yang dipakai lebih dari satu tahap ditulis **sekali** ke `lib/` dengan `%%writefile`, di tahap pertama kali fungsi itu dibutuhkan: Tahap 5 untuk fungsi teks, Tahap 10 untuk fungsi pelatihan. Tahap lain cukup meng-*import*-nya.
- Tidak ada variabel yang "dibawa" antar-sel, kecuali yang dibuat Tahap 0 (`PATH`, `CFG`, `TARGET`, dan fungsi bantu). Antar-tahap berkomunikasi lewat berkas di `hasil/`.
- Komentar kode singkat dan berbahasa Indonesia. Setiap angka konfigurasi diberi komentar asalnya.

### 3.3 Keluaran hanya yang dibutuhkan

- **Matikan semua kebisingan di Tahap 0:**
  - `warnings.filterwarnings("ignore")`;
  - `transformers.logging.set_verbosity_error()`;
  - *progress bar* (`tqdm`, `datasets`, Hugging Face Hub) dimatikan;
  - `os.environ["TOKENIZERS_PARALLELISM"] = "false"`;
  - pesan "Some weights … newly initialized" tidak muncul.
- **Jangan mencetak:** `df.head()`, `df.info()`, daftar kolom, isi berkas, atau log per langkah (*step*) pelatihan.
- **Setiap tahap mencetak satu blok ringkasan** dengan bentuk tetap:

```
══════════ TAHAP 5 — PRE-PROCESSING ══════════
Masukan : hasil/tahap04_label.parquet (7.204 baris)
Keluaran: hasil/tahap05_bersih.parquet (7.141 baris)
──────────────────────────────────────────────
[1–2 tabel ringkas yang memang dibutuhkan]
──────────────────────────────────────────────
Cek otomatis:
  ✅ jumlah setelah pre-processing = 7.141 (target 7.141)
Waktu: 3,2 detik
```

- Angka di ringkasan memakai format Indonesia (7.204; 0,8360). Berkas CSV memakai angka mentah.
- **Pelatihan:** satu baris per pelatihan, berisi konfigurasi, *seed*, *epoch* terbaik, *Macro F1* validasi terbaik, dan durasi.
- **Gambar:** hanya yang tercantum di Bagian 5. Disimpan ke `bab4/gambar/` dan ditampilkan satu kali.
- **Cek otomatis tidak memakai `assert`**, supaya program tidak berhenti. Cetak ✅ atau ⚠️ beserta nilai aktual dan targetnya. Angka target ditulis di kamus `TARGET` di Tahap 0.

### 3.4 Keterulangan (*reproducibility*)

- Setiap sumber keacakan memakai *seed* sendiri (lihat Bagian 4).
- Fungsi `atur_seed(s)` menjalankan:
  - `random.seed`, `np.random.seed`, `torch.manual_seed`, `torch.cuda.manual_seed_all`, `transformers.set_seed`;
  - `torch.backends.cudnn.deterministic = True` dan `torch.backends.cudnn.benchmark = False`.
- `os.environ["CUBLAS_WORKSPACE_CONFIG"] = ":4096:8"` diatur di Tahap 0, sebelum torch memakai GPU.
- Tahap 11 dan 12 menyimpan *state* ke `log/` **setelah setiap evaluasi**. Kalau Colab terputus, menjalankan ulang sel yang sama melanjutkan dari evaluasi terakhir, bukan dari awal.
- Tahap 0 menyimpan versi Python, nama GPU, dan versi seluruh package ke `log/versi_package.json`.

---

## 4. Konfigurasi tetap (jangan diubah)

Tulis semua nilai ini dalam satu kamus `CFG` di Tahap 0, dengan komentar asalnya.

| Nama | Nilai | Asal |
|---|---|---|
| *Checkpoint* | `indobenchmark/indobert-base-p1` | Rujukan: Wilie dkk. (2020) |
| Label | `rating` ≥ 4 → 1 (Positif); ≤ 3 → 0 (Non-Positif) | Keputusan: *top-2-box*, Morgan & Rego (2006) |
| Proporsi pembagian | latih 0,70 · validasi 0,15 · uji 0,15 | Keputusan |
| *Seed* pembagian data | 42 | Keputusan |
| Kelompok panjang | 1–2, 3–10, >10 kata | Keputusan (bukti Tahap 2) |
| Panjang masukan | L_max = b·⌈P_q / b⌉, q = 100, b = 32, **dihitung dari data latih saja** | Keputusan (Jalur 2) |
| *Batch size* `B` | 16 | Rujukan: Devlin dkk. (2019) |
| *Epoch* maksimum | 4 | Rujukan: Sun dkk. (2019) |
| *Patience* | 2 | Keputusan |
| *Gradient clipping* | norma maksimum 1,0 | Keputusan |
| *Optimizer* | AdamW, β = (0,9; 0,999), ε = 1e-8 | Bawaan PyTorch |
| Jadwal *learning rate* | naik linear (*warmup*), lalu turun linear ke 0; T = ⌈n_latih / B⌉ × 4; T_w = ⌊ρ·T⌋ | Rujukan (Bab II) |
| Skema *Default* | lr 2e-5 · *warmup ratio* 0,1 · *weight decay* 0,01 | Rujukan: Devlin dkk. (2019); Sun dkk. (2019) |
| Ruang pencarian | lr [1e-5; 5e-5] skala log · wr [0; 0,2] · wd [0; 0,3] | Rujukan: Liu & Wang (2021); Bischl dkk. (2023) |
| Dekode f ∈ [0,1]³ | lr = 1e-5 · 5^f₁ · wr = 0,2·f₂ · wd = 0,3·f₃ | Hitungan |
| *Fitness* | rata-rata *Macro F1* validasi terbaik dari *seed* 42 dan 43 | Keputusan |
| *Random search* | M = 50; `rng = np.random.default_rng(2024)`; `F = rng.random((50, 3))` | Keputusan |
| PSO | s = 10; G_max = 4 (10 × (1 + 4) = 50 evaluasi); ω = 0,729; φp = φg = 1,49445; V_max = 1,0; *seed* 7 | Rujukan: Eberhart & Shi (2000); Clerc & Kennedy (2002); Lorenzo dkk. (2017); Robinson & Rahmat-Samii (2004) |
| Ragam antar-*seed* | K = 100 (50 RS + 50 PSO), J = 2 | Hitungan |
| Model akhir | 3 skema × *seed* 42, 43, 44 | Keputusan |
| Ansambel | suara terbanyak dari 3 model | Keputusan |
| *Bootstrap* | R = 10.000; `default_rng(42)`; persentil; berpasangan | Rujukan: Efron & Tibshirani (1993) |
| Koreksi banyak-uji | Bonferroni, m = 3, α = 0,05 → α' = 0,05/3; selang 98,33% | Rujukan: Dror dkk. (2017; 2018) |
| Analisis daya | daya 0,80; Δ_target = 0,005; Δ_min = (z_{1−α'/2} + z_{0,80}) · SE | Rujukan: Bloom (1995) |
| Kembaran-dekat di data uji | `review_id` 339, 595, 2854, 3946, 4128 | Data (temuan Tahap 7) |

**Penamaan:** jumlah sampel ulang *bootstrap* ditulis `R`, bukan `B`, karena `B` sudah dipakai untuk *batch size*.

---

## 5. Spesifikasi per tahap

Setiap tahap memuat: package, masukan, proses, yang ditampilkan, yang disimpan, dan angka target. **Angka target Tahap 1–8 sudah diverifikasi** dengan menjalankan ulang pipeline di luar Colab. Kalau output berbeda, itu tanda ada *bug*.

### Tahap 0 — Persiapan lingkungan

- **Package:** `os`, `sys`, `json`, `platform`, `random`, `warnings` (bawaan Python); `numpy`, `pandas`, `torch`, `transformers`, `scikit-learn`, `scipy`, `matplotlib`, `openpyxl`, `pyarrow`; `google.colab.drive`.
- **Proses:**
  - *mount* Google Drive dan buat struktur folder (Bagian 3.1);
  - cek berkas data ada;
  - cek GPU (nama dan memori), dan beri ⚠️ kalau bukan T4;
  - atur variabel lingkungan untuk determinisme dan matikan kebisingan (Bagian 3.3–3.4);
  - buat `PATH`, `CFG`, `TARGET`, dan fungsi bantu `fmt_id()` (format angka Indonesia), `judul()`, `cek(nama, aktual, target, toleransi)`, `simpan_tabel()`;
  - instal hanya package yang belum ada (`pip install -q`).
- **Tampilkan:** tabel package dan versinya, GPU, dan path folder kerja.
- **Simpan:** `log/versi_package.json`.

### Tahap 1 — Pengumpulan data (memuat data mentah)

- **Package:** `pandas`, `numpy`, `openpyxl` (dipakai `pandas` untuk membaca .xlsx).
- **Proses:**
  - baca .xlsx dan buang kolom `Unnamed*`;
  - tambahkan `review_id` = 1 sampai 7.204 sesuai urutan baris. Ini nomor urut baris, bukan ID dari Google;
  - isi data mentah tidak diubah.
- **Tampilkan:** jumlah baris dan kolom, rentang tanggal, nilai unik `versi`, jumlah ulasan per bulan.
- **Target:**
  - 7.204 baris;
  - kolom `nama, rating, ulasan, tanggal, likes, balasan_dev, versi`;
  - tanggal 19 Juli 2026 sampai 11 September 2026;
  - versi hanya `1.6.0`;
  - per bulan: Juli 1.747 · Agustus 3.899 · September 1.558.
- **Simpan:** `hasil/tahap01_mentah.parquet`.

### Tahap 2 — Pemeriksaan kualitas data (tanpa mengubah data)

- **Package:** `pandas`, `numpy`, `re` (bawaan Python), `scipy.stats.spearmanr`.
- **Proses dan target:**

| Pemeriksaan | Definisi | Target |
|---|---|---|
| Nilai kosong | `isna()` per kolom | hanya `balasan_dev`: 4.834 |
| Balasan pengembang | persentase ulasan yang dibalas, per kelompok *rating* | *rating* 1–4: 95,0% · *rating* 5: 1,0% |
| *Likes* | persentase bernilai nol; korelasi Spearman *likes*–*rating* | 95,1%; ρ = −0,2483 |
| *Rating* 3 yang panjang | *rating* 3 dengan ≥ 8 kata | 237 dari 366 |
| Median kata per *rating* | `ulasan` dipisah spasi | 15, 15, 11, 3, 2 |
| Emoji atau simbol | baris yang cocok pola `EMO` atau `SIMBOL` (Bagian 6) | 404 |
| Huruf berulang ≥ 3 | pola `HURUF` pada teks huruf kecil | 227 (3,2%) |
| Tanda baca berulang | pola `BACA` | 634 |
| Duplikat persis | `ulasan.duplicated()` (salinan kedua dan seterusnya) | 34,8% |
| Teks berlabel ganda | teks (huruf kecil, tanpa spasi tepi) yang muncul dengan dua label | 33 teks, 1.968 baris |
| Nama unik | `nama.nunique()` | 7.153 |

- **Catatan:** pola emoji, huruf berulang, dan tanda baca di sini **sama dengan Tahap 5**, supaya angkanya konsisten. Angka lama 398 / 600 / 563 memakai pola yang berbeda dan tidak dipakai lagi.
- **Tampilkan:** satu tabel "Temuan → keputusan yang dibenarkan" dengan kolom temuan, angka, dan tahap yang memakainya.
- **Cek:** jumlah baris tetap 7.204.
- **Simpan:** `bab4/tabel/tahap02_kualitas.csv`.

### Tahap 3 — Seleksi atribut

- **Package:** `pandas`.
- **Proses:**
  - buang `nama`, `balasan_dev`, dan `likes`;
  - pertahankan `review_id, ulasan, rating, tanggal, versi`;
  - `ulasan` adalah satu-satunya masukan model, sedangkan `rating` adalah sumber label, bukan fitur.
- **Tampilkan:** tabel atribut | peran | alasan (7 baris).
- **Target:** 7.204 baris, 5 kolom.
- **Simpan:** `hasil/tahap03_atribut.parquet`.

### Tahap 4 — Pelabelan (*top-2-box*)

- **Package:** `pandas`, `matplotlib`.
- **Proses:** `label = (rating >= 4).astype(int)`. Tidak ada baris yang dihapus.
- **Tampilkan:** tabel *rating* → jumlah → kelas, ditambah total per kelas.
- **Target:**
  - *rating* 1: 1.305 · 2: 283 · 3: 366 · 4: 492 · 5: 4.758;
  - Positif 5.250 (72,88%), Non-Positif 1.954 (27,12%).
- **Gambar:** `bab4/gambar/tahap04_distribusi_rating.png`, grafik batang per *rating* yang diwarnai per kelas.
- **Simpan:** `hasil/tahap04_label.parquet`.

### Tahap 5 — *Pre-processing*

- **Package:** `re`, `unicodedata` (bawaan Python), `pandas`.
- **Tulis `lib/wondr_teks.py`** dari kode acuan di Bagian 6. **Salin persis.**
- **Urutan langkah:**
  1. normalisasi Unicode NFKC;
  2. emoji dan simbol diganti spasi;
  3. huruf kecil (*case folding*);
  4. huruf yang berulang ≥ 3 kali dipendekkan menjadi 2 (hanya huruf, bukan angka);
  5. tanda baca berulang dipendekkan menjadi 1;
  6. spasi dirapikan.

  Setelah itu, teks kosong dibuang.
- Tanpa *stemming*, tanpa penghapusan *stopword*, dan tanpa normalisasi kamus.
- **Tampilkan:**
  - tabel jumlah ulasan yang berubah di setiap langkah;
  - jumlah teks kosong yang dibuang beserta *rating*-nya;
  - 3 contoh sebelum → sesudah untuk langkah 4.
- **Target:**
  - perubahan per langkah, dihitung berurutan: 55 · 404 · 1.518 · 227 · 634 · 403;
  - teks kosong 63, semuanya Positif (*rating* 5: 57, *rating* 4: 6);
  - hasil **7.141**.
- **Simpan:** `hasil/tahap05_bersih.parquet`, dengan tambahan kolom `text_clean`.

### Tahap 6 — Deduplikasi

- **Package:** `re`, `pandas`, `numpy`, `lib.wondr_teks`.
- **Proses** (Bagian 6):
  - buat kunci deduplikasi, dan buang kunci kosong (target 0);
  - per kunci, hitung suara label; kunci yang suaranya seri dibuang;
  - label = label mayoritas;
  - wakil = anggota berlabel mayoritas dengan `text_clean` yang paling sering, lalu `review_id` terkecil.
- **Tampilkan:**
  - alur jumlah: 7.141 → jumlah kunci → jumlah dokumen;
  - rincian yang dibuang: salinan berlabel sama, anggota kalah suara, anggota kunci seri;
  - proporsi kelas sebelum dan sesudah.
- **Target:**
  - 4.372 kunci, termasuk 8 kunci seri (16 ulasan);
  - **4.364 dokumen**;
  - dibuang 2.777 = 2.688 + 73 + 16;
  - Positif 72,6% → 57,3%.
- **Simpan:** `hasil/tahap06_dedup.parquet` dan `bab4/tabel/tahap06_alur.csv`.

### Tahap 7 — Pembagian data (berbasis grup, stratifikasi label × panjang)

- **Package:** `numpy`, `pandas`, `lib.wondr_teks`.
- **Proses** (Bagian 6):
  - buat kunci grup. Kata fungsi dibuang **hanya untuk kunci**; teksnya tidak diubah;
  - label grup = label mayoritas. Kalau seri, pakai label anggota dengan `review_id` terkecil;
  - kelompok panjang grup ditentukan dari median jumlah kata anggotanya, sehingga terbentuk 6 strata;
  - urutan grup di setiap strata diacak dengan `default_rng(42)`. Satu generator dipakai untuk semua strata, dan strata diproses berurutan;
  - grup dimasukkan utuh ke bagian dengan tingkat keterisian (terisi ÷ target) terkecil.
- **Tampilkan:** jumlah grup, tabel bagian × label, tabel 6 strata × bagian, dan hasil cek kebocoran.
- **Target:**
  - 4.293 grup;
  - latih 3.051 (Non-Positif 1.303 / Positif 1.748);
  - validasi 657 (282 / 375);
  - uji 656 (280 / 376);
  - tidak ada `group_key`, `text_clean`, atau `dedup_key` yang muncul di lebih dari satu bagian;
  - kelima `review_id` kembaran-dekat (Bagian 4) berada di data uji.
- **Simpan:**
  - `hasil/tahap07_latih.parquet`, `hasil/tahap07_validasi.parquet`, `hasil/tahap07_uji.parquet`, dengan kolom `review_id, text_clean, label, kelompok_panjang`;
  - `bab4/tabel/tahap07_strata.csv`.
- **Mulai di sini data uji dikunci.** Tahap 9–14 tidak boleh menghitung metrik apa pun di data uji. Pengecualiannya hanya acuan minimum di Tahap 8, yang tidak memengaruhi keputusan apa pun.

### Tahap 8 — Eksplorasi data dan acuan minimum

- **Package:** `pandas`, `numpy`, `scipy.stats.chi2_contingency`, `scikit-learn` (`DummyClassifier`, `f1_score`, `accuracy_score`).
- **Proses:**
  - hitung proporsi label dan kelompok panjang per bagian;
  - uji khi-kuadrat untuk bagian × label dan bagian × kelompok panjang;
  - acuan minimum: `DummyClassifier(strategy="most_frequent")` yang dilatih pada label data latih.
- **Tampilkan:** dua tabel proporsi, dua nilai p, dan tabel acuan (akurasi dan *Macro F1* di validasi dan uji).
- **Target:**
  - Non-Positif di latih / validasi / uji: 42,7% / 42,9% / 42,7%;
  - kelompok panjang 1–2 / 3–10 / >10 kata: latih 14,0 / 47,1 / 38,9%; validasi 14,3 / 46,9 / 38,8%; uji 14,0 / 47,1 / 38,9%;
  - p = 0,9945 (label) dan 0,9998 (panjang);
  - acuan di validasi: akurasi 0,5708, *Macro F1* **0,3634**;
  - acuan di uji: akurasi 0,5732, *Macro F1* **0,3643**.
- **Simpan:** `hasil/tahap08_acuan.json` dan `bab4/tabel/tahap08_proporsi.csv`.

### Tahap 9 — Tokenisasi dan panjang masukan

- **Package:** `transformers.AutoTokenizer`, `numpy`, `pandas`, `math` (bawaan Python), `matplotlib`.
- **Proses.** L_max ditetapkan dari **data latih saja**:
  - *fertility* F = jumlah token subkata (tanpa `[CLS]`/`[SEP]`) ÷ jumlah kata (dipisah spasi);
  - r_UNK = jumlah token `[UNK]` ÷ jumlah token subkata;
  - panjang token per dokumen, termasuk `[CLS]` dan `[SEP]`;
  - L_max = 32 × ⌈panjang maksimum di data latih ÷ 32⌉;
  - sebagai deskripsi saja: panjang maksimum di validasi dan uji, serta jumlah dokumen yang terpotong pada L_max.
- **Tampilkan:**
  - tabel per bagian: jumlah dokumen, median, P95, P99, maksimum, dan jumlah terpotong;
  - nilai F, r_UNK, dan L_max.
- **Target:**
  - L_max = **128** (perkiraan dari reproduksi);
  - dokumen terpotong: latih 0, validasi 0, uji 1 (`review_id` 2741, sekitar 141 token);
  - kalau L_max ≠ 128, berhenti dan laporkan, karena naskah memakai 128.
- **Gambar:** `bab4/gambar/tahap09_panjang_token.png`, histogram panjang token data latih dengan garis L_max.
- **Simpan:** `hasil/tahap09_tokenisasi.json`, berisi F, r_UNK, L_max, dan statistik per bagian.

### Tahap 10 — *Fine-tuning* skema *Default*

- **Package:** `torch`; `transformers` (`AutoModelForSequenceClassification`, `AutoTokenizer`, `get_linear_schedule_with_warmup`); `scikit-learn.metrics.f1_score`; `numpy`; `pandas`; `time` (bawaan Python).
- **Tulis `lib/wondr_latih.py`** dengan dua fungsi:
  - `latih(lr, wr, wd, seed, kembalikan_prob=False)` → dict berisi `f1_terbaik`, `epoch_terbaik`, `f1_per_epoch`, `detik`. Jika `kembalikan_prob=True`, dict juga berisi peluang kelas validasi dan uji dari bobot *epoch* terbaik.
  - `fitness(lr, wr, wd)` → rata-rata `f1_terbaik` dari *seed* 42 dan 43, beserta kedua skornya.
- **Rincian `latih()`:**
  1. Panggil `atur_seed(seed)` **sebelum** model dibuat, karena bobot awal lapisan klasifikasi (2 × 768) bergantung pada *seed*.
  2. Tokenisasi dengan `truncation=True, max_length=L_max`, *padding* dinamis per *batch*. Tokenisasi dilakukan sekali per sesi lalu disimpan di memori (*cache*).
  3. DataLoader latih memakai `shuffle=True` dengan `torch.Generator().manual_seed(seed)`. DataLoader validasi tidak diacak.
  4. AdamW. *Weight decay* tidak dikenakan pada bias dan `LayerNorm.weight`, mengikuti implementasi BERT asli. **Ini Keputusan teknis: konfirmasikan dulu ke saya dan pastikan sesuai dengan Bab III.**
  5. T = ⌈3.051 / 16⌉ × 4 = 764 dan T_w = ⌊ρ · 764⌋. Jadwal tetap memakai T = 764 walaupun pelatihan berhenti lebih awal.
  6. *Loss* *cross-entropy* tanpa bobot kelas (bawaan model), lalu `clip_grad_norm_(…, 1.0)`.
  7. Setelah setiap *epoch*, hitung *Macro F1* validasi dengan `average="macro", zero_division=0`. Skor terbaik diperbarui hanya kalau nilainya **lebih besar**, bukan sama dengan. Pelatihan berhenti bila 2 *epoch* berturut-turut tidak naik.
  8. Bebaskan memori GPU setelah selesai.
- **Proses tahap:** jalankan `fitness(2e-5, 0.1, 0.01)`.
- **Tampilkan:**
  - nilai T dan T_w;
  - satu baris per *seed*: *Macro F1* per *epoch*, *epoch* terbaik, dan durasi;
  - *fitness* *Default*;
  - perkiraan waktu Tahap 11 dan Tahap 12 masing-masing (durasi rata-rata × 100 pelatihan).
- **Target:** T = 764, T_w = 76, dan *Macro F1* jauh di atas 0,3634.
- **Simpan:** `hasil/tahap10_default.json`.

### Tahap 11 — *Random search*

- **Package:** `numpy`, `json` (bawaan Python), `pandas`, `lib.wondr_latih`.
- **Proses:**
  - `rng = np.random.default_rng(2024)` lalu `F = rng.random((50, 3))`. Baris adalah kandidat; kolom adalah lr, wr, dan wd;
  - dekode F menjadi nilai *hyperparameter* (Bagian 4);
  - evaluasi `fitness()` berurutan dari k = 0 sampai 49;
  - simpan *state* setelah setiap kandidat, dan lanjutkan dari *state* jika berkasnya sudah ada.
- **Tampilkan:**
  - selama berjalan, satu baris pendek per kandidat: k, lr, wr, wd, skor42, skor43, *fitness*, terbaik sejauh ini;
  - di akhir: tabel 5 kandidat teratas, kandidat terpilih (argmax; kalau seri pilih indeks terkecil), dan selisihnya dengan *Default*.
- **Cek:**
  - 50 kandidat dan 100 pelatihan selesai;
  - semua nilai berada di dalam rentang: lr [1e-5; 5e-5], wr [0; 0,2], wd [0; 0,3].
- **Simpan:** `log/rs_state.json`, `hasil/tahap11_rs.csv`, `bab4/tabel/tahap11_rs_top5.csv`.

### Tahap 12 — PSO

- **Package:** `numpy`, `json` (bawaan Python), `pandas`, `matplotlib`, `lib.wondr_latih`.
- **Proses.** Pembaruan **serentak** (semua partikel pindah dulu, baru dievaluasi), tanpa ambang δ/ε atau kriteria berhenti lain:

```
rng = np.random.default_rng(7)
X = rng.uniform(0, 1, (10, 3));  V = rng.uniform(-1, 1, (10, 3))
evaluasi fitness 10 posisi  →  P = X.copy(), fP = fitness, g = P[argmax fP]
untuk t = 1..4:
    rp = rng.random((10, 3));  rg = rng.random((10, 3))
    V = 0.729*V + 1.49445*rp*(P - X) + 1.49445*rg*(g - X)
    V = clip(V, -1, 1)                                  # V_max
    X = X + V
    luar = (X < 0) | (X > 1)
    X = clip(X, 0, 1);  V[luar] = 0                     # dinding penyerap
    evaluasi fitness 10 posisi baru
    P_i ← X_i jika fitness baru > fP_i (lebih besar ketat);  g = P[argmax fP]
```

  - Total angka acak dari *seed* 7 adalah 300: 60 + 4 × 60.
  - *State* (X, V, P, fP, g, iterasi, partikel terakhir yang dievaluasi, dan `rng.bit_generator.state`) disimpan setelah setiap evaluasi. Dengan begitu, angka acak tidak diundi ulang saat program dilanjutkan.
- **Tampilkan:**
  - satu baris per evaluasi: iterasi, partikel, lr, wr, wd, *fitness*, apakah p_i diperbarui, apakah kena dinding;
  - satu baris ringkasan per iterasi: *global best* dan jumlah partikel yang memperbaiki p_i;
  - di akhir: *global best* (nilai asli lr, wr, wd), serta selisihnya dengan *Default* dan *random search*.
- **Cek:**
  - 50 evaluasi selesai;
  - *global best* tidak pernah turun;
  - semua posisi di [0, 1] dan semua |V| ≤ 1.
- **Gambar:** `bab4/gambar/tahap12_konvergensi.png`, grafik "terbaik sejauh ini" terhadap jumlah evaluasi untuk RS dan PSO dalam satu grafik, ditambah garis mendatar *Default*.
- **Simpan:** `log/pso_state.json`, `hasil/tahap12_pso.csv` (seluruh 50 evaluasi), `bab4/tabel/tahap12_iterasi.csv`.

### Tahap 13 — Estimasi ragam antar-*seed*

- **Package:** `numpy`, `pandas`.
- **Proses.** Dari 100 konfigurasi (50 RS + 50 PSO):
  - d_k = skor42 − skor43;
  - s_k² = d_k² / 2;
  - s_gab = √(rata-rata s_k²);
  - SE_selisih = s_gab · √(2/J), dengan J = 2, sehingga SE_selisih = s_gab;
  - zona kebetulan = ±2 · SE.
- **Tampilkan:**
  - nilai K, s_gab, SE, dan zona;
  - tabel 3 selisih *fitness* (RS − *Default*, PSO − *Default*, PSO − RS) dengan keterangan "di dalam zona" atau "di luar zona";
  - catatan satu kalimat bahwa ini pemeriksaan awal di data validasi, bukan uji akhir.
- **Simpan:** `hasil/tahap13_ragam.json` dan `bab4/tabel/tahap13_selisih.csv`.

### Tahap 14 — Pelatihan model akhir

- **Package:** `torch`, `transformers`, `numpy`, `pandas`, `lib.wondr_latih`.
- **Proses:**
  - latih 3 skema (*Default*, terbaik RS, *global best* PSO) × *seed* 42, 43, 44 = 9 pelatihan, dengan prosedur Tahap 10;
  - dari bobot *epoch* terbaik, simpan **peluang kelas** (*softmax*) untuk data validasi dan data uji ke `hasil/tahap14_prediksi.parquet`;
  - **tidak ada metrik data uji yang dihitung atau dicetak di tahap ini;**
  - bobot model tidak disimpan ke Drive (menghemat sekitar 4,5 GB), kecuali saya minta.
- **Cek determinisme:** skor validasi terbaik *seed* 42 dan 43 di tahap ini harus sama dengan yang tercatat di Tahap 10, 11, dan 12 (selisih ≤ 1e-4). Kalau tidak sama, berhenti dan laporkan.
- **Tampilkan:**
  - tabel 9 baris: skema, *seed*, lr, wr, wd, *epoch* terbaik, *Macro F1* validasi;
  - hasil cek determinisme.
- **Simpan:** `hasil/tahap14_prediksi.parquet` dan `hasil/tahap14_model.json`.

### Tahap 15 — Evaluasi pada data uji

- **Package:** `numpy`, `pandas`, `scikit-learn` (`confusion_matrix`, `precision_recall_fscore_support`, `accuracy_score`, `f1_score`), `matplotlib`.
- **Proses:**
  - tebakan = argmax peluang;
  - per model: *confusion matrix*, *precision*/*recall*/F1 per kelas, akurasi, dan *Macro F1*;
  - per skema: rata-rata ± simpangan baku (ddof = 1) dari 3 *seed*;
  - ansambel suara terbanyak (minimal 2 dari 3 model) per skema, lalu *confusion matrix* dan metriknya;
  - bandingkan dengan acuan minimum 0,3643;
  - **analisis sensitivitas:** *Macro F1* ansambel tanpa 5 `review_id` kembaran-dekat.
- **Tampilkan:**
  - (a) tabel 9 model, cukup kolom *Macro F1*;
  - (b) tabel rata-rata ± simpangan baku per skema;
  - (c) tabel ansambel: akurasi, P/R/F1 per kelas, *Macro F1*;
  - (d) tabel sensitivitas: dengan dan tanpa 5 dokumen, beserta selisihnya.
- **Cek:**
  - tidak ada seri di ansambel;
  - n = 656;
  - semua skema di atas 0,3643.
- **Gambar:** `bab4/gambar/tahap15_cm_ansambel.png`, 3 panel *confusion matrix* ansambel.
- **Simpan:** `hasil/tahap15_ansambel.parquet` (`review_id`, label, tebakan ansambel ketiga skema) dan `bab4/tabel/tahap15_*.csv`.

### Tahap 16 — Uji signifikansi statistik

- **Package:** `numpy`, `pandas`, `scipy.stats.norm`, `scikit-learn.metrics.f1_score` (untuk verifikasi), `matplotlib`.
- **Proses:**
  1. Tiga perbandingan: *Default* vs RS, *Default* vs PSO, RS vs PSO. Untuk setiap pasangan (A, B), δ̂ = *Macro F1*(B) − *Macro F1*(A), dihitung pada tebakan ansambel.
  2. `rng = np.random.default_rng(42)` lalu `idx = rng.integers(0, 656, size=(10000, 656))`. **Satu matriks indeks dipakai untuk ketiga skema dan ketiga perbandingan**, sehingga ujinya berpasangan.
  3. *Macro F1* setiap sampel ulang dihitung secara tervektorisasi dari TP/FP/FN. Verifikasi: pada data asli, hasilnya sama dengan `sklearn.metrics.f1_score` sampai 1e-12.
  4. α' = 0,05/3. Selang kepercayaan = persentil [100·α'/2 ; 100·(1 − α'/2)] = [0,8333 ; 99,1667] dari 10.000 nilai δ*.
  5. Keputusan: **signifikan** jika selang tidak memuat 0.
  6. Analisis daya:
     - SE = simpangan baku δ* (ddof = 1);
     - z₁ = `norm.ppf(1 − α'/2)` = 2,3940 dan z₂ = `norm.ppf(0.80)` = 0,8416;
     - Δ_min = 3,2356 · SE;
     - n_perlu = ⌈656 · (Δ_min / 0,005)²⌉.
- **Tampilkan:**
  - satu tabel 3 baris: perbandingan, δ̂, batas bawah, batas atas, signifikan?, SE, Δ_min, n_perlu;
  - satu kalimat kesimpulan untuk setiap baris.
- **Gambar:** `bab4/gambar/tahap16_bootstrap.png`, 3 histogram δ* dengan garis nol dan batas selang.
- **Simpan:** `bab4/tabel/tahap16_signifikansi.csv` dan `hasil/tahap16_bootstrap.npz` (10.000 δ* per perbandingan).

---

## 6. Kode acuan Tahap 5–7 (wajib menghasilkan angka yang sama)

Kode di bawah sudah diverifikasi pada `data_wonder_bybni_3.xlsx`. Hasilnya: perubahan per langkah 55 · 404 · 1.518 · 227 · 634 · 403; 7.141 ulasan → 4.372 kunci → 4.364 dokumen → 4.293 grup → latih 3.051 / validasi 657 / uji 656.

Simpan kode ini ke `lib/wondr_teks.py` lalu panggil dari sel Tahap 5–7. **Logika, pola, urutan, dan urutan kunci `PROPORSI` jangan diubah**, karena urutan kunci menentukan pemecah seri saat alokasi.

```python
import re, unicodedata
import numpy as np, pandas as pd

# ── Pola teks (Tahap 2 dan Tahap 5 memakai pola yang sama) ──
EMO = re.compile("[\U0001F000-\U0001FAFF\U0001F1E6-\U0001F1FF☀-➿⬀-⯿"
                 "⌀-⏿←-⇿⤀-⥿〰〽㊗㊙"
                 "©®‼⁉™ℹⓂ▪-◾️‍"
                 "\U000E0020-\U000E007F]")
SIMBOL = re.compile("[☀-➿⬀-⯿️‍\U0001F3FB-\U0001F3FF]")
HURUF = re.compile(r"([^\W\d_])\1{2,}")      # huruf yang berulang >= 3 kali (hanya huruf)
BACA = re.compile(r"[!?.,;:]{2,}")           # tanda baca berurutan >= 2

LANGKAH = [  # (nama, fungsi) -- urutan Tahap 5, jangan diubah
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

# ── Tahap 6: deduplikasi ──
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

# ── Tahap 7: pembagian data berbasis grup ──
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

## 7. Yang TIDAK dikerjakan (sudah diputuskan, jangan ditambahkan)

- Model pembanding TF-IDF + SVM/Regresi Logistik, dan uji McNemar.
- Normalisasi kamus (Salsabila dkk.), aturan Brody, *stemming*, penghapusan *stopword* pada teks.
- Validasi label dengan Cohen's Kappa.
- Bobot kelas pada *loss*.
- Penyetelan ambang keputusan, ansambel rata-rata peluang, penambahan *seed* model akhir di atas 3, dan pelatihan ulang pada gabungan latih + validasi.
- Perbaikan kunci grup (membuang tanda baca). Pembagian data dipertahankan; dampak kembaran-dekat cukup ditunjukkan lewat analisis sensitivitas di Tahap 15.

Kalau kamu menilai salah satu hal di atas penting, sampaikan sebagai saran di akhir balasan. Jangan dimasukkan ke program.

---

## 8. Mulai

Kerjakan **Tahap 0 saja** dengan format balasan di Bagian 2, lalu tunggu output saya.

---

## 9. Keputusan tambahan (disetujui pengguna, 9 Oktober 2026)

Keputusan ini diambil setelah prompt dicocokkan dengan proposal (Bab I–III). **Kalau bertentangan dengan Bagian 1–8, bagian ini yang berlaku.**

| No. | Hal | Keputusan | Tahap |
|---|---|---|---|
| 1 | Cakupan *weight decay* | Tidak dikenakan pada bias dan `LayerNorm.weight`, mengikuti BERT asli (Devlin dkk., 2019). Naskah 2.8.2 perlu tambahan satu kalimat karena (2.15) menulis θ secara umum | 10 |
| 2 | Arah selisih δ | δ = skema baru − skema lama: RS − *Default*, PSO − *Default*, PSO − RS (sama dengan Tahap 13). Pada (2.40), A dibaca sebagai skema yang lebih baru. Keputusan signifikan tidak berubah | 16 |
| 3 | Waktu komputasi | `tahap11_rs.csv` dan `tahap12_pso.csv` menyimpan durasi per pelatihan (`detik42`, `detik43`). Tahap 15 menambah tabel (e): metode, jumlah evaluasi, jumlah pelatihan, total waktu (Bab III no. 15) | 11, 12, 15 |
| 4 | *Global best* PSO saat seri | Mengikuti (2.26) dan Algoritma 2.1: diperbarui per partikel sesuai urutan evaluasi, hanya jika **lebih besar ketat**. Kalau seri, posisi lama dipertahankan. Ini menggantikan `g = P[argmax fP]` | 12 |
| 5 | Cakupan Tahap 8 | Tetap seperti Bagian 5 (proporsi dan khi-kuadrat mencakup data uji; yang dibaca hanya label dan panjang). Naskah Bab III no. 8 disesuaikan | 8 |
| 6 | `review_id` | Dibuat di **Tahap 3** (Seleksi Atribut), bukan Tahap 1, sesuai Bab III no. 3. Tahap 1 hanya membuang kolom `Unnamed*`. Angka tidak berubah karena urutan baris tetap | 1, 3 |
| 7 | Nama tahap | Judul tahap persis sama dengan Bab III (daftar di bawah). Tahap 2 menambah baris "rentang *rating*" (target 1–5) | semua |
| 8 | Notasi | Ragam antar-*seed* memakai M = 100 seperti (2.45). Jumlah kandidat *random search* ditulis `n_kandidat_rs` = 50 | 11, 13 |
| 9 | Karakter tak terlihat | U+FE0F dan U+200D di `lib/wondr_teks.py` ditulis sebagai *escape* `️` dan `‍`. Pola regex tetap identik dengan Bagian 6, dan Tahap 5 mengecek kesamaannya secara otomatis | 5 |
| 10 | Keterulangan GPU | `torch.use_deterministic_algorithms(True)` diaktifkan di Tahap 0 dan di `atur_seed()`. Tahap 10 menjalankan ulang *seed* 42 satu kali dan memastikan skornya identik sebelum Tahap 11–12 | 0, 10 |

**Judul tahap (Bab III):**
1. Pengumpulan Data
2. Audit Struktur dan Kualitas Data
3. Seleksi Atribut
4. Pelabelan Data
5. *Pre-processing*
6. Deduplikasi Data
7. Pembagian Data
8. Eksplorasi Data dan Penetapan Acuan Minimum
9. Tokenisasi
10. *Fine-tuning* Skema *Default*
11. Optimasi *Hyperparameter* dengan *Random Search*
12. Optimasi *Hyperparameter* dengan PSO
13. Estimasi Ragam Antar-*seed*
14. Pelatihan Model Akhir
15. Evaluasi Model
16. Uji Signifikansi Statistik

**Catatan untuk naskah (program tetap):**
- Bab III no. 6: `kunci_dedup` meringkas semua karakter berulang, termasuk angka ("100" menjadi "10"). Kata "huruf" di naskah sebaiknya diganti "karakter".
- (2.3) dan Bab III no. 7 belum menulis tiga aturan kode: kunci memakai seluruh kata jika semua katanya kata fungsi; label grup seri memakai `review_id` terkecil; kelompok panjang grup dari median jumlah kata.

**Verifikasi lokal (9 Oktober 2026):** kode acuan Bagian 6 dijalankan ulang pada `data_wonder_bybni_3.xlsx` di lingkungan Claude Code (Python 3.13, pandas 3.0.5). Hasilnya **56 dari 56 target Tahap 1–8 cocok**.

## 10. Catatan repositori

- Notebook: `skripsi_wondr_v2.ipynb` di akar repo. Setiap tahap berisi 1 sel markdown dan 1–2 sel kode.
- Dataset dan berkas hasil (`*.xlsx`, `*.parquet`, `*.npz`) **tidak** di-*commit* karena dataset memuat kolom `nama` (lihat `.gitignore`).
- Cabang kerja: `claude/review-prompt-program-setup-d1vidv`. Lakukan *commit* dan *push* setelah setiap tahap.

| Tahap | Status |
|---|---|
| 0 | Revisi 1: `drive.mount` gagal (`ValueError: mount failed`); pengalihan stdout saat mount dihapus dan pesan petunjuk ditambahkan. Menunggu output Colab |
