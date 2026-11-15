# Panduan menjalankan program v4 sampai selesai

Disusun 15-11-2026. Panduan ini melanjutkan dari posisi berikut: **Colab sudah ✅ sampai Tahap 10.1**, dan **K0 Kaggle (KT-21) sudah ✅**.

Yang tersisa ada empat:

1. *random search* (RS) di Kaggle;
2. PSO di Colab, berjalan bersamaan dengan RS;
3. menggabungkan hasil dan membuat ringkasan;
4. model akhir, evaluasi, dan uji signifikansi.

---

## Daftar periksa singkat

Centang setiap langkah yang sudah selesai.

- [ ] **1. Kaggle:** unggah `paket_kaggle.zip` sebagai dataset **Private** (Bagian 2)
- [ ] **2. Kaggle:** jalankan K0 → K1 → K2. Bila K2 ✅, ikuti **Jalur A**. Bila K2 gagal, ikuti **Jalur B** (Bagian 3)
- [ ] **3A. Kaggle:** lakukan *commit* RS 2–3 kali sampai 5 *run* × 40 kandidat selesai, lalu unggah zip keluaran ke Drive (Bagian 4)
- [ ] **3B. Colab:** jalankan PSO di Tahap 12.1. Langkah ini berjalan **bersamaan** dengan 3A (Bagian 5)
- [ ] **4. Colab:** jalankan 11.2 → 11.3 → 11.4 → 12.1b → 12.2 (Bagian 7)
- [ ] **5. Colab:** cek sisa ruang Google Drive, lalu jalankan Tahap 13, dua sel (Bagian 8)
- [ ] **6. Colab:** jalankan 14.1 → 14.2 → 14.3 → 15 (Bagian 9)
- [ ] **7.** Simpan semua output dan berkas Bab IV (Bagian 10)

### Perkiraan waktu

Perkiraan ini (Hitungan) memakai 2,89 menit per pelatihan dari Tahap 10. Angka itu cenderung terlalu tinggi, karena *seed* 42 di Tahap 10 lebih lambat daripada di Tahap 10.0.

| Langkah | Tempat | Jumlah pelatihan | Perkiraan |
|---|---|---|---|
| Uji kesetaraan K0–K2 | Kaggle, interaktif | 2 | 20–30 menit |
| RS (5 *run* × 40 kandidat × 2 *seed*) | Kaggle, *commit* | 400 | ±19 jam GPU (2–3 *commit*) |
| PSO (5 *run* × 40 evaluasi × 2 *seed*) | Colab | 400 | ±19 jam (±3,9 jam per *run*) |
| Gabung dan ringkasan 11.2–12.2 | Colab | 2 (11.3) | ±15 menit |
| Model akhir (Tahap 13) | Colab | 15 | ±45–60 menit |
| Evaluasi dan uji signifikansi 14–15 | Colab | 0 | ±10 menit |

Pada Jalur B, RS dan PSO keduanya dijalankan di Colab secara bergantian, sehingga totalnya sekitar 38,5 jam (Hitungan: 800 × 2,89 menit).

---

## Bagian 1 — Aturan emas (baca dulu)

1. **Jangan pakai *Run all* dan jangan pakai *Run after*.**
   - Jalankan sel dengan mengklik ▶ di setiap sel, berurutan dari atas ke bawah. Kalau Anda mengklik ▶ beberapa sel sekaligus, Colab dan Kaggle akan **mengantrekannya** sehingga Anda tidak perlu menunggu satu per satu.
   - Kalau satu sel galat (tulisan merah), sel-sel yang masih antre otomatis batal.
2. **Jangan menjalankan ulang** sel yang sudah selesai:
   - Tahap 1–9.4 (bagian B dan C, final);
   - Tahap 10.0, 10.0b, 10, dan 10.1.

   Tahap 10 adalah acuan uji kesetaraan Kaggle, dan 10.1 adalah paket Kaggle yang sudah diunggah.
3. **Hanya ada dua kode yang boleh diubah**, dan hanya pada keadaan berikut:
   - `MAKS_RUN = None` menjadi `MAKS_RUN = 1` di sel 12.1, **hanya pada Jalur B** (Bagian 6);
   - `"simpan_bobot": "semua"` di 0.2 sel 2/2, **hanya bila ruang Drive tidak cukup**, dan diubah sebelum Tahap 13 (Bagian 8).

   Selain itu tidak ada kode yang diubah.
4. **Setelah setiap sel selesai, baca baris cek di bagian bawah outputnya.**
   - Semua ✅ → lanjut.
   - Ada ⚠️ atau tulisan merah → **berhenti**. Salin output sel itu ke berkas catatan, atau buat tangkapan layar. Jangan mencoba memperbaiki kodenya sendiri; output itu dianalisis bersama Claude di sesi berikutnya.
   - Tanda ⚠️ **tidak** menghentikan antrean; hanya galat yang menghentikannya. Karena itu baca output setiap sel, termasuk sel yang tadinya ikut antre.
5. **RS hanya dijalankan di satu tempat:**
   - Jalur A: di Kaggle;
   - Jalur B: di Colab, Tahap 11.1.

   Jangan di keduanya. PSO selalu di Colab.
6. **Kerahasiaan data:**
   - dataset Kaggle dan notebook Kaggle harus **Private**, karena keluaran notebook memuat teks ulasan data latih dan validasi;
   - data uji tidak pernah dikirim ke Kaggle;
   - berkas data dan hasil tidak diunggah ke GitHub, karena repo bersifat publik.
7. **Tidak ada berkas yang dihapus** (F14). Kalau ragu, biarkan berkasnya.

---

## Bagian 2 — Kaggle: unggah paket (sekali saja)

1. Buka Google Drive, masuk ke `MyDrive/skripsi_wondr_v4/kaggle/`, klik kanan `paket_kaggle.zip`, lalu pilih **Download**. Ukurannya ±0,8 MB (Data, Tahap 10.1).
2. Buka kaggle.com. Di menu kiri, pilih **Datasets** lalu **+ New Dataset**.
3. Seret `paket_kaggle.zip` ke kotak unggah. Beri judul, misalnya `paket-wondr-v4`.
4. Pastikan pilihan visibilitas **Private**, lalu klik **Create**. Tunggu sampai prosesnya selesai.
   - Kaggle mungkin mengekstrak zip secara otomatis. Itu tidak masalah, karena K1 bisa membaca zip maupun folder hasil ekstraksi.

---

## Bagian 3 — Kaggle: uji kesetaraan K0 → K1 → K2 (interaktif, ±20–30 menit)

1. Buka notebook Kaggle yang tadi menjalankan **K0 versi KT-21** (yang memberi 5 ✅ pada pukul 04:28).
   - Pastikan notebook itu **Private**: menu **File** atau tombol **Share**, lalu pilih *Private*.
   - Kalau notebook itu tidak ada:
     1. buka repo GitHub, cabang `claude/new-session-xc71do`;
     2. unduh `kaggle_pencarian.ipynb`;
     3. di Kaggle, pilih **Create → New Notebook → File → Import Notebook**.
2. Atur panel kanan **Session options**:
   - *Accelerator*: **GPU T4 x2** (program hanya memakai satu GPU);
   - *Internet*: **On**;
   - *Environment*: **Pin to original environment**;
   - *Dependency Manager*: dibiarkan **kosong**.
3. Di panel kanan **Input**, klik **+ Add Input**, cari dataset `paket-wondr-v4` milik Anda, lalu klik **Add**.
   - Pastikan hanya ada **satu** paket. Paket lama dihapus lewat ⋮ → *Remove*.
4. Klik ▶ **K0**, lalu ▶ **K1**, lalu ▶ **K2**. Boleh diklik langsung bertiga; sel akan antre.
5. Output yang harus terlihat:
   - **K0:** 5 baris ✅, sama seperti pukul 04:28 (satu Tesla T4; versi di disk dan di proses baru = `VERSI`).
   - **K1:**
     - `✅ MD5 setelah disalin · data uji tidak ada di paket · amp sudah diisi = [0, False, True]`;
     - `state dipulihkan: tidak ada`.
   - **K2:** dua baris, masing-masing untuk *seed* 42 dan 43, dengan bentuk `seed 42: F1 per epoch Kaggle [...] · Colab [...] · maks selisih peluang 0`. Setelah itu muncul `✅ Tahap 10 Kaggle identik persis dengan Colab ... = ya`.
     - Angka Colab yang ditampilkan (Data, Tahap 10, 6 desimal):
       - *seed* 42: 0,9222… · 0,9152… · 0,9235… · 0,9246…;
       - *seed* 43: 0,8975… · 0,9271… · 0,9128… · 0,9165….
     - Angka Kaggle harus **sama persis**, dan *maks selisih peluang* harus **0**.
6. **Salin output K0, K1, dan K2** ke catatan. Output ini adalah bukti kesetaraan platform untuk Bab III.
7. Tentukan jalur:
   - K2 ✅ → **Jalur A** (Bagian 4 dan Bagian 5, berjalan bersamaan);
   - K2 galat `UJI KESETARAAN GAGAL` → **Jalur B** (Bagian 6);
   - galat lain di K0, K1, atau K2 → berhenti dan simpan output.
8. **Hentikan sesi interaktif** dengan tombol ⏻ atau **Stop session**, supaya kuota GPU tidak terbuang. **Jangan** menjalankan K3 di sesi interaktif.

---

## Bagian 4 — Jalur A: RS di Kaggle (*commit*; berjalan sendiri di server Kaggle)

### 4.1 Commit pertama

1. Periksa dulu bahwa input masih berisi paket dan *Session options* masih sama seperti Bagian 3.
2. Di kanan atas, klik **Save Version**, pilih **Save & Run All (Commit)**, lalu klik **Save**.
   - Kalau ada *Advanced Settings*, pilih opsi agar output **selalu disimpan**.
3. Notebook berjalan di server Kaggle. **Laptop dan browser boleh ditutup.** Lamanya ±11,2 jam:
   - K0 memasang paket (±5 menit);
   - K1;
   - K2 dijalankan ulang (±5–8 menit, harus ✅ lagi);
   - K3 menjalankan RS dan berhenti rapi 11 jam sejak K0 mulai (Keputusan, `kaggle_batas_jam` = 11; batas Kaggle 12 jam);
   - K4 membuat zip.
4. Cara memantau: buka halaman notebook, lalu lihat daftar **Versions**. Statusnya *Running* lalu berubah menjadi *Complete*. Log K3 mencetak satu baris setiap 10 kandidat, misalnya `RS run  1 · evaluasi 10/40 · fitness terbaik … · … menit`.
5. Setelah statusnya *Complete*:
   1. buka versi itu, lalu tab **Output**;
   2. unduh `keluaran_kaggle_YYYYmmdd_HHMM.zip`;
   3. salin output K4.

   Pada *commit* pertama, baris cek K4 `tugas RS lengkap` masih ⚠️ karena belum kelima *run* selesai. **Ini normal.**

### 4.2 Commit kedua (dan ketiga bila perlu)

1. Buka notebook dan klik **Edit**.
2. Di panel **Input**, klik **+ Add Input**, pilih bagian notebook milik Anda (*Your Work → Notebooks* atau *Notebook Output Files*), pilih **notebook ini sendiri, versi terakhir**, lalu klik **Add**. Dataset paket **tetap** ada di input.
3. Klik **Save Version** → **Save & Run All (Commit)** → **Save**. Yang akan terlihat:
   - K1 menulis `state dipulihkan: rs 1 = 40 · rs 2 = 40 · rs 3 = …`;
   - K2 menulis `K2 dilewati: uji kesetaraan sudah lulus`;
   - K3 melanjutkan dari kandidat terakhir.
4. Ulangi sampai output K4 menampilkan kelima berkas berikut dengan status `40/40 evaluasi`, beserta cek ✅:

   ```
   rs_state_seed01.json   40/40 evaluasi
   ...
   rs_state_seed05.json   40/40 evaluasi
   ```

   Perkiraannya 2 *commit*. Hitungan: sekitar 10,7 jam efektif per *commit* ÷ 2,89 menit ≈ 220 pelatihan, dari total 400.
5. Untuk *commit* ketiga, tambahkan output versi **terbaru** ke input. Output versi lama boleh di-*Remove*, karena aturan gabung selalu memakai *state* yang terpanjang.
6. Unduh zip keluaran dari **versi terakhir**, karena zip itu memuat semua *state*. Unggah ke Google Drive `MyDrive/skripsi_wondr_v4/kaggle/` tanpa mengganti namanya.

**Kuota GPU Kaggle.** Kuota GPU dihitung per minggu; sisanya terlihat di panel kanan atau di profil. RS memerlukan sekitar 20 jam GPU, termasuk K0 dan K2 (Hitungan). Kalau kuota habis sebelum RS selesai, tunggu kuota direset, lalu lakukan *commit* lagi seperti langkah 4.2.

---

## Bagian 5 — Colab: PSO (bersamaan dengan Bagian 4)

### 5.1 Notebook yang dipakai (disarankan)

Pakai notebook versi repo terbaru, supaya semua kode sel sama dengan yang sudah diuji. Sel 0.1 sel 2/2 di notebook lama Anda masih versi lama.

1. Di Colab, pilih **File → Open notebook → tab GitHub**, lalu tempel alamat ini:
   `https://github.com/hmadaguss20-dot/skripsi-wondr-v2/blob/claude/new-session-xc71do/skripsi_wondr_v4_colab.ipynb`
2. Setelah notebook terbuka, pilih **File → Save a copy in Drive**.
3. Pilih **Runtime → Change runtime type → T4 GPU**.
4. Notebook lama Anda **simpan sebagai arsip** output Tahap 1–10.1. Jangan dihapus.

Untuk berpindah ke tahap tertentu, pakai panel kiri **Table of contents** (ikon ☰).

### 5.2 Awal setiap sesi Colab: bagian A

1. ▶ **0.1 sel 1/2**.
   - Kalau muncul pesan **RESTART RUNTIME** (karena versi paket baru dipasang), pilih **Runtime → Restart session**, lalu ulangi dari 0.1 sel 1/2.
2. ▶ **0.1 sel 2/2**. Semua harus ✅, termasuk GPU **Tesla T4** dan `versi yang dimuat = VERSI`.
3. ▶ **0.2 sel 1/2**, lalu ▶ **0.2 sel 2/2**. Hasilnya harus 7 cek ✅ dan baris `amp (F6): True`.
4. Bagian **D** (3 sel `%%writefile`) cukup dijalankan **sekali** pada notebook baru dari GitHub. Isinya sama dengan yang sudah ada di Drive, jadi aman. Setelah itu D tidak perlu dijalankan lagi.

### 5.3 Pertama kali: 11.0 lalu 12.1

1. ▶ **11.0** (cek *state*, tanpa GPU). Hasil yang diharapkan:
   - `state yang ada 0 dari 10`;
   - `✅ state dari konfigurasi lain · lingkungan tidak sah · berkas asing = [0, 0, 0]`.
2. ▶ **12.1 PSO**. Pada Jalur A, `MAKS_RUN = None` **jangan diubah**.
   - Satu baris dicetak per iterasi, misalnya `PSO seed 11 · t 0 · evaluasi 10/40 · fitness maks iterasi … · global best … · … menit`.
   - Setiap *run* punya 4 baris (t 0–3), dan ada 5 *run* (*seed* 11–15).
   - Perkiraan ±3,9 jam per *run* dan ±19 jam total (Hitungan: 80 pelatihan × 2,89 menit per *run*).
   - Setelah kelima *run* selesai, akan muncul `seed 11: 40/40 evaluasi · global best …` (5 baris) dan 3 cek ✅:
     - contoh Tabel 2.17: [5,6237; 3,6355] · [2,6718; 3,3698] · 9,8923 (Rujukan, Bab II);
     - run lengkap = 5;
     - `[ya, ya, ya, ya, ya]`.
3. **Biarkan tab Colab tetap terbuka**, dan atur laptop supaya tidak tidur (*sleep*). Pastikan saldo unit komputasi Colab cukup untuk ±20 jam T4 (Hitungan).

### 5.4 Kalau Colab terputus (ini normal untuk proses yang panjang)

1. Klik **Connect** atau **Reconnect**. Kalau ditanya *runtime*, pilih **T4 GPU**.
2. Jalankan bagian **A** (5.2), lalu ▶ **12.1** lagi. Boleh ▶ 11.0 lebih dulu untuk melihat progres.
3. PSO **melanjutkan sendiri** dari evaluasi terakhir yang tersimpan, karena *state* disimpan setelah setiap evaluasi. Paling banyak satu evaluasi (±6 menit, Hitungan) yang perlu diulang.
4. Output 12.1 yang terputus memang tidak berisi baris cek akhir. Baris cek baru muncul pada jalan yang menyelesaikan kelima *run*.

---

## Bagian 6 — Jalur B (hanya bila K2 gagal)

1. **Jangan melakukan *commit* di Kaggle.** Simpan output K2 yang gagal sebagai bukti untuk Bab III.
2. Di Colab, ubah satu baris di sel **12.1**: `MAKS_RUN = None` menjadi `MAKS_RUN = 1`. Sel **11.1** sudah `MAKS_RUN = 1`, jadi tidak perlu diubah.
3. Satu putaran terdiri dari:
   1. **A**;
   2. ▶ **11.1**: RS satu *run*, ±3,9 jam;
   3. ▶ **12.1**: PSO satu *run*, ±3,9 jam.

   Ulangi 5 putaran sampai:
   - 11.1 menampilkan `seed 1: 40/40 · … · seed 5: 40/40` dan `✅ run RS lengkap = 5`;
   - 12.1 menampilkan 5 *run* lengkap dan 3 cek ✅.
4. Kalau terputus di tengah: jalankan **A**, lalu sel yang tadi sedang berjalan (11.1 atau 12.1). Sel itu melanjutkan dari *state*.
5. Setelah selesai, **lewati 11.2** karena tidak ada zip Kaggle. Lanjutkan ke Bagian 7 mulai dari 11.3; 11.3 tetap dijalankan sebagai cek determinisme.

---

## Bagian 7 — Gabungkan dan ringkas (Colab, ±15 menit)

**Syarat:**

- Jalur A: K4 versi terakhir menampilkan 5 *run* RS dengan `40/40`, dan zip keluarannya sudah diunggah ke `kaggle/` di Drive.
- 12.1 sudah menampilkan 5 *run* lengkap dan 3 cek ✅.

**Urutan:** A → **11.2** → **11.3** → **11.4** → **12.1b** → **12.2**. Semuanya boleh diantrekan.

| Sel | Hasil yang diharapkan |
|---|---|
| **11.2** Impor Kaggle | `uji kesetaraan Kaggle ... lulus: ya`; RS *seed* 1–5 `40/40 · lingkungan sah ya · platform kaggle`; PSO *seed* 11–15 `40/40 · platform colab`; ✅ `[0, 0]`; ✅ `[5, 5]` |
| **11.3** Cek silang (GPU, ±6 menit) | `✅ skor seed 42 · 43 identik persis = [ya, ya]`. Kalau berbeda, muncul galat **TITIK BERHENTI 2**. Antrean berhenti sendiri; **berhenti** dan simpan output |
| **11.4** Ringkasan RS | ✅ `[5, 200, 400]`; dua cek lain ✅ (`ya`) |
| **12.1b** Ringkasan PSO | ✅ `[5, 200, 400]`; cek *global best* dan dekode ✅; lingkungan identik ✅ |
| **12.2** Ringkasan metode | 2 cek ✅; gambar konvergensi tampil dan tersimpan |

Kalau semua ✅, lanjut ke Bagian 8. Ringkasan di 12.2 bersifat **deskriptif** dan tidak memuat uji statistik (F12). Keputusan statistik hanya diambil di Tahap 15.

---

## Bagian 8 — Model akhir (Tahap 13; GPU ±45–60 menit)

### 8.1 Cek ruang Google Drive dulu

- `CFG["simpan_bobot"] = "semua"` menyimpan bobot 15 model, kira-kira **7,5 GB** (Keputusan F13; ukuran pastinya dicetak di baris pertama Tahap 13).
- Buka drive.google.com dan lihat **Penyimpanan** di kiri bawah. Diperlukan ruang kosong **≥ 8 GB**.
- Kalau ruang tidak cukup:
  1. di **0.2 sel 2/2**, ganti `"simpan_bobot": "semua"` menjadi `"simpan_bobot": "terbaik"`, yaitu 5 model skema terbaik (±2,5 GB, dilatih ulang 5 kali, ±15 menit tambahan). Pilihan `"tidak"` berarti tidak ada bobot yang disimpan;
  2. jalankan ulang **0.2 sel 1/2** dan **0.2 sel 2/2**.
- Tentukan pilihan ini **sebelum** Tahap 13 dimulai, dan **jangan** mengubahnya di tengah jalan.

### 8.2 Jalankan

**Urutan:** A → **13 sel 1/2** → **13 sel 2/2**.

- **13 sel 1/2:**
  - baris pertama berbentuk `simpan_bobot = 'semua': 15 model × … MB ≈ … GB`;
  - lalu 15 baris (*default*, rs, pso × *seed* 44–48), misalnya `default  seed 44 · lr … · epoch terbaik …`;
  - di akhir `✅ ... = [15, 15]`. Untuk pilihan `"terbaik"`, angka bobotnya 5; untuk `"tidak"`, 0.
  - Kalau terputus: jalankan **A**, lalu **13 sel 1/2** lagi. Sel ini melanjutkan dari pelatihan terakhir yang tersimpan.
- **13 sel 2/2:** tabel 15 model, `✅ [15, ya, ya]`, dan `✅ [852, 852, ya, ya]` (Data: validasi 852, uji 852).

---

## Bagian 9 — Evaluasi dan uji signifikansi (tanpa GPU, ±10 menit)

**Urutan** (boleh diantrekan): A → **14.1 sel 1/2** → **14.1 sel 2/2** → **14.2 sel 1/2** → **14.2 sel 2/2** → **14.3** → **15 sel 1/2** → **15 sel 2/2**.

Data uji baru dipakai mulai Tahap 14 (F14).

| Sel | Hasil yang diharapkan |
|---|---|
| 14.1 sel 1/2 | 15 model, n uji 852 per model; rata-rata *Macro F1* setiap skema > acuan minimum uji ✅ |
| 14.1 sel 2/2 | total pelatihan Tahap 10–13 = **817** ✅ (F5) |
| 14.2 sel 1/2 | 852 dokumen, 5 suara per dokumen dan skema, seri 0 ✅; ansambel = kelas 1 bila ≥ 3 dari 5 ✅ |
| 14.2 sel 2/2 | 3 panel *confusion matrix*, jumlah sel per panel 852, gambar tersimpan ✅ |
| 14.3 | n uji 852 dan aturan label ✅; κ rumus (2.5) = κ sklearn ✅ |
| 15 sel 1/2 | contoh Tabel 2.19 = **0,6703**; persentil [**0,8333**; **99,1667**] ✅ (Rujukan); *Macro F1* asli = sklearn ✅; 15 model, 5 *seed* per skema ✅; tabel 3 perbandingan dengan kolom **signifikan?** |
| 15 sel 2/2 | 3 histogram δ*, R = 10.000 per panel, gambar tersimpan ✅ |

**Program selesai** setelah 15 sel 2/2 ✅.

---

## Bagian 10 — Yang perlu disimpan

### 10.1 Output yang disalin, untuk audit dan Bab IV

Simpan teks output sel-sel berikut ke satu berkas catatan, misalnya `output_v4.txt` di laptop:

- **Kaggle:** K0, K1, dan K2 interaktif (Bagian 3), serta K2 dan K4 dari **setiap** *commit*;
- **Colab:** 11.0, 12.1 (jalan terakhir yang memuat cek akhir), 11.2, 11.3, 11.4, 12.1b, 12.2, 13 sel 1/2, 13 sel 2/2, 14.1 (2 sel), 14.2 (2 sel), 14.3, dan 15 (2 sel).
- Setelah selesai, pilih **File → Download → Download .ipynb** di Colab, dan simpan notebook beserta outputnya di laptop. **Jangan diunggah ke GitHub**, karena repo bersifat publik.

### 10.2 Berkas di Drive (`MyDrive/skripsi_wondr_v4/`) untuk Bab IV

| Tahap | Tabel / gambar |
|---|---|
| 11.4 | `bab4/tabel/tahap11_rs_run.csv` · `hasil/tahap11_rs_terpilih.json` |
| 12.1b | `bab4/tabel/tahap12_pso_run.csv` · `hasil/tahap12_pso_terpilih.json` |
| 12.2 | `bab4/tabel/tahap12_ringkasan_metode.csv` · `bab4/tabel/tahap12_iterasi.csv` · `bab4/gambar/tahap12_konvergensi.png` |
| 13 | `hasil/tahap13_model.json` · `hasil/tahap13_prediksi.parquet` |
| 14.1 | `bab4/tabel/tahap14_model.csv` · `bab4/tabel/tahap14_waktu.csv` |
| 14.2 | `bab4/tabel/tahap14_ansambel.csv` · `bab4/gambar/tahap14_cm_ansambel.png` |
| 14.3 | `bab4/tabel/tahap14_kesesuaian_label.csv` |
| 15 | `bab4/tabel/tahap15_signifikansi.csv` · `bab4/tabel/tahap15_selisih_seed.csv` · `bab4/gambar/tahap15_bootstrap.png` |

Setelah semuanya selesai, buat cadangan: unduh folder `bab4/`, `hasil/`, dan `log/` dari Drive dalam bentuk zip, lalu simpan di laptop atau *flashdisk*.

### 10.3 Saat membuka sesi Claude berikutnya

Tulis, misalnya: *"Lanjut v4 sesuai PANDUAN_MENJALANKAN.md. Sudah sampai Bagian X. Berikut output …"*, lalu tempel output dari 10.1. Claude akan membaca `CLAUDE.md` (Bagian 11) dan menganalisis output itu terhadap target.

---

## Bagian 11 — Masalah yang mungkin muncul

| Gejala | Arti | Yang dilakukan |
|---|---|---|
| Colab 0.1 sel 1/2 menulis **RESTART RUNTIME** | Versi paket baru saja dipasang | **Runtime → Restart session**, lalu ulangi A dari 0.1 sel 1/2 |
| 0.1 sel 2/2 ⚠️ `versi yang dimuat = VERSI · berbeda: …` | Sesi belum di-*restart* setelah pemasangan | **Runtime → Restart session**, lalu ulangi A |
| Galat `GPU sesi ini: … Program ini hanya dijalankan di NVIDIA T4` | *Runtime* bukan T4 | **Runtime → Change runtime type → T4 GPU**, lalu ulangi A dan sel itu |
| Galat `CFG['amp'] masih None` | 0.2 sel 2/2 bukan versi repo, atau 0.2 belum dijalankan | Pastikan baris `"amp": True,` ada di 0.2 sel 2/2, jalankan ulang 0.2, lalu ulangi sel itu |
| Colab: *Runtime disconnected* | Batas waktu sesi atau idle | Lihat Bagian 5.4 (A, lalu sel yang sama; lanjut dari *state*) |
| 11.0 ⚠️ ada `TIDAK DIKENAL` atau `lingkungan tidak sah` | Ada berkas *state* asing atau dari lingkungan lain | **Berhenti**, jangan jalankan 12.1; simpan output 11.0 |
| Kaggle K0 ⚠️ (GPU bukan satu T4 atau versi berbeda) | Pengaturan sesi salah | Periksa *Session options* (GPU T4 x2, Internet On, *Pin to original environment*, *Dependency Manager* kosong), lalu jalankan K0 lagi. Kalau masih ⚠️, berhenti dan simpan output |
| Kaggle galat ``Cell magic `%%proses_baru` not found`` | K0 belum dijalankan di sesi itu | Jalankan K0 dulu, lalu sel itu |
| K1 `Paket tidak ditemukan` | Dataset paket belum ditambahkan sebagai input | **+ Add Input** → `paket-wondr-v4` |
| K1 `Ada lebih dari satu versi paket` | Dua paket berbeda ada di input | *Remove* paket yang lama |
| K2 `UJI KESETARAAN GAGAL` | Kaggle tidak identik dengan Colab | Ikuti **Jalur B** (Bagian 6) |
| K3 `Batas 11 jam tercapai` | Normal; *commit* berhenti rapi | Lakukan *commit* berikutnya (Bagian 4.2) |
| Kuota GPU Kaggle habis | Kuota mingguan terpakai | Tunggu kuota direset, lalu lakukan *commit* lagi (Bagian 4.2) |
| 11.2 galat `Belum ada kaggle/keluaran_kaggle_*.zip` | Zip belum diunggah ke folder `kaggle/` | Unggah zip dari tab *Output* Kaggle ke `MyDrive/skripsi_wondr_v4/kaggle/` |
| 11.2 galat tentang *state* yang tidak identik saat digabung | *State* Colab dan Kaggle bertentangan | **Berhenti**, simpan output; jangan memindahkan berkas sendiri |
| 11.3 **TITIK BERHENTI 2** | Skor Colab ≠ catatan Kaggle | **Berhenti**, simpan output |
| 13 sel 1/2 galat `log/tahap13_state.json berasal dari skema … lain` | *State* Tahap 13 lama tidak cocok | **Berhenti**, simpan output |
| Drive penuh saat Tahap 13 | Bobot "semua" ±7,5 GB | **Berhenti**, simpan output; jangan menghapus berkas apa pun. Penanganannya dibahas bersama Claude |
| Ada ⚠️ lain, atau galat yang tidak ada di tabel ini | — | **Berhenti**, simpan output, dan lanjutkan bersama Claude |
