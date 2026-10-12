# Analisis Tingkat Pengunduran Diri Karyawan
Analisis menyeluruh (end-to-end) mengenai tingkat keluar-masuk karyawan (attrition) menggunakan dataset SDM IBM: MySQL, Python (EDA & ML), Excel, dan dasbor interaktif Power BI.

# Pertanyaan Bisnis & Ruang Lingkup Project

## 1. Latar Belakang

Karyawan adalah tulang punggung sebuah organisasi, dan kinerja organisasi sangat bergantung pada kualitas serta pengalaman mereka. Ketika karyawan keluar (*attrition*), organisasi menghadapi biaya dan waktu untuk merekrut dan melatih pengganti, hilangnya karyawan berpengalaman, serta terganggunya produktivitas tim.

Sebelum menganalisis data, project ini merumuskan pertanyaan bisnis terlebih dahulu, sehingga setiap analisis, model, dan halaman dashboard ada untuk menjawab masalah yang jelas.

## 2. Pengguna Dashboard

Target utama adalah seorang **HR Director** yang ingin mengetahui di mana attrition terkonsentrasi, faktor apa yang berasosiasi dengannya, dan ke mana upaya retensi sebaiknya diprioritaskan lebih dulu.

## 3. Gambaran Dataset

| Item | Keterangan |
|---|---|
| Sumber | IBM HR Analytics Employee Attrition & Performance (Kaggle) |
| Ukuran | 1.470 karyawan, 35 kolom |
| Target | `Attrition` (Yes = 237 atau 16,1%; No = 1.233 atau 83,9%) |
| Kualitas data | Tidak ada missing value dan tidak ada baris duplikat |
| Kolom dibuang | `EmployeeCount`, `Over18`, `StandardHours` (nilainya konstan), `EmployeeNumber` (hanya ID) |
| Sifat data | Dataset fiktif buatan data scientist IBM, sehingga temuan menggambarkan penerapan metode dan bukan kondisi perusahaan nyata |

## 4. Pertanyaan Bisnis

| # | Pertanyaan | Pendekatan | Output Dashboard |
|---|---|---|---|
| **Q1** | Seberapa besar attrition, dan unit mana yang paling terdampak? | Agregasi SQL berdasarkan `Department`, `JobRole`, `JobLevel` | Executive Summary |
| **Q2** | Faktor apa yang paling berasosiasi dengan karyawan yang keluar? | EDA, korelasi / uji chi-square, dan regresi logistik (*odds ratio*) | Attrition Drivers |
| **Q3** | Seperti apa profil karyawan berisiko tinggi, dan seberapa baik model mengenalinya? | Segmentasi dan skor risiko dari regresi logistik, evaluasi model (recall, precision, lift/gain) | Segmentation, Risk Prediction |
| **Q4** | Kebijakan retensi apa yang layak diprioritaskan HR? | Sintesis Q1-Q3 | Bagian Rekomendasi |

## 5. KPI

- Attrition rate (%), jumlah karyawan yang keluar, dan jumlah karyawan aktif
- Attrition rate per departemen, job role, dan job level (selalu ditampilkan bersama jumlah karyawan `n`)
- Rata-rata `MonthlyIncome`, `YearsAtCompany`, dan `JobLevel`: karyawan yang keluar vs yang bertahan
- *Odds ratio* faktor-faktor utama dari regresi logistik
- Jumlah dan persentase karyawan di atas ambang risiko tinggi (dapat diatur dengan slider di Power BI)

## 6. Model & Definisi Sukses

### Model yang digunakan

Project ini memakai **satu model: regresi logistik**, bukan perbandingan banyak algoritma. Alasannya:

- Hasilnya mudah ditafsirkan lewat *odds ratio* (misalnya: peluang keluar karyawan yang lembur sekian kali karyawan yang tidak lembur, dengan faktor lain dijaga tetap).
- Dapat dijelaskan kepada pembaca non-teknis seperti HR.
- Tetap menghasilkan skor risiko untuk dashboard.

Ketidakseimbangan kelas ditangani dengan pembobotan kelas (*class weight*), dan data dibagi menjadi data latih dan data uji dengan proporsi kelas yang dijaga (*stratified split*).

### Definisi sukses

Karena hanya 16,1% karyawan yang keluar, akurasi saja menyesatkan (menebak "bertahan" untuk semua karyawan sudah menghasilkan akurasi sekitar 84%).

- Metrik utama: **recall** (mengukur *false negative*, yaitu yang keluar tetapi terlewat) dan **precision** (mengukur *false positive*, yaitu alarm palsu) untuk kelas "Attrition = Yes", ditambah PR-AUC
- Metrik yang mudah dipahami bisnis: persentase seluruh karyawan yang keluar yang tertangkap dalam 20% karyawan dengan skor risiko tertinggi (lift/gain)
- Ambang risiko dapat digeser pembaca di dashboard untuk melihat *trade-off* antara recall dan precision

## 7. Pedoman Etis untuk Atribut Sensitif

`Gender`, `MaritalStatus`, dan `Age` tetap dipertahankan dalam analisis, dengan batasan berikut:

1. **Deskriptif, bukan preskriptif.** Dipakai untuk menggambarkan pola, tidak pernah menjadi dasar rekomendasi.
2. **Pemeriksaan kewajaran.** Tingkat *false positive* dan *false negative* model dibandingkan antar kelompok gender, status pernikahan, dan usia, dan setiap kesenjangan dilaporkan.
3. **Dengan vs tanpa atribut sensitif.** Regresi logistik dijalankan dengan dan tanpa atribut ini, lalu hasilnya dibandingkan.
4. **Risiko hanya secara agregat.** Dashboard menampilkan risiko per segmen, bukan individu bernama. `EmployeeNumber` tidak ditampilkan.
5. **Hanya pendukung keputusan.** Skor risiko dipakai untuk memprioritaskan program retensi, dan tidak boleh dipakai untuk penilaian kinerja atau keputusan pemutusan hubungan kerja.
6. **Asosiasi, bukan sebab-akibat.** Temuan menggambarkan hubungan antar variabel dalam data.

## 8. Catatan Data & Keterbatasan

- Data fiktif: kesimpulan tidak dapat digeneralisasi ke organisasi nyata.
- `DailyRate`, `HourlyRate`, dan `MonthlyRate` hampir tidak berkorelasi dengan `MonthlyIncome` (sekitar 0), sehingga `MonthlyIncome` dipakai sebagai ukuran gaji.
- Kelompok kecil (misalnya departemen Human Resources, n = 63) menghasilkan persentase yang tidak stabil, sehingga setiap persentase ditampilkan bersama `n`.
- Kolom ordinal yang tersimpan sebagai angka (`Education`, `JobSatisfaction`, `WorkLifeBalance`, dan lainnya) diperlakukan sebagai kategori berurutan.
- Hanya satu model (regresi logistik) yang digunakan demi keterbacaan. Regresi logistik mengasumsikan hubungan linear pada skala log-odds, sehingga model yang lebih kompleks dapat memberi prediksi lebih akurat (pengembangan lanjutan).
- Tidak ada dimensi waktu, sehingga tren dari waktu ke waktu tidak dapat dianalisis.
- Dampak finansial attrition tidak dikuantifikasi karena dataset tidak memuat data biaya (dapat menjadi pengembangan lanjutan).
