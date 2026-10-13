# Laporan Praktikum 1: Pemrograman Shared Memory dengan OpenMP
**Mata Kuliah:** Pemrosesan Paralel  
**Nama:** Muhammad Farhan Abdillah  
**NIM:** 09011482528006  

---

## 📋 Lingkungan Pengujian Sistem

| Komponen | Spesifikasi |
| :--- | :--- |
| **Prosesor** | Intel Core i7-1165G7 @ 2.80GHz |
| **Core Fisik / Logis** | 4 Core Fisik / 8 Core Logis |
| **Memori (RAM)** | 16 GB RAM |
| **Versi GCC** | GCC 11.4.0 |
| **Opsi Kompilasi** | `-O2 -fopenmp -lm` |

---

## 📁 Struktur Direktori

```text
p1-openmp/
├── src/
│   ├── matmul.c
│   └── prima.c
├── hasil/
│   ├── matmul.txt
│   └── prima.txt
└── README.md
```

---

## 🚀 Cara Kompilasi dan Menjalankan Program

### 1. Persiapan Direktori Kerja
```bash
mkdir -p p1-openmp/src p1-openmp/hasil
cd p1-openmp
```

### 2. Perkalian Matriks Paralel (`matmul.c`)
**Kompilasi:**
```bash
gcc -O2 -fopenmp src/matmul.c -o matmul -lm
```

**Menjalankan (Ukuran Matriks $n=1536$):**
```bash
# Menjalankan dengan 4 Thread
OMP_NUM_THREADS=4 ./matmul 1536

# Menjalankan dengan variasi Thread (1, 2, 4, 8)
for t in 1 2 4 8; do
    echo "=== Running with $t Threads ==="
    OMP_NUM_THREADS=$t ./matmul 1536
done
```

### 3. Penghitungan Bilangan Prima & Scheduling (`prima.c`)
**Kompilasi:**
```bash
gcc -O2 -fopenmp src/prima.c -o prima -lm
```

**Menjalankan dengan Pengaturan `OMP_SCHEDULE`:**
```bash
# Contoh running dengan penjadwalan dynamic, chunk 1000 dan 4 Thread
OMP_SCHEDULE="dynamic,1000" OMP_NUM_THREADS=4 ./prima

# Uji coba dengan berbagai kebijakan scheduling
OMP_SCHEDULE="static" OMP_NUM_THREADS=8 ./prima
OMP_SCHEDULE="dynamic,1000" OMP_NUM_THREADS=8 ./prima
OMP_SCHEDULE="guided" OMP_NUM_THREADS=8 ./prima
```

---

## 📊 Hasil Pengamatan & Kinerja

### 1. Perkalian Matriks ($n = 1536$)

| Thread | Waktu Serial (s) | Waktu Paralel (s) | Speedup | Efisiensi | Hasil Benar? |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | 8.452 | 8.510 | 0.99 | 0.99 | Ya |
| **2** | 8.452 | 4.320 | 1.96 | 0.98 | Ya |
| **4** | 8.452 | 2.215 | 3.82 | 0.95 | Ya |
| **8** | 8.452 | 1.348 | 6.27 | 0.78 | Ya |

### 2. Bilangan Prima ($N = 10^7$, Target Prima = 664.579)

| Kebijakan Scheduling | 1 Thread (s) | 2 Thread (s) | 4 Thread (s) | 8 Thread (s) | Speedup (8 Th) | Hasil Benar? |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **`static`** | 1.420 | 0.740 | 0.395 | 0.290 | 4.90x | Ya |
| **`dynamic,1000`** | 1.440 | 0.735 | 0.380 | 0.245 | **5.88x** | Ya |
| **`guided`** | 1.435 | 0.730 | 0.375 | 0.250 | 5.74x | Ya |

---

## 💻 Potongan Kode Kunci

### Bagian A: `matmul_paralel`
```c
static void matmul_paralel(const double *A, const double *B, double *C, int n) {
    #pragma omp parallel for collapse(2) schedule(static)
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            double s = 0.0;
            for (int k = 0; k < n; k++) {
                s += A[(long)i * n + k] * B[(long)k * n + j];
            }
            C[(long)i * n + j] = s;
        }
    }
}
```

### Bagian B: `prima.c`
```c
#pragma omp parallel for reduction(+:jumlah) schedule(runtime)
for (long x = 2; x <= N; x++) {
    jumlah += is_prima(x);
}
```

---

## 📝 Ringkasan Analisis

1. **Perkalian Matriks:** Penggunaan `#pragma omp parallel for collapse(2)` efektif membagi iterasi. Variabel lokal `s` dan `k` bersifat private secara otomatis untuk menghindari *race condition*. Speedup linier hingga 4 core fisik (3.82x), kemudian efisiensi menurun pada 8 thread (0.78) akibat pembatasan Hyper-Threading 4 core fisik.
2. **Scheduling Bilangan Prima:** Kebijakan `dynamic,1000` dan `guided` memberikan performa terbaik (Speedup 5.88x) karena mampu mengatasi masalah *load imbalance* (pengujian prima untuk angka besar memakan waktu lebih lama daripada angka kecil).
3. **Klausa Reduction:** Tanpa klausa `reduction(+:jumlah)`, terjadi *race condition* yang mengakibatkan *lost update* sehingga hasil akhir penjumlahan tidak tepat dan berubah-ubah.