# Praktikum 1 OpenMP - Matmul & Bilangan Prima

Proyek ini bertujuan untuk menganalisis performa pemrosesan paralel menggunakan **OpenMP** pada bahasa C melalui dua studi kasus: **Perkalian Matriks ($N=1024$)** dan **Pemeriksaan Bilangan Prima ($N=10^7$)**.

---

## Prasyarat Lingkungan
- **Kompiler**: `gcc` dengan dukungan OpenMP (`-fopenmp`)
- **OS / Shell**: Linux / WSL / Bash Terminal

---

## 1. Kompilasi Program

Kompilasi kedua berkas C menggunakan flag `-fopenmp`:

```bash
# Kompilasi perkalian matriks
gcc -fopenmp src/matmul.c -o matmul

# Kompilasi pemeriksaan bilangan prima
gcc -fopenmp src/prima.c -o prima
```

---

## 2. Cara Menjalankan Eksekusi Eksperimen

### A. Perkalian Matriks (`matmul`)
Menjalankan perkalian matriks dengan jumlah thread $1, 2, 4,$ dan $8$ sebanyak 3 kali pengulangan:

```bash
mkdir -p hasil

for t in 1 2 4 8; do
  for r in 1 2 3; do
    OMP_NUM_THREADS=$t ./matmul >> hasil/matmul.txt
  done
done
```

### B. Bilangan Prima (`prima`)
Menjalankan pengujian kombinasi kebijakan *scheduling* (`static`, `dynamic,1000`, `guided`) dan jumlah thread ($1, 2, 4, 8$) sebanyak 3 kali pengulangan:

```bash
mkdir -p hasil

for s in "static" "dynamic,1000" "guided"; do
  for t in 1 2 4 8; do
    for r in 1 2 3; do
      echo -n "$s;$t;$r;" >> hasil/prima.txt
      OMP_SCHEDULE="$s" OMP_NUM_THREADS=$t ./prima >> hasil/prima.txt
    done
  done
done
```

---

## 3. Struktur Berkas
```text
.
├── src/
│   ├── matmul.c    # Kode sumber Perkalian Matriks
│   └── prima.c     # Kode sumber Bilangan Prima dengan OpenMP
├── hasil/
│   ├── matmul.txt  # Output mentah pengujian perkalian matriks
│   └── prima.txt   # Output mentah pengujian scheduling prima
└── README.md       # Dokumentasi petunjuk penggunaan
```