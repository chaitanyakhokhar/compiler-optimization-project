# compiler-optimization-project
/*
 * Optimized QuickSort: Pivot Selection & Partitioning Strategies
 * Compile:  gcc -O2 -o pbl.c -lm
 * Run:      ./quicksort
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <math.h>
typedef struct {
    long long comparisons;
    long long swaps;
    int       max_depth;
    double    time_ms;
} Metrics;

static void metrics_reset(Metrics *m) {
    m->comparisons = 0;
    m->swaps       = 0;
    m->max_depth   = 0;
    m->time_ms     = 0.0;
}
static inline void swap(int *a, int *b, Metrics *m) {
    int t = *a; *a = *b; *b = t;
    m->swaps++;
}
static inline void swap_raw(int *a, int *b) {
    int t = *a; *a = *b; *b = t;
}
static void insertion_sort(int arr[], int lo, int hi, Metrics *m) {
    for (int i = lo + 1; i <= hi; i++) {
        int key = arr[i], j = i - 1;
        while (j >= lo) {
            m->comparisons++;
            if (arr[j] > key) {
                arr[j + 1] = arr[j];
                m->swaps++;
                j--;
            } else break;
        }
        arr[j + 1] = key;
    }
}
static void heap_sort(int arr[], int lo, int hi, Metrics *m) {
    int n = hi - lo + 1;
    int *sub = arr + lo;
    for (int s = (n - 2) / 2; s >= 0; s--) {
        int root = s;
        while (2 * root + 1 < n) {
            int child = 2 * root + 1;
            if (child + 1 < n) {
                m->comparisons++;
                if (sub[child] < sub[child + 1]) child++;
            }
            m->comparisons++;
            if (sub[root] < sub[child]) {
                int t = sub[root]; sub[root] = sub[child]; sub[child] = t;
                m->swaps++;
                root = child;
            } else break;
        }
    }
    for (int end = n - 1; end > 0; end--) {
        int t = sub[0]; sub[0] = sub[end]; sub[end] = t;
        m->swaps++;
        int root = 0;
        while (2 * root + 1 < end) {
            int child = 2 * root + 1;
            if (child + 1 < end) {
                m->comparisons++;
                if (sub[child] < sub[child + 1]) child++;
            }
            m->comparisons++;
            if (sub[root] < sub[child]) {
                int tmp = sub[root]; sub[root] = sub[child]; sub[child] = tmp;
                m->swaps++;
                root = child;
            } else break;
        }
    }
}

static int partition_lomuto(int arr[], int lo, int hi, Metrics *m) {
    int pivot = arr[hi];
    int i = lo - 1;
    for (int j = lo; j < hi; j++) {
        m->comparisons++;
        if (arr[j] <= pivot) {
            i++;
            swap(&arr[i], &arr[j], m);
        }
    }
    swap(&arr[i + 1], &arr[hi], m);
    return i + 1;
}


static void qs_naive(int arr[], int lo, int hi, Metrics *m, int depth) {
    if (depth > m->max_depth) m->max_depth = depth;
    if (lo < hi) {
        swap(&arr[lo], &arr[hi], m);          
        int p = partition_lomuto(arr, lo, hi, m);
        qs_naive(arr, lo, p - 1, m, depth + 1);
        qs_naive(arr, p + 1, hi, m, depth + 1);
    }
}


static void qs_random(int arr[], int lo, int hi, Metrics *m, int depth) {
    if (depth > m->max_depth) m->max_depth = depth;
    if (lo < hi) {
        int ri = lo + rand() % (hi - lo + 1);
        swap(&arr[ri], &arr[hi], m);
        int p = partition_lomuto(arr, lo, hi, m);
        qs_random(arr, lo, p - 1, m, depth + 1);
        qs_random(arr, p + 1, hi, m, depth + 1);
    }
}


static void med3_select(int arr[], int lo, int hi, Metrics *m) {
    int mid = lo + (hi - lo) / 2;
    m->comparisons += 3;
    if (arr[lo] > arr[mid]) swap(&arr[lo], &arr[mid], m);
    if (arr[lo] > arr[hi])  swap(&arr[lo], &arr[hi], m);
    if (arr[mid] > arr[hi]) swap(&arr[mid], &arr[hi], m);
    swap(&arr[mid], &arr[hi], m);  
}

static void qs_median3(int arr[], int lo, int hi, Metrics *m, int depth) {
    if (depth > m->max_depth) m->max_depth = depth;
    if (lo < hi) {
        med3_select(arr, lo, hi, m);
        int p = partition_lomuto(arr, lo, hi, m);
        qs_median3(arr, lo, p - 1, m, depth + 1);
        qs_median3(arr, p + 1, hi, m, depth + 1);
    }
}


static int med3_idx(int arr[], int a, int b, int c, Metrics *m) {
    m->comparisons += 3;
    if (arr[a] > arr[b]) { int t = a; a = b; b = t; }
    if (arr[a] > arr[c]) { int t = a; a = c; c = t; }
    if (arr[b] > arr[c]) { int t = b; b = c; c = t; }
    return b;
}

static void qs_ninther(int arr[], int lo, int hi, Metrics *m, int depth) {
    if (depth > m->max_depth) m->max_depth = depth;
    if (lo < hi) {
        int n = hi - lo + 1;
        if (n < 9) {
            med3_select(arr, lo, hi, m);
        } else {
            int step = n / 8;
            int m1 = med3_idx(arr, lo, lo+step, lo+2*step, m);
            int m2 = med3_idx(arr, lo+3*step, lo+4*step, lo+5*step, m);
            int m3 = med3_idx(arr, lo+6*step, lo+7*step, hi, m);
            int pivot_idx = med3_idx(arr, m1, m2, m3, m);
            swap(&arr[pivot_idx], &arr[hi], m);
        }
        int p = partition_lomuto(arr, lo, hi, m);
        qs_ninther(arr, lo, p - 1, m, depth + 1);
        qs_ninther(arr, p + 1, hi, m, depth + 1);
    }
}


static void qs_dualpivot(int arr[], int lo, int hi, Metrics *m, int depth) {
    if (depth > m->max_depth) m->max_depth = depth;
    if (lo >= hi) return;

    m->comparisons++;
    if (arr[lo] > arr[hi]) swap(&arr[lo], &arr[hi], m);

    int p = arr[lo], q = arr[hi];
    int l = lo + 1, g = hi - 1, k = lo + 1;

    while (k <= g) {
        m->comparisons++;
        if (arr[k] < p) {
            swap(&arr[k], &arr[l], m);
            l++;
        } else {
            m->comparisons++;
            if (arr[k] > q) {
                while (arr[g] > q && k < g) { m->comparisons++; g--; }
                swap(&arr[k], &arr[g], m);
                g--;
                m->comparisons++;
                if (arr[k] < p) {
                    swap(&arr[k], &arr[l], m);
                    l++;
                }
            }
        }
        k++;
    }
    l--; g++;
    swap(&arr[lo], &arr[l], m);
    swap(&arr[hi], &arr[g], m);

    qs_dualpivot(arr, lo, l - 1, m, depth + 1);
    qs_dualpivot(arr, l + 1, g - 1, m, depth + 1);
    qs_dualpivot(arr, g + 1, hi, m, depth + 1);
}


#define INSERTION_THRESHOLD 16

static void qs_hybrid_impl(int arr[], int lo, int hi, Metrics *m,
                            int depth, int depth_limit) {
    while (lo < hi) {
        if (depth > m->max_depth) m->max_depth = depth;


        if (hi - lo + 1 <= INSERTION_THRESHOLD) {
            insertion_sort(arr, lo, hi, m);
            return;
        }

        if (depth >= depth_limit) {
            heap_sort(arr, lo, hi, m);
            return;
        }

        if (hi - lo >= 2)
            med3_select(arr, lo, hi, m);
        int p = partition_lomuto(arr, lo, hi, m);

        if (p - lo < hi - p) {
            qs_hybrid_impl(arr, lo, p - 1, m, depth + 1, depth_limit);
            lo = p + 1;
        } else {
            qs_hybrid_impl(arr, p + 1, hi, m, depth + 1, depth_limit);
            hi = p - 1;
        }
        depth++;
    }
}

static void qs_hybrid(int arr[], int lo, int hi, Metrics *m, int depth) {
    (void)depth;
    int depth_limit = 2 * (int)(log2((double)(hi - lo + 1)));
    qs_hybrid_impl(arr, lo, hi, m, 0, depth_limit);
}


static void gen_random(int arr[], int n) {
    for (int i = 0; i < n; i++) arr[i] = rand() % (n * 10);
}
static void gen_sorted(int arr[], int n) {
    for (int i = 0; i < n; i++) arr[i] = i;
}
static void gen_reverse(int arr[], int n) {
    for (int i = 0; i < n; i++) arr[i] = n - i;
}
static void gen_nearly_sorted(int arr[], int n) {
    for (int i = 0; i < n; i++) arr[i] = i;
    int swaps = n / 20;  
    for (int s = 0; s < swaps; s++) {
        int a = rand() % n, b = rand() % n;
        swap_raw(&arr[a], &arr[b]);
    }
}
static void gen_duplicates(int arr[], int n) {
    for (int i = 0; i < n; i++) arr[i] = rand() % 10;  
}


static int is_sorted(int arr[], int n) {
    for (int i = 1; i < n; i++)
        if (arr[i] < arr[i-1]) return 0;
    return 1;
}


typedef void (*SortFunc)(int[], int, int, Metrics*, int);

typedef struct {
    const char *name;
    SortFunc    func;
} Strategy;

typedef struct {
    const char *name;
    void (*gen)(int[], int);
} Workload;

int main(void) {
    srand((unsigned)time(NULL));

    Strategy strategies[] = {
        {"Naive (First)",   qs_naive},
        {"Random Pivot",    qs_random},
        {"Median-of-3",    qs_median3},
        {"Ninther",         qs_ninther},
        {"Dual-Pivot",      qs_dualpivot},
        {"Hybrid",          qs_hybrid}
    };
    int nstrat = sizeof(strategies) / sizeof(strategies[0]);

    Workload workloads[] = {
        {"Random",         gen_random},
        {"Sorted",         gen_sorted},
        {"Reverse",        gen_reverse},
        {"Nearly Sorted",  gen_nearly_sorted},
        {"Many Duplicates",gen_duplicates}
    };
    int nwork = sizeof(workloads) / sizeof(workloads[0]);

    int sizes[] = {1000, 5000, 10000, 50000};
    int nsizes = sizeof(sizes) / sizeof(sizes[0]);

    printf("\n================ QUICK SORT PERFORMANCE ANALYSIS ================\n");

for (int si = 0; si < nsizes; si++) {
    int n = sizes[si];
    int *original = (int *)malloc(n * sizeof(int));
    int *working  = (int *)malloc(n * sizeof(int));

    printf("\n\n===============================================================\n");
    printf("Array Size: %d\n", n);
    printf("===============================================================\n");

    for (int wi = 0; wi < nwork; wi++) {
        workloads[wi].gen(original, n);

        printf("\n--- Workload: %s ---\n", workloads[wi].name);
        printf("--------------------------------------------------------------------------\n");
        printf("%-15s | %-12s | %-10s | %-10s | %-10s | %-8s\n",
               "Strategy", "Comparisons", "Swaps", "MaxDepth", "Time(ms)", "Correct");
        printf("--------------------------------------------------------------------------\n");

        for (int st = 0; st < nstrat; st++) {

            if (st == 0 && n > 10000 && (wi == 1 || wi == 2)) continue;

            memcpy(working, original, n * sizeof(int));
            Metrics m;
            metrics_reset(&m);

            struct timespec t0, t1;
            clock_gettime(CLOCK_MONOTONIC, &t0);
            strategies[st].func(working, 0, n - 1, &m, 0);
            clock_gettime(CLOCK_MONOTONIC, &t1);

            m.time_ms = (t1.tv_sec - t0.tv_sec) * 1000.0 + (t1.tv_nsec - t0.tv_nsec) / 1e6;
            int correct = is_sorted(working, n);
            printf("%-15s | %-12lld | %-10lld | %-10d | %-10.3f | %-8s\n",
                   strategies[st].name,
                   m.comparisons,
                   m.swaps,
                   m.max_depth,
                   m.time_ms,
                   correct ? "YES" : "NO");
        }
    }

    free(original);
    free(working);
}

printf("\n=========================== END ===========================\n");
}

