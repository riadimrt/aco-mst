# b-ACO Borůvka Sollin untuk d-MST dengan Early Stopping

Dokumen ini merangkum **langkah-langkah b-ACO** berbasis **Borůvka atau Sollin** untuk membangun **pohon merentang** dengan **batas derajat** pada setiap simpul. Pendekatan ini memakai mekanisme ACO untuk memilih edge secara probabilistik, lalu menggabungkan komponen seperti Borůvka.

---

## Ringkasan ide

- **Borůvka atau Sollin**: membangun pohon per fase dengan memilih **satu edge keluar terbaik dari setiap komponen**, lalu **menggabungkan komponen** secara serentak.
- **ACO**: pemilihan edge tidak murni greedy, tetapi dipandu **feromon** dan **heuristik biaya**.
- **Batas derajat d**: setiap simpul tidak boleh memiliki derajat lebih dari **d**.
- **Early stopping**: berhenti lebih awal jika solusi terbaik tidak membaik selama beberapa iterasi.

---

## Input dan parameter

**Input utama**
- Graf berbobot, tidak berarah, dengan bobot non-negatif
- Jumlah simpul: **n**
- Batas derajat: **d**

**Parameter ACO**
- **alpha**: pengaruh feromon
- **beta**: pengaruh heuristik biaya
- **rho**: laju evaporasi feromon
- **Q**: konstanta deposit feromon
- **tau0**: feromon awal pada semua edge
- Jumlah semut: **mAnts**
- Iterasi maksimum: **MaxIter**

**Parameter early stopping**
- **k**: jumlah iterasi berturut-turut tanpa perbaikan
- **epsilon**: ambang perubahan kecil, jika digunakan

---

## Flowchart proses b-ACO Borůvka Sollin

```mermaid
flowchart TD
  A[START] --> B[Input<br>Graph berbobot<br>n dan batas derajat d<br>parameter ACO alfa beta rho Q<br>jumlah semut dan iterasi maksimum]
  B --> C[Inisialisasi feromon<br>tau pada semua edge]
  C --> D[Mulai iterasi t dari 1 sampai iterasi maksimum]

  D --> E[Set best global iterasi<br>kosong dan cost sangat besar]
  E --> F[Untuk setiap semut]

  F --> G[Inisialisasi solusi semut<br>E kosong<br>derajat semua simpul 0<br>setiap simpul menjadi 1 komponen]
  G --> H{Apakah jumlah komponen masih lebih dari 1}

  H -->|Ya| I[Untuk setiap komponen<br>buat kandidat edge keluar<br>menuju komponen lain]
  I --> J[Pilih 1 edge per komponen<br>secara probabilistik<br>berdasarkan feromon dan heuristik biaya]
  J --> K{Apakah edge valid<br>tidak membentuk siklus antar komponen<br>dan tidak melanggar batas derajat}
  K -->|Tidak| I
  K -->|Ya| L[Kumpulkan edge terpilih<br>dari semua komponen]
  L --> M[Tambahkan edge yang valid ke solusi semut<br>update derajat simpul]
  M --> N[Gabungkan komponen yang tersambung<br>update struktur komponen]
  N --> H

  H -->|Tidak| O[Hitung cost solusi semut<br>total bobot edge]
  O --> P{Apakah cost lebih baik dari best iterasi}
  P -->|Ya| Q[Simpan sebagai best iterasi]
  P -->|Tidak| R[Lanjut semut berikutnya]
  Q --> R
  R --> S{Apakah semua semut sudah selesai}
  S -->|Tidak| F
  S -->|Ya| T[Update feromon<br>evaporasi dengan rho<br>deposit pada edge solusi terbaik]

  T --> U{Early stopping aktif<br>dan tidak ada perbaikan selama k iterasi}
  U -->|Ya| V[Hentikan lebih awal]
  U -->|Tidak| W{Apakah iterasi maksimum tercapai}
  W -->|Tidak| D
  W -->|Ya| X[Output solusi terbaik global<br>tree memenuhi batas derajat]
  V --> X
  X --> Y[END]
```

---

## Langkah-langkah b-ACO Borůvka Sollin untuk d-MST

1. **Mulai**.

2. **Input**:
   - Graf berbobot
   - Jumlah simpul **n**
   - Batas derajat **d**
   - Parameter ACO: **alpha, beta, rho, Q**
   - Jumlah semut **mAnts**
   - Iterasi maksimum **MaxIter**

3. **Inisialisasi feromon**:
   - Set nilai feromon awal pada semua edge, misalnya **tau0**.

4. **Untuk setiap iterasi** dari 1 sampai **MaxIter**:
   1) Set **solusi terbaik iterasi** sebagai kosong, dan cost terbaik sebagai nilai sangat besar.  
   2) **Untuk setiap semut**:
      - a) **Inisialisasi komponen**  
        Setiap simpul dianggap sebagai 1 komponen.  
        Solusi edge **E = kosong**.  
        Derajat setiap simpul di-set 0.
      - b) **Ulangi selama jumlah komponen lebih dari 1**:
        - i) Untuk setiap komponen, buat daftar **kandidat edge** yang keluar dari komponen menuju komponen lain.
        - ii) Dari kandidat tersebut, pilih **satu edge** menggunakan probabilitas ACO yang dipandu:
          - feromon pada edge, dan
          - heuristik biaya, misalnya kebalikan bobot edge.

          Edge yang dipilih harus memenuhi syarat:
          - tidak membentuk siklus antar komponen, dan
          - tidak melanggar batas derajat:
            - derajat u + 1 tidak lebih dari d
            - derajat v + 1 tidak lebih dari d
        - iii) Kumpulkan edge terpilih dari semua komponen.
        - iv) Tambahkan edge-edge yang **valid** ke solusi.  
          Jika terjadi konflik atau duplikasi, ambil edge yang tetap valid.
        - v) **Update komponen**: gabungkan komponen yang tersambung oleh edge-edge tersebut, dan update derajat simpul.
      - c) Hitung **cost solusi semut** sebagai total bobot semua edge pada solusi.
      - d) Jika cost solusi semut lebih baik dari solusi terbaik iterasi, simpan sebagai **solusi terbaik iterasi**.

5. **Update feromon**:
   - a) **Evaporasi**: kurangi feromon pada semua edge dengan faktor **rho**.
   - b) **Deposit**: tambahkan feromon pada edge yang termasuk solusi terbaik, misalnya proporsional terhadap **Q dibagi cost**.

6. **Cek terminasi opsional**:
   - Jika solusi terbaik tidak membaik selama **k** iterasi berturut-turut, lakukan **early stopping**.

7. **Output**:
   - Solusi terbaik keseluruhan, berupa pohon dengan jumlah edge **n minus 1** dan semua simpul memenuhi batas derajat **d**.

8. **Selesai**.

---

## Catatan praktis

- Pada d-MST, banyak edge murah bisa menjadi tidak feasible karena batas derajat, jadi perlu pemeriksaan feasibility saat pemilihan edge.
- Early stopping berguna karena pada banyak kasus, perbaikan terbesar terjadi di iterasi awal, dan iterasi lanjutan sering memberi peningkatan kecil.
