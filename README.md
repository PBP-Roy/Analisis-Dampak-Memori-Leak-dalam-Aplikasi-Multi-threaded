---

# Analisis Dampak Memori Leak dalam Aplikasi Multi-threaded

## Deskripsi Proyek

Eksperimen ini bertujuan untuk menganalisis dampak dari memori leak dalam aplikasi multi-threaded. Memori leak adalah kondisi di mana memori yang dialokasikan oleh aplikasi tidak dibebaskan kembali setelah selesai digunakan. Dalam aplikasi multi-threaded, memori leak dapat menyebabkan crash yang sulit didiagnosis, penurunan performa, dan kehabisan memori sistem lebih cepat karena beberapa thread secara paralel mengalokasikan memori tanpa membebaskannya.

## Tujuan Eksperimen

1. **Menganalisis Dampak Memori Leak**: Mengevaluasi bagaimana memori leak dapat mempengaruhi stabilitas dan performa aplikasi multi-threaded.
2. **Memahami Pola Peningkatan Memori**: Mengidentifikasi pola konsumsi memori dalam aplikasi dengan banyak thread yang mengalami memori leak.
3. **Evaluasi Kompleksitas Diagnostik**: Menilai kompleksitas yang dihadapi dalam proses debugging ketika memori leak terjadi di lingkungan multi-threaded.

## Metodologi Eksperimen

Eksperimen dilakukan dengan mengembangkan sebuah aplikasi multi-threaded sederhana yang secara sengaja dibuat untuk memiliki memori leak. Berikut langkah-langkah dalam metodologi eksperimen ini:

1. **Membangun Aplikasi Multi-threaded**: Mengembangkan aplikasi yang terdiri dari beberapa thread, masing-masing mengalokasikan memori tanpa pernah membebaskannya.
2. **Monitoring Penggunaan Memori**: Menjalankan aplikasi dan memantau penggunaan memori dari waktu ke waktu menggunakan alat monitoring sistem.
3. **Evaluasi Dampak**: Mengukur dampak memori leak dengan memperhatikan waktu hingga sistem kehabisan memori atau aplikasi mengalami crash.
4. **Analisis Hasil**: Menganalisis hasil yang diperoleh untuk memahami bagaimana memori leak mempengaruhi aplikasi multi-threaded.

## Pelaksanaan Eksperimen

### Langkah 1: Implementasi Aplikasi Multi-threaded

Kode berikut ini digunakan untuk membuat aplikasi multi-threaded dengan memori leak:

```c
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>

#define NUM_THREADS 10
#define ALLOCATION_SIZE 1024 * 1024 // 1 MB

DWORD WINAPI leak_memory(LPVOID lpParam) {
    int thread_id = *(int *)lpParam;
    printf("Thread %d started\n", thread_id);

    while (1) {
        void *memory = malloc(ALLOCATION_SIZE); // Allocate 1 MB memory
        if (memory == NULL) {
            printf("Thread %d: Memory allocation failed\n", thread_id);
            return 1;
        }
        // Simulate some work with the memory (optional)
        for (int i = 0; i < ALLOCATION_SIZE; i++) {
            ((char*)memory)[i] = 0; // Touch memory to prevent optimization
        }
        // Note: No free() here, causing a memory leak
        Sleep(1000); // Slow down the loop for observation
    }
    return 0;
}

int main() {
    HANDLE threads[NUM_THREADS];
    int thread_ids[NUM_THREADS];

    // Create multiple threads
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_ids[i] = i;
        threads[i] = CreateThread(
            NULL,       // Default security attributes
            0,          // Default stack size
            leak_memory,  // Thread function
            &thread_ids[i], // Thread function argument
            0,          // Default creation flags
            NULL);      // Ignore the thread identifier

        if (threads[i] == NULL) {
            printf("Error: Unable to create thread %d\n", i);
            return 1;
        }
    }

    // Wait for all threads to finish (they won't in this case)
    WaitForMultipleObjects(NUM_THREADS, threads, TRUE, INFINITE);

    // Close thread handles
    for (int i = 0; i < NUM_THREADS; i++) {
        CloseHandle(threads[i]);
    }

    return 0;
}

```

### Langkah 2: Kompilasi dan Eksekusi

1. **Kompilasi Program**:
   ```bash
   gcc -o memory_leak memory_leak.c -lpthread
   ```

2. **Eksekusi Program**:
   ```bash
   ./memory_leak
   ```

3. **Monitoring Penggunaan Memori**:
   Gunakan alat monitoring seperti `top`, `htop`, atau `Task Manager` untuk memantau penggunaan memori oleh program.

### Langkah 3: Analisis Hasil

- **Penggunaan Memori**: Program akan terus mengalokasikan memori hingga seluruh memori sistem habis.
- **Crash**: Aplikasi mungkin akan mengalami crash saat sistem tidak bisa lagi menyediakan memori.
- **Performa**: Seiring waktu, Anda mungkin akan melihat penurunan performa sistem seiring dengan meningkatnya penggunaan memori.

## Hasil Eksperimen

Eksperimen menunjukkan bahwa memori leak dalam aplikasi multi-threaded dapat dengan cepat menghabiskan memori sistem, terutama ketika banyak thread aktif secara paralel. Hasil utama dari eksperimen ini adalah:

1. **Peningkatan Penggunaan Memori**: Memori sistem secara konstan menurun karena memori yang dialokasikan tidak pernah dibebaskan.
2. **Crash Sistem**: Aplikasi akhirnya mengalami crash ketika sistem kehabisan memori.
3. **Kesulitan Diagnostik**: Memori leak di lingkungan multi-threaded bisa sulit didiagnosis, karena setiap thread mungkin memicu masalah pada waktu yang berbeda.

## Kesimpulan

Memori leak dalam aplikasi multi-threaded merupakan masalah serius yang dapat menyebabkan crash, penurunan performa, dan kesulitan dalam debugging. Eksperimen ini mengilustrasikan pentingnya manajemen memori yang tepat dalam pengembangan aplikasi multi-threaded untuk menjaga stabilitas dan kinerja sistem.
