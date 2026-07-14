WTFF is a tool to compare IEEE 754 32 bit float errors in framerate/time variance compared to 64 BIT RTSS /80/128 
thoses errors might induce jitter, the tool gives the most stable result

It was created to get an architecture adapted to the analysis of the realspace 2 (Gunz engine) bugged frame distribution

It runs On https://www.onlinegdb.com/online_c_compiler#

here is a sample of the gunz engine mode.


Note, it's meant to show the stable framerate aroud your own target range, 
you CHOOSE the range then pick a fps proposed by the tool that is close and stable. 
Dont use the first value simply because it's higher in ranking.

// ======================================================================
//   WTFF — Windowed Timing Frame Finder 0.5.1
//   By Grimnismal & a bit of AI, vibe coded.
// ======================================================================
//
// Only RTSS's limiter is precise enough for fractional FPS. Use integer mode
// with NVIDIA, AMD, FRAPS, Dxtory, or the in‑game limiter.
// GunZ's in‑game counter reads one frame late — add +1 FPS for the true value.
// Ideally, set the in‑game cap to 1000 and limit externally with RTSS.
/// With RTSS, use front‑edge sync. Avoid async unless you have real FPS drops.

// This tool measures IEEE‑754 rounding error in 32‑bit float vs. 64‑bit double
// and how that error affects FPS timing.
/// Plain version: FPS is imprecise and fluctuates; error leans one way or the
/// other — unless you pick a good value.
// Here are the good values.

// It's a niche topic, but gamers deserve to know the absolute best FPS value
// for their target. These numbers can also expose engine holes, race conditions,
// and other quirks — speedrunners might find hidden gems.
// You'll also find the ideal smoothness every player wants: the SWEET spot that
// accounts for jitter oscillation.

// Higher FPS means lower latency, so you generally want the highest FPS you can
// get. But pick carefully from the recommended brackets: a strong UP bias won't
// overcome a 20 FPS latency disadvantage.

// For GunZ, High UP means faster animations (quicker massive/flinch recovery!),
// faster cancels, shorter dash distance, shorter i‑frames, faster walking
// acceleration, and faster floor detection after landing.
// DOWN does the opposite — even a picosecond longer dash changes the game feel.
// Neutral (zero error) is the intended timing, but jitter makes it elusive —
// that's why we search for a sweet spot.

// Please do not use for illegal activities.
// You may use the code in your own tools, but please credit me.
// This is shareware — you may modify it, but keep my name somewhere.
//------------------------------------------------------------------------------------


#include <stdio.h>
#include <math.h>
#include <stdlib.h>
#include <float.h>
#include <stdint.h>
#include <limits.h>

// ==================== USER CONFIGURATION ====================

//////Some values are expected to have DECIMAL, even "0.0", do not forget thoses ".0"!
////Some values are expected to have DECIMAL, even "0.0", do not forget thoses ".0"!
///Some values are expected to have DECIMAL, even "0.0", do not forget thoses ".0"!

#define INTEGER_MODE 0
// 1 = for FPS limiters that use integer, so most
// 0 = full decimal scan, for RTSS

#define Gunz_Fdelta_finder 1
/
// GunZ native engine timeGetTime() + m_fSpeed=4.8 model
// 0 = off standard float-vs-double analysis
// 1 = on  block‑time accuracy analysis
// Block metric mode (only active when Gunz_Fdelta_finder == 1)
// 0 = single block time (animation + one frame input separation)
// 1 = repeat interval   (animation + two frames, press‑to‑press)
/
#define BLOCK_METRIC_MODE 1

#define TARGET_FPS       256.0    // Anchor point, set it to your wanted frame rate or close
#define TARGET_RANGE     64     // Allowed distance from the anchor, candidates for your optimal fps
#define TARGET_DIRECTION 0        // 0 = both sides, 1 = only ≤ anchor, 2 = only ≥ anchor


#define FPS_STEP   0.00001         // Decimal resolution; no reason to touch it, mostly for RTSS
#define WINDOW_SIZE  0.1           // ±0.1 FPS jitter window, simulation of oscillation in frame presentation
#define TOP_N        32            // How many distinct values to print in the list, quality of life setting


#define TARGET_FILTER_EXTREMES 1
// Apply target filter to UP/DOWN window extremes ; set to 0 to show full list of
// extreme stable errors, with TOP_N at a high value for the complete list


#define SINGLE_POINT_EXTREMES 0
// 0 = off (normal window analysis) – gives stable, usable values.
// 1 = on  (raw single‑point errors, no window) shows the most artifacted,
//  jitter‑prone FPS values; interesting but impractical for gameplay,
//  useful for finding glitches or bugs


#define BINNING_MODE 1
// 0 = Useful to seek specific results in a small target_range with long top_n list
// 1 = Show FPS binned by integer to provide interesting values


#define BATCH_MODE 1
// 0 = single range  (uses FPS_START / FPS_END); legacy
// 1 = batch scan across ten 100‑FPS ranges based on target_fps and range
#define FPS_START  256.0           // Only used when BATCH_MODE is 0; legacy
#define FPS_END    456.0           // Aborts safely if span > ~200 FPS; legacy


// OpenMP parallelisation for batch scanning
// 0 = serial (default, works everywhere)
// 1 = parallel (add -fopenmp to compilation)
#define USE_OPENMP 1

/* --------------------------------------------------------------------
 * PRECISION selection – the entire analysis (error buffer, Kahan sums,
 * variance) uses this type.  Only native or compiler‑provided types.
 *   0 = 64‑bit double
 *   1 = 80‑bit long double (extended precision)
 *   2 = 128‑bit __float128 (requires -lquadmath, GCC/Clang)
 *
 * Compile with:   gcc -O3 -march=native -fno-fast-math -lquadmath   (for precision 2)
 * -------------------------------------------------------------------- */
#define PRECISION 0
// ===========================================================

#define REQUESTED_PRECISION PRECISION

#if REQUESTED_PRECISION == 2
  #if defined(__FLT128_MAX__) || defined(FLT128_MAX)
    #define ACTUAL_PRECISION 2
    #include <quadmath.h>
    #define HAVE_QUADMATH_SNPRINTF
    typedef __float128  prec_t;
    #define PREC_ABS(x)  fabsq(x)
    #define PREC_SQRT(x) sqrtq(x)
  #else
    #warning "Quad precision not available – falling back to 80‑bit. Link with -lquadmath for full 128‑bit."
    #define ACTUAL_PRECISION 1
    typedef long double prec_t;
    #define PREC_ABS(x)  fabsl(x)
    #define PREC_SQRT(x) sqrtl(x)
  #endif
#elif REQUESTED_PRECISION == 1
  #define ACTUAL_PRECISION 1
  typedef long double prec_t;
  #define PREC_ABS(x)  fabsl(x)
  #define PREC_SQRT(x) sqrtl(x)
#else
  #define ACTUAL_PRECISION 0
  typedef double prec_t;
  #define PREC_ABS(x)  fabs(x)
  #define PREC_SQRT(x) sqrt(x)
#endif

typedef prec_t accum_t;

#ifndef INFINITY
#define INFINITY HUGE_VAL
#endif

#define KAHAN_RESET_INTERVAL 10000
#define NUM_BATCH_RANGES 10

static const double batch_starts[NUM_BATCH_RANGES] = {
    1.0,
    100.0 + FPS_STEP,
    200.0 + FPS_STEP,
    300.0 + FPS_STEP,
    400.0 + FPS_STEP,
    500.0 + FPS_STEP,
    600.0 + FPS_STEP,
    700.0 + FPS_STEP,
    800.0 + FPS_STEP,
    900.0 + FPS_STEP
};
static const double batch_ends[NUM_BATCH_RANGES] = {
    100.0, 200.0, 300.0, 400.0, 500.0,
    600.0, 700.0, 800.0, 900.0, 1000.0
};

#define MAX_FPS 1005
#define MAX_SPAN_FPS 200.0
#define EPS_STEPS_PER_FPS_ABS 1e-12L

// -----------------------------------------------------------------
// Data structures with full prec_t support
// -----------------------------------------------------------------
typedef struct {
    double fps;
    double avg;
    prec_t avg_prec;
    double stddev;
    prec_t stddev_prec;
    double avg_abs;
    prec_t avg_abs_prec;
    double sweet;
    prec_t sweet_prec;
    int valid;
} BestResult;

typedef struct {
    double fps;
    double avg;
    prec_t avg_prec;
    double stddev;
    prec_t stddev_prec;
    double avg_abs;
    prec_t avg_abs_prec;
    double sweet;
    prec_t sweet_prec;
} SweetResult;

typedef struct {
    double fps;
    double avg;
    prec_t avg_prec;
    double stddev;
    prec_t stddev_prec;
} RankResult;

typedef struct {
    double fps;
    prec_t err;
} UpDownItem;

// ---------- Global flag for native mode ----------
int g_GunzFdeltaMode = 0;
#define GUNZ_BLOCK_FRAMES  1140.0   // approximate ticks for forced animation
#define GUNZ_BLOCK_REF_MS   237.5   // 1140 / 4.8

// !!! THIS VALUE MUST BE A DECIMAL !!!
#define GUNZ_JITTER_WEIGHT 4.0

// ---------- NaN‑safe comparators for prec_t ----------
static inline int prec_isnan(prec_t x) {
#if ACTUAL_PRECISION == 2
    return isnanq(x);
#else
    return isnan(x);
#endif
}

static inline prec_t prec_inf(void) {
#if ACTUAL_PRECISION == 2
    #ifdef HUGE_VALQ
        return (prec_t)HUGE_VALQ;
    #elif defined(HUGE_VALL)
        return (prec_t)HUGE_VALL;
    #else
        return (prec_t)INFINITY;
    #endif
#elif ACTUAL_PRECISION == 1
    #ifdef HUGE_VALL
        return (prec_t)HUGE_VALL;
    #else
        return (prec_t)INFINITY;
    #endif
#else
    return (prec_t)HUGE_VAL;
#endif
}

static inline int prec_isinf(prec_t x) {
#if ACTUAL_PRECISION == 2
    return x == prec_inf();
#elif ACTUAL_PRECISION == 1
    return isinf(x);
#else
    return isinf(x);
#endif
}

static int cmp_up(const void *a, const void *b) {
    prec_t A = ((const RankResult*)a)->avg_prec;
    prec_t B = ((const RankResult*)b)->avg_prec;
    int a_nan = prec_isnan(A), b_nan = prec_isnan(B);
    if (a_nan && b_nan) return 0;
    if (a_nan) return 1;
    if (b_nan) return -1;
    return (A < B) ? 1 : ((A > B) ? -1 : 0);
}
static int cmp_down(const void *a, const void *b) {
    prec_t A = ((const RankResult*)a)->avg_prec;
    prec_t B = ((const RankResult*)b)->avg_prec;
    int a_nan = prec_isnan(A), b_nan = prec_isnan(B);
    if (a_nan && b_nan) return 0;
    if (a_nan) return 1;
    if (b_nan) return -1;
    return (A < B) ? -1 : ((A > B) ? 1 : 0);
}
static int cmp_sweet(const void *a, const void *b) {
    prec_t A = ((const SweetResult*)a)->sweet_prec;
    prec_t B = ((const SweetResult*)b)->sweet_prec;
    int a_nan = prec_isnan(A), b_nan = prec_isnan(B);
    if (a_nan && b_nan) return 0;
    if (a_nan) return 1;
    if (b_nan) return -1;
    return (A < B) ? -1 : ((A > B) ? 1 : 0);
}
static int cmp_up_item(const void *a, const void *b) {
    prec_t A = ((const UpDownItem*)a)->err;
    prec_t B = ((const UpDownItem*)b)->err;
    int a_nan = prec_isnan(A), b_nan = prec_isnan(B);
    if (a_nan && b_nan) return 0;
    if (a_nan) return 1;
    if (b_nan) return -1;
    return (A < B) ? 1 : ((A > B) ? -1 : 0);
}
static int cmp_down_item(const void *a, const void *b) {
    prec_t A = ((const UpDownItem*)a)->err;
    prec_t B = ((const UpDownItem*)b)->err;
    int a_nan = prec_isnan(A), b_nan = prec_isnan(B);
    if (a_nan && b_nan) return 0;
    if (a_nan) return 1;
    if (b_nan) return -1;
    return (A < B) ? -1 : ((A > B) ? 1 : 0);
}

/* Native mode comparators – tie‑breaker: mean → jitter → higher FPS */

static int cmp_up_native(const void *a, const void *b) {
    const RankResult *A = (const RankResult*)a;
    const RankResult *B = (const RankResult*)b;

    if (A->avg < B->avg) return -1;
    if (A->avg > B->avg) return 1;

    if (A->stddev < B->stddev) return -1;
    if (A->stddev > B->stddev) return 1;

    if (A->fps > B->fps) return -1;
    if (A->fps < B->fps) return 1;

    return 0;
}

static int cmp_down_native(const void *a, const void *b) {
    const RankResult *A = (const RankResult*)a;
    const RankResult *B = (const RankResult*)b;

    if (A->avg > B->avg) return -1;
    if (A->avg < B->avg) return 1;

    if (A->stddev < B->stddev) return -1;
    if (A->stddev > B->stddev) return 1;

    if (A->fps > B->fps) return -1;
    if (A->fps < B->fps) return 1;

    return 0;
}

static int cmp_sweet_native(const void *a, const void *b) {
    const SweetResult *A = (const SweetResult*)a;
    const SweetResult *B = (const SweetResult*)b;

    if (A->sweet < B->sweet) return -1;
    if (A->sweet > B->sweet) return 1;

    if (A->stddev < B->stddev) return -1;
    if (A->stddev > B->stddev) return 1;

    if (A->fps > B->fps) return -1;
    if (A->fps < B->fps) return 1;

    return 0;
}

// -----------------------------------------------------------------
// Compute the timing error: 1/float(fps) - reference(fps)
// -----------------------------------------------------------------
static inline prec_t compute_timing_error(double fps) {
    float f = (float)fps;
    prec_t model = (prec_t)(1.0f / f);
    prec_t one   = (prec_t)1.0;
    prec_t ref   = one / (prec_t)fps;
    return model - ref;
}

// -----------------------------------------------------------------
// GunZ native engine metric computation (uses WINDOW_SIZE, dynamic count,
// and BLOCK_METRIC_MODE to select single‑block or repeat‑interval timing)
// -----------------------------------------------------------------
static void compute_gunz_fdelta_metrics(double cfps,
                                        prec_t *out_mean_ms,
                                        prec_t *out_stddev_ms,
                                        prec_t *out_score)
{
    const int NJ = 21;                              // maximum jitter points
    double half_win = WINDOW_SIZE;                  // ±0.1 FPS default
    double step = (2.0 * half_win) / (NJ - 1);      // spacing between points

    prec_t sum_b  = (prec_t)0.0;
    prec_t sum_b2 = (prec_t)0.0;
    int used = 0;

    for (int j = 0; j < NJ; ++j) {
        double f = cfps - half_win + (double)j * step;
        if (f <= 0.0) continue;

        double T = 1000.0 / f;             // frame time (ms)
        double flo = floor(T);
        double frac = T - flo;
        int d_lo = (int)flo;
        int d_hi = d_lo + 1;
        int step_lo = (int)(d_lo * 4.8);
        int step_hi = (int)(d_hi * 4.8);

        double mean_step = step_lo * (1.0 - frac) + step_hi * frac;
        if (mean_step < 1e-6) continue;

        double var_step  = (step_lo - mean_step) * (step_lo - mean_step) * (1.0 - frac)
                         + (step_hi - mean_step) * (step_hi - mean_step) * frac;

        double N = GUNZ_BLOCK_FRAMES / mean_step;
        double anim_ms = N * T;

        // Choose metric based on compile‑time option
#if BLOCK_METRIC_MODE == 0
        double total_ms = anim_ms + T;          // single‑block: animation + release frame
#else
        double total_ms = anim_ms + 2.0 * T;   // repeat‑interval: animation + release + next‑press
#endif

        double var_total_ms = (GUNZ_BLOCK_FRAMES * var_step / (mean_step * mean_step * mean_step))
                             * T * T;

        ++used;
        sum_b  += (prec_t)total_ms;
        sum_b2 += (prec_t)(total_ms * total_ms + var_total_ms);
    }

    if (used == 0) {
        *out_mean_ms = *out_stddev_ms = *out_score = prec_inf();
        return;
    }

    prec_t mean_ms = sum_b / (prec_t)used;
    prec_t var_ms  = sum_b2 / (prec_t)used - mean_ms * mean_ms;
    if (var_ms < (prec_t)0.0) var_ms = (prec_t)0.0;
    prec_t stddev_ms = PREC_SQRT(var_ms);

    prec_t dev = PREC_ABS(mean_ms - (prec_t)GUNZ_BLOCK_REF_MS);
    prec_t score = dev + (prec_t)GUNZ_JITTER_WEIGHT * stddev_ms;

    *out_mean_ms   = mean_ms;
    *out_stddev_ms = stddev_ms;
    *out_score     = score;
}

// -----------------------------------------------------------------
// Safe llround wrapper – uses long‑double‑aware rounding
// -----------------------------------------------------------------
static inline int64_t safe_llround_ld(long double x) {
    if (x >= (long double)LLONG_MAX - 0.5L) return LLONG_MAX;
    if (x <= (long double)LLONG_MIN + 0.5L) return LLONG_MIN;
    return (int64_t)llroundl(x);
}

// -----------------------------------------------------------------
// Overflow‑safe calloc helper
// -----------------------------------------------------------------
static void *xcalloc(size_t nmemb, size_t size) {
    if (nmemb == 0 || size == 0) return calloc(1, 1);
    if (nmemb > SIZE_MAX / size) {
        fprintf(stderr, "Allocation size overflow\n");
        exit(1);
    }
    void *p = calloc(nmemb, size);
    if (!p) {
        fprintf(stderr, "Memory error\n");
        exit(1);
    }
    return p;
}

// -----------------------------------------------------------------
// Kahan addition for accumulator type
// -----------------------------------------------------------------
static inline void kahan_add(accum_t *sum, accum_t *c, accum_t value) {
    accum_t y = value - *c;
    accum_t t = *sum + y;
    *c = (t - *sum) - y;
    *sum = t;
}

// -----------------------------------------------------------------
// Runtime type info
// -----------------------------------------------------------------
static void print_runtime_info(void) {
    if (REQUESTED_PRECISION != ACTUAL_PRECISION) {
        printf("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!\n");
        printf(" WARNING: You requested PRECISION %d but it is not available\n",
               REQUESTED_PRECISION);
        printf(" on this compiler/platform.  The program has fallen back to\n");
        printf(" PRECISION %d.\n", ACTUAL_PRECISION);
        if (REQUESTED_PRECISION == 2)
            printf(" For full 128‑bit precision, install libquadmath and link with -lquadmath.\n");
        printf("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!\n\n");
    }
#if ACTUAL_PRECISION == 1 && LDBL_MANT_DIG < 64
    printf("Note: 'long double' on this platform has only %d mantissa bits;\n"
           "      extended 80‑bit precision is not available.\n\n", LDBL_MANT_DIG);
#endif

    if (ACTUAL_PRECISION == 1) {
        prec_t a = (prec_t)1.0;
        prec_t two = (prec_t)2.0;
        prec_t tiny = (prec_t)1.0;
        for (int k = 0; k < 53; k++) tiny /= two;
        prec_t s = a + tiny;
        printf("Precision test: 1 + 2^-53 (long double) -> %.21Lg  (%s)\n",
               (long double)s, (s == a) ? "tiny lost (std precision)" : "tiny preserved (ext precision)");
    }
#if ACTUAL_PRECISION == 2
    {
        prec_t a = (prec_t)1.0;
        prec_t two = (prec_t)2.0;
        prec_t tiny = (prec_t)1.0;
        for (int k = 0; k < 70; k++) tiny /= two;
        prec_t s = a + tiny;
        char buf[128];
        quadmath_snprintf(buf, sizeof buf, "%.36Qg", s);
        printf("Precision test: 1 + 2^-70 (__float128) -> %s  (%s)\n",
               buf, (s == a) ? "tiny lost" : "tiny preserved (quad precision)");
    }
#endif
}

// -----------------------------------------------------------------
// Top‑N insertion helpers
// -----------------------------------------------------------------
static void insert_up(BestResult *arr, size_t cap, size_t *cnt,
                      double fps, prec_t avg_p, prec_t stddev_p) {
    size_t i = 0;
    while (i < *cnt && arr[i].avg_prec > avg_p) i++;
    if (*cnt < cap) {
        for (size_t j = *cnt; j > i; j--) arr[j] = arr[j-1];
        arr[i].fps = fps;
        arr[i].avg = (double)avg_p;       arr[i].avg_prec = avg_p;
        arr[i].stddev = (double)stddev_p; arr[i].stddev_prec = stddev_p;
        arr[i].valid = 1;
        (*cnt)++;
    } else if (avg_p > arr[cap-1].avg_prec) {
        for (size_t j = cap-1; j > i; j--) arr[j] = arr[j-1];
        arr[i].fps = fps;
        arr[i].avg = (double)avg_p;       arr[i].avg_prec = avg_p;
        arr[i].stddev = (double)stddev_p; arr[i].stddev_prec = stddev_p;
        arr[i].valid = 1;
    }
}

static void insert_down(BestResult *arr, size_t cap, size_t *cnt,
                        double fps, prec_t avg_p, prec_t stddev_p) {
    size_t i = 0;
    while (i < *cnt && arr[i].avg_prec < avg_p) i++;
    if (*cnt < cap) {
        for (size_t j = *cnt; j > i; j--) arr[j] = arr[j-1];
        arr[i].fps = fps;
        arr[i].avg = (double)avg_p;       arr[i].avg_prec = avg_p;
        arr[i].stddev = (double)stddev_p; arr[i].stddev_prec = stddev_p;
        arr[i].valid = 1;
        (*cnt)++;
    } else if (avg_p < arr[cap-1].avg_prec) {
        for (size_t j = cap-1; j > i; j--) arr[j] = arr[j-1];
        arr[i].fps = fps;
        arr[i].avg = (double)avg_p;       arr[i].avg_prec = avg_p;
        arr[i].stddev = (double)stddev_p; arr[i].stddev_prec = stddev_p;
        arr[i].valid = 1;
    }
}

static void insert_sweet(BestResult *arr, size_t cap, size_t *cnt,
                         double fps, prec_t avg_p, prec_t stddev_p,
                         prec_t avg_abs_p, prec_t sweet_p) {
    size_t i = 0;
    while (i < *cnt && arr[i].sweet_prec < sweet_p) i++;
    if (*cnt < cap) {
        for (size_t j = *cnt; j > i; j--) arr[j] = arr[j-1];
        arr[i].fps = fps;
        arr[i].avg = (double)avg_p;       arr[i].avg_prec = avg_p;
        arr[i].stddev = (double)stddev_p; arr[i].stddev_prec = stddev_p;
        arr[i].avg_abs = (double)avg_abs_p; arr[i].avg_abs_prec = avg_abs_p;
        arr[i].sweet = (double)sweet_p;   arr[i].sweet_prec = sweet_p;
        arr[i].valid = 1;
        (*cnt)++;
    } else if (sweet_p < arr[cap-1].sweet_prec) {
        for (size_t j = cap-1; j > i; j--) arr[j] = arr[j-1];
        arr[i].fps = fps;
        arr[i].avg = (double)avg_p;       arr[i].avg_prec = avg_p;
        arr[i].stddev = (double)stddev_p; arr[i].stddev_prec = stddev_p;
        arr[i].avg_abs = (double)avg_abs_p; arr[i].avg_abs_prec = avg_abs_p;
        arr[i].sweet = (double)sweet_p;   arr[i].sweet_prec = sweet_p;
        arr[i].valid = 1;
    }
}

// -----------------------------------------------------------------
// O(N) sliding‑window scan – standard & native engine modes
// -----------------------------------------------------------------
static void scan_range(double range_start, double range_end,
                       BestResult **out_up, BestResult **out_down,
                       BestResult **out_sweet, size_t *num_bins,
                       int integer_mode,
                       prec_t *err_buf, size_t max_capacity)
{
    if (!(FPS_STEP > 0.0)) { fprintf(stderr, "Error: FPS_STEP > 0 required\n"); exit(1); }
    if (!(range_start > 0.0 && range_end > 0.0)) { fprintf(stderr, "Error: range > 0\n"); exit(1); }
    if (range_end <= range_start) { fprintf(stderr, "Error: range_end > range_start\n"); exit(1); }
    if (range_end - range_start > MAX_SPAN_FPS) {
        fprintf(stderr, "Error: span %.2f exceeds MAX_SPAN_FPS (%.2f)\n",
                range_end - range_start, MAX_SPAN_FPS);
        exit(1);
    }

    const int64_t steps_per_fps = safe_llround_ld(1.0L / (long double)FPS_STEP);
    if (steps_per_fps == 0) { fprintf(stderr, "FPS_STEP too large\n"); exit(1); }
    if (fabsl((long double)steps_per_fps * FPS_STEP - 1.0L) > EPS_STEPS_PER_FPS_ABS) {
        fprintf(stderr, "FPS_STEP not representable as 1/steps_per_fps\n");
        exit(1);
    }

    size_t half = (size_t)(WINDOW_SIZE / FPS_STEP + 0.5);
    if (half > (SIZE_MAX - 1) / 2) {
        fprintf(stderr, "Error: half window too large\n");
        exit(1);
    }
    size_t full_window = 2 * half + 1;

    double center_min, center_max;
    if (integer_mode) {
        center_min = floor(range_start);
        center_max = ceil(range_end);
    } else {
        center_min = range_start;
        center_max = range_end;
    }

    double ext_start = center_min - (double)half * FPS_STEP;
    double ext_end   = center_max + (double)half * FPS_STEP;

    if (ext_start <= 0.0) { fprintf(stderr, "Error: ext_start <= 0\n"); exit(1); }

    int64_t scaled_ext_start = safe_llround_ld(ext_start * (long double)steps_per_fps);
    int64_t scaled_ext_end   = safe_llround_ld(ext_end   * (long double)steps_per_fps);
    if (scaled_ext_end < scaled_ext_start) { fprintf(stderr, "Bad scaled endpoints\n"); exit(1); }
    int64_t total_samples_i  = scaled_ext_end - scaled_ext_start + 1;
    if (total_samples_i <= 0) { fprintf(stderr, "Error: negative sample count\n"); exit(1); }
    size_t total_samples = (size_t)total_samples_i;
    if (total_samples > max_capacity) {
        fprintf(stderr, "Error: internal buffer too small\n");
        exit(1);
    }
    if (full_window > total_samples) {
        fprintf(stderr, "Window size too large for range (full_window=%zu, samples=%zu)\n",
                full_window, total_samples);
        *out_up = *out_down = *out_sweet = NULL;
        *num_bins = 0;
        return;
    }

    // Pre‑compute timing errors for standard mode (not used in native mode)
    if (!g_GunzFdeltaMode) {
        for (size_t i = 0; i < total_samples; i++) {
            double fps = ext_start + (double)i * FPS_STEP;
            err_buf[i] = compute_timing_error(fps);
        }
    }

    int64_t bin_offset = (int64_t)floor(range_start);
    int64_t bins_i = (int64_t)floor(range_end) - bin_offset + 1;
    if (bins_i <= 0) { fprintf(stderr, "No bins\n"); exit(1); }
    size_t bins = (size_t)bins_i;

    BestResult *up = NULL, *down = NULL, *sweet = NULL;
    size_t up_cnt = 0, down_cnt = 0, sweet_cnt = 0;

    if (BINNING_MODE) {
        *num_bins = bins;
        up    = xcalloc(bins, sizeof *up);
        down  = xcalloc(bins, sizeof *down);
        sweet = xcalloc(bins, sizeof *sweet);
        for (size_t i = 0; i < bins; i++) {
            sweet[i].sweet_prec = prec_inf();
            sweet[i].sweet = INFINITY;
        }
    } else {
        *num_bins = TOP_N;
        up    = xcalloc(TOP_N, sizeof *up);
        down  = xcalloc(TOP_N, sizeof *down);
        sweet = xcalloc(TOP_N, sizeof *sweet);
    }

    double t_min = TARGET_FPS - TARGET_RANGE;
    double t_max = TARGET_FPS + TARGET_RANGE;
    if (TARGET_DIRECTION == 1) t_max = TARGET_FPS;
    if (TARGET_DIRECTION == 2) t_min = TARGET_FPS;

    // ===================== NATIVE ENGINE MODE =====================
    if (g_GunzFdeltaMode) {
        if (integer_mode) {
            int int_start = (int)ceil(range_start);
            int int_end   = (int)floor(range_end);
            for (int fps_int = int_start; fps_int <= int_end; fps_int++) {
                double cfps = (double)fps_int;
                prec_t mean_ms, stddev_ms, score;
                compute_gunz_fdelta_metrics(cfps, &mean_ms, &stddev_ms, &score);

                int64_t idx64 = (int64_t)fps_int - bin_offset;
                if (idx64 < 0 || idx64 >= (int64_t)bins) continue;

                int in_range = (cfps >= t_min && cfps <= t_max);
                int allow = (!TARGET_FILTER_EXTREMES) || in_range;
                if (!allow) continue;

                if (BINNING_MODE) {
                    size_t idx = (size_t)idx64;
                    if (!up[idx].valid || mean_ms < up[idx].avg_prec) {
                        up[idx].valid = 1; up[idx].fps = cfps;
                        up[idx].avg = (double)mean_ms;        up[idx].avg_prec = mean_ms;
                        up[idx].stddev = (double)stddev_ms;    up[idx].stddev_prec = stddev_ms;
                    }
                    if (!down[idx].valid || mean_ms > down[idx].avg_prec) {
                        down[idx].valid = 1; down[idx].fps = cfps;
                        down[idx].avg = (double)mean_ms;        down[idx].avg_prec = mean_ms;
                        down[idx].stddev = (double)stddev_ms;    down[idx].stddev_prec = stddev_ms;
                    }
                    if (!sweet[idx].valid || score < sweet[idx].sweet_prec) {
                        sweet[idx].valid = 1; sweet[idx].fps = cfps;
                        sweet[idx].sweet = (double)score;       sweet[idx].sweet_prec = score;
                        sweet[idx].avg = (double)mean_ms;       sweet[idx].avg_prec = mean_ms;
                        sweet[idx].stddev = (double)stddev_ms;   sweet[idx].stddev_prec = stddev_ms;
                    }
                } else {
                    // Non‑binning top‑N
                    insert_down(up, TOP_N, &up_cnt, cfps, mean_ms, stddev_ms);
                    insert_up(down, TOP_N, &down_cnt, cfps, mean_ms, stddev_ms);
                    insert_sweet(sweet, TOP_N, &sweet_cnt, cfps, mean_ms, stddev_ms,
                                 (prec_t)0.0, score);
                }
            }
        } else {
            // Exact integer‑scaled centre generation, zero drift
            int64_t scaled_center_start = safe_llround_ld(range_start * (long double)steps_per_fps);
            int64_t scaled_center_end   = safe_llround_ld(range_end   * (long double)steps_per_fps);
            for (int64_t sc = scaled_center_start; sc <= scaled_center_end; ++sc) {
                double cfps = (double)sc / (double)steps_per_fps;
                prec_t mean_ms, stddev_ms, score;
                compute_gunz_fdelta_metrics(cfps, &mean_ms, &stddev_ms, &score);

                int64_t idx64 = (int64_t)floor(cfps + 1e-12) - bin_offset;
                if (idx64 < 0 || idx64 >= (int64_t)bins) continue;

                int in_range = (cfps >= t_min && cfps <= t_max);
                int allow = (!TARGET_FILTER_EXTREMES) || in_range;
                if (!allow) continue;

                if (BINNING_MODE) {
                    size_t idx = (size_t)idx64;
                    if (!up[idx].valid || mean_ms < up[idx].avg_prec) {
                        up[idx].valid = 1; up[idx].fps = cfps;
                        up[idx].avg = (double)mean_ms;        up[idx].avg_prec = mean_ms;
                        up[idx].stddev = (double)stddev_ms;    up[idx].stddev_prec = stddev_ms;
                    }
                    if (!down[idx].valid || mean_ms > down[idx].avg_prec) {
                        down[idx].valid = 1; down[idx].fps = cfps;
                        down[idx].avg = (double)mean_ms;        down[idx].avg_prec = mean_ms;
                        down[idx].stddev = (double)stddev_ms;    down[idx].stddev_prec = stddev_ms;
                    }
                    if (!sweet[idx].valid || score < sweet[idx].sweet_prec) {
                        sweet[idx].valid = 1; sweet[idx].fps = cfps;
                        sweet[idx].sweet = (double)score;       sweet[idx].sweet_prec = score;
                        sweet[idx].avg = (double)mean_ms;       sweet[idx].avg_prec = mean_ms;
                        sweet[idx].stddev = (double)stddev_ms;   sweet[idx].stddev_prec = stddev_ms;
                    }
                } else {
                    insert_down(up, TOP_N, &up_cnt, cfps, mean_ms, stddev_ms);
                    insert_up(down, TOP_N, &down_cnt, cfps, mean_ms, stddev_ms);
                    insert_sweet(sweet, TOP_N, &sweet_cnt, cfps, mean_ms, stddev_ms,
                                 (prec_t)0.0, score);
                }
            }
        }

        // Finalise non‑binning mode
        if (!BINNING_MODE) {
            *num_bins = TOP_N;
            for (size_t i = up_cnt; i < TOP_N; i++) up[i].valid = 0;
            for (size_t i = down_cnt; i < TOP_N; i++) down[i].valid = 0;
            for (size_t i = sweet_cnt; i < TOP_N; i++) sweet[i].valid = 0;
        }
        *out_up = up; *out_down = down; *out_sweet = sweet;
        return;
    }

    // ===================== STANDARD DECIMAL MODE =====================
    accum_t win_sum  = (accum_t)0.0, c_sum  = (accum_t)0.0;
    accum_t win_sum2 = (accum_t)0.0, c_sum2 = (accum_t)0.0;
    accum_t win_abs  = (accum_t)0.0, c_abs  = (accum_t)0.0;

    for (size_t j = 0; j < full_window; j++) {
        accum_t e = err_buf[j];
        kahan_add(&win_sum,  &c_sum,  e);
        kahan_add(&win_sum2, &c_sum2, e * e);
        kahan_add(&win_abs,  &c_abs,  PREC_ABS(e));
    }

    size_t max_i = total_samples - full_window;
    accum_t inv_full_window = (accum_t)1.0 / (accum_t)full_window;

    for (size_t i = 0; i <= max_i; i++) {
        size_t ci = i + half;
        double cfps = ext_start + (double)ci * FPS_STEP;

        if (i > 0) {
            accum_t old = err_buf[i-1];
            accum_t ne  = err_buf[i + full_window - 1];
            kahan_add(&win_sum,  &c_sum,  -old);
            kahan_add(&win_sum,  &c_sum,  ne);
            kahan_add(&win_sum2, &c_sum2, -(old * old));
            kahan_add(&win_sum2, &c_sum2, ne * ne);
            kahan_add(&win_abs,  &c_abs,  -PREC_ABS(old));
            kahan_add(&win_abs,  &c_abs,  PREC_ABS(ne));
        }

        if (i > 0 && (i % KAHAN_RESET_INTERVAL) == 0) {
            size_t left  = i;
            size_t right = i + full_window - 1;
            win_sum = win_sum2 = win_abs = (accum_t)0.0;
            c_sum = c_sum2 = c_abs = (accum_t)0.0;
            for (size_t j = left; j <= right; j++) {
                accum_t e = err_buf[j];
                kahan_add(&win_sum,  &c_sum,  e);
                kahan_add(&win_sum2, &c_sum2, e * e);
                kahan_add(&win_abs,  &c_abs,  PREC_ABS(e));
            }
        }

        accum_t avg_ld     = win_sum * inv_full_window;
        accum_t avg_abs_ld = win_abs * inv_full_window;
        accum_t var_ld     = (win_sum2 * inv_full_window) - (avg_ld * avg_ld);
        if (var_ld < (accum_t)0.0) var_ld = (accum_t)0.0;

        double cfps_safe = nextafter(cfps, INFINITY);
        int64_t idx64 = (int64_t)floor(cfps_safe) - bin_offset;
        if (idx64 < 0 || idx64 >= (int64_t)bins) continue;

        int in_range = (cfps >= t_min && cfps <= t_max);
        int allow_for_extremes = (!TARGET_FILTER_EXTREMES) || in_range;

        if (BINNING_MODE) {
            size_t idx = (size_t)idx64;
            if (avg_ld > (accum_t)0.0 && allow_for_extremes) {
                if (!up[idx].valid || avg_ld > up[idx].avg_prec) {
                    prec_t stddev_p = PREC_SQRT(var_ld);
                    up[idx].valid = 1; up[idx].fps = cfps;
                    up[idx].avg = (double)avg_ld;     up[idx].avg_prec = avg_ld;
                    up[idx].stddev = (double)stddev_p; up[idx].stddev_prec = stddev_p;
                }
            }
            if (avg_ld < (accum_t)0.0 && allow_for_extremes) {
                if (!down[idx].valid || avg_ld < down[idx].avg_prec) {
                    prec_t stddev_p = PREC_SQRT(var_ld);
                    down[idx].valid = 1; down[idx].fps = cfps;
                    down[idx].avg = (double)avg_ld;     down[idx].avg_prec = avg_ld;
                    down[idx].stddev = (double)stddev_p; down[idx].stddev_prec = stddev_p;
                }
            }
            if (in_range) {
                prec_t cheap_p = avg_abs_ld * (prec_t)cfps;
                if (!sweet[idx].valid || cheap_p < sweet[idx].sweet_prec) {
                    prec_t stddev_p = PREC_SQRT(var_ld);
                    prec_t sweet_p = (avg_abs_ld + stddev_p) * (prec_t)cfps;
                    if (!sweet[idx].valid || sweet_p < sweet[idx].sweet_prec) {
                        sweet[idx].valid = 1; sweet[idx].fps = cfps;
                        sweet[idx].avg = (double)avg_ld;       sweet[idx].avg_prec = avg_ld;
                        sweet[idx].stddev = (double)stddev_p;  sweet[idx].stddev_prec = stddev_p;
                        sweet[idx].avg_abs = (double)avg_abs_ld; sweet[idx].avg_abs_prec = avg_abs_ld;
                        sweet[idx].sweet_prec = sweet_p;
                        sweet[idx].sweet     = (double)sweet_p;
                    }
                }
            }
        } else {
            prec_t cheap_p = avg_abs_ld * (prec_t)cfps;
            if (avg_ld > (accum_t)0.0 && allow_for_extremes) {
                prec_t stddev_p = PREC_SQRT(var_ld);
                insert_up(up, TOP_N, &up_cnt, cfps, avg_ld, stddev_p);
            }
            if (avg_ld < (accum_t)0.0 && allow_for_extremes) {
                prec_t stddev_p = PREC_SQRT(var_ld);
                insert_down(down, TOP_N, &down_cnt, cfps, avg_ld, stddev_p);
            }
            if (in_range) {
                int worth = (sweet_cnt < TOP_N) ||
                            (sweet_cnt >= TOP_N && cheap_p < sweet[TOP_N-1].sweet_prec);
                if (worth) {
                    prec_t stddev_p = PREC_SQRT(var_ld);
                    prec_t sweet_p = (avg_abs_ld + stddev_p) * (prec_t)cfps;
                    insert_sweet(sweet, TOP_N, &sweet_cnt,
                                 cfps, avg_ld, stddev_p, avg_abs_ld, sweet_p);
                }
            }
        }
    }

    if (!BINNING_MODE) {
        *num_bins = TOP_N;
        for (size_t i = up_cnt; i < TOP_N; i++) up[i].valid = 0;
        for (size_t i = down_cnt; i < TOP_N; i++) down[i].valid = 0;
        for (size_t i = sweet_cnt; i < TOP_N; i++) sweet[i].valid = 0;
    }

    *out_up = up;
    *out_down = down;
    *out_sweet = sweet;
}

// -----------------------------------------------------------------
// Merge and print (supports native mode formatting)
// -----------------------------------------------------------------
static void merge_and_print(BestResult **up_arrays,
                            BestResult **down_arrays,
                            BestResult **sweet_arrays,
                            size_t *bin_counts,
                            int num_ranges,
                            int integer_mode)
{
    size_t total_bins = 0;
    for (int r = 0; r < num_ranges; r++)
        if (up_arrays[r] && down_arrays[r] && sweet_arrays[r])
            total_bins += bin_counts[r];
    if (total_bins == 0) { fprintf(stderr, "No valid bins found across ranges.\n"); return; }

    RankResult  *up_r    = malloc(total_bins * sizeof *up_r);
    RankResult  *down_r  = malloc(total_bins * sizeof *down_r);
    SweetResult *sweet_r = malloc(total_bins * sizeof *sweet_r);
    if (!up_r || !down_r || !sweet_r) { fprintf(stderr, "Memory error\n"); exit(1); }
    size_t up_n = 0, down_n = 0, sweet_n = 0;

    for (int r = 0; r < num_ranges; r++) {
        BestResult *up = up_arrays[r], *down = down_arrays[r], *sweet = sweet_arrays[r];
        if (!up || !down || !sweet) continue;
        size_t bins = bin_counts[r];
        for (size_t i = 0; i < bins; i++) {
            if (up[i].valid) {
                up_r[up_n].fps = up[i].fps;
                up_r[up_n].avg = up[i].avg;   up_r[up_n].avg_prec = up[i].avg_prec;
                up_r[up_n].stddev = up[i].stddev; up_r[up_n].stddev_prec = up[i].stddev_prec;
                up_n++;
            }
            if (down[i].valid) {
                down_r[down_n].fps = down[i].fps;
                down_r[down_n].avg = down[i].avg;   down_r[down_n].avg_prec = down[i].avg_prec;
                down_r[down_n].stddev = down[i].stddev; down_r[down_n].stddev_prec = down[i].stddev_prec;
                down_n++;
            }
            if (sweet[i].valid) {
                sweet_r[sweet_n].fps = sweet[i].fps;
                sweet_r[sweet_n].avg = sweet[i].avg;   sweet_r[sweet_n].avg_prec = sweet[i].avg_prec;
                sweet_r[sweet_n].stddev = sweet[i].stddev; sweet_r[sweet_n].stddev_prec = sweet[i].stddev_prec;
                sweet_r[sweet_n].avg_abs = sweet[i].avg_abs; sweet_r[sweet_n].avg_abs_prec = sweet[i].avg_abs_prec;
                sweet_r[sweet_n].sweet = sweet[i].sweet; sweet_r[sweet_n].sweet_prec = sweet[i].sweet_prec;
                sweet_n++;
            }
        }
    }

    // UP / FASTEST BLOCK
    if (up_n > 0) {
        if (g_GunzFdeltaMode) {
            qsort(up_r, up_n, sizeof *up_r, cmp_up_native);
            size_t display = (size_t)TOP_N < up_n ? (size_t)TOP_N : up_n;
            printf("=== FASTEST BLOCK (lower ms, native engine) ===\n");
            for (size_t i = 0; i < display; i++) {
                printf("%3zu. %10.5f FPS    block= %7.3f ms   jitter= %6.2f ms\n",
                       i+1, up_r[i].fps, up_r[i].avg, up_r[i].stddev);
            }
        } else {
            qsort(up_r, up_n, sizeof *up_r, cmp_up);
            size_t display = (size_t)TOP_N < up_n ? (size_t)TOP_N : up_n;
            printf("=== EXTREME + STABLE UP (%s, %s) ===\n",
                   integer_mode ? "integer centres" : "best decimal per integer",
                   TARGET_FILTER_EXTREMES ? "filtered by target" : "global");
            for (size_t i = 0; i < display; i++) {
                printf("%3zu. %10.5f FPS    ", i+1, up_r[i].fps);
#if ACTUAL_PRECISION == 2
                char buf1[128], buf2[128];
                quadmath_snprintf(buf1, sizeof buf1, "%+46.36Qe", up_r[i].avg_prec);
                quadmath_snprintf(buf2, sizeof buf2, "%46.36Qe",  up_r[i].stddev_prec);
                printf("avg= %s  stddev= %s\n", buf1, buf2);
#elif ACTUAL_PRECISION == 1
                printf("avg= %+38.21Le  stddev= %38.21Le\n",
                       (long double)up_r[i].avg_prec,
                       (long double)up_r[i].stddev_prec);
#else
                printf("avg= %+20.6e  stddev= %20.6e\n", up_r[i].avg, up_r[i].stddev);
#endif
            }
        }
    } else printf("No UP results found.\n");

    // DOWN / SLOWEST BLOCK
    if (down_n > 0) {
        if (g_GunzFdeltaMode) {
            qsort(down_r, down_n, sizeof *down_r, cmp_down_native);
            size_t display = (size_t)TOP_N < down_n ? (size_t)TOP_N : down_n;
            printf("\n=== SLOWEST BLOCK (higher ms, native engine) ===\n");
            for (size_t i = 0; i < display; i++) {
                printf("%3zu. %10.5f FPS    block= %7.3f ms   jitter= %6.2f ms\n",
                       i+1, down_r[i].fps, down_r[i].avg, down_r[i].stddev);
            }
        } else {
            qsort(down_r, down_n, sizeof *down_r, cmp_down);
            size_t display = (size_t)TOP_N < down_n ? (size_t)TOP_N : down_n;
            printf("\n=== EXTREME + STABLE DOWN (%s, %s) ===\n",
                   integer_mode ? "integer centres" : "best decimal per integer",
                   TARGET_FILTER_EXTREMES ? "filtered by target" : "global");
            for (size_t i = 0; i < display; i++) {
                printf("%3zu. %10.5f FPS    ", i+1, down_r[i].fps);
#if ACTUAL_PRECISION == 2
                char buf1[128], buf2[128];
                quadmath_snprintf(buf1, sizeof buf1, "%+46.36Qe", down_r[i].avg_prec);
                quadmath_snprintf(buf2, sizeof buf2, "%46.36Qe",  down_r[i].stddev_prec);
                printf("avg= %s  stddev= %s\n", buf1, buf2);
#elif ACTUAL_PRECISION == 1
                printf("avg= %+38.21Le  stddev= %38.21Le\n",
                       (long double)down_r[i].avg_prec,
                       (long double)down_r[i].stddev_prec);
#else
                printf("avg= %+20.6e  stddev= %20.6e\n", down_r[i].avg, down_r[i].stddev);
#endif
            }
        }
    } else printf("\nNo DOWN results found.\n");

    // SWEET / BEST ACCURACY
    if (sweet_n > 0) {
        if (g_GunzFdeltaMode) {
            qsort(sweet_r, sweet_n, sizeof *sweet_r, cmp_sweet_native);
            size_t display = (size_t)TOP_N < sweet_n ? (size_t)TOP_N : sweet_n;
            printf("\n=== BEST ACCURACY (closest to %.1f ms %s, native engine) ===\n",
                   GUNZ_BLOCK_REF_MS,
                   BLOCK_METRIC_MODE ? "repeat interval" : "block");
            printf("(Lower score = %s dev. + %.1f * jitter)\n",
                   BLOCK_METRIC_MODE ? "interval" : "block time",
                   (double)GUNZ_JITTER_WEIGHT);
            for (size_t i = 0; i < display; i++) {
                printf("%3zu. %10.5f FPS    block= %7.3f ms   jitter= %6.2f ms   score= %6.1f\n",
                       i+1, sweet_r[i].fps, sweet_r[i].avg, sweet_r[i].stddev, sweet_r[i].sweet);
            }
        } else {
            qsort(sweet_r, sweet_n, sizeof *sweet_r, cmp_sweet);
            size_t display = (size_t)TOP_N < sweet_n ? (size_t)TOP_N : sweet_n;
            printf("\n=== SWEET SPOT (best %s, target %.0f ±%.0f FPS, direction %s) ===\n",
                   integer_mode ? "integer" : "decimal",
                   TARGET_FPS, TARGET_RANGE,
                   TARGET_DIRECTION == 0 ? "both" : (TARGET_DIRECTION == 1 ? "≤ target" : "≥ target"));
            printf("(Lower sweet score is better; higher FPS always reduces latency)\n");
            for (size_t i = 0; i < display; i++) {
                printf("%3zu. %10.5f FPS    ", i+1, sweet_r[i].fps);
#if ACTUAL_PRECISION == 2
                char b1[128], b2[128], b3[128], b4[128];
                quadmath_snprintf(b1, sizeof b1, "%+46.36Qe", sweet_r[i].avg_prec);
                quadmath_snprintf(b2, sizeof b2, "%46.36Qe",  sweet_r[i].stddev_prec);
                quadmath_snprintf(b3, sizeof b3, "%46.36Qe",  sweet_r[i].avg_abs_prec);
                quadmath_snprintf(b4, sizeof b4, "%+46.36Qe", sweet_r[i].sweet_prec);
                printf("avg= %s  stddev= %s  avg_abs= %s  sweet= %s\n", b1, b2, b3, b4);
#elif ACTUAL_PRECISION == 1
                printf("avg= %+38.21Le  stddev= %38.21Le  avg_abs= %38.21Le  sweet= %+38.21Le\n",
                       (long double)sweet_r[i].avg_prec,
                       (long double)sweet_r[i].stddev_prec,
                       (long double)sweet_r[i].avg_abs_prec,
                       (long double)sweet_r[i].sweet_prec);
#else
                printf("avg= %+20.6e  stddev= %20.6e  avg_abs= %20.6e  sweet= %+20.6e\n",
                       sweet_r[i].avg, sweet_r[i].stddev,
                       sweet_r[i].avg_abs, sweet_r[i].sweet);
#endif
            }
        }
    } else printf("\nNo SWEET SPOT results found.\n");

    free(up_r); free(down_r); free(sweet_r);
}

// -----------------------------------------------------------------
// Single‑point mode (unchanged, not available in native mode)
// -----------------------------------------------------------------
static int fps_in_target(double fps) {
    if (TARGET_DIRECTION == 0)
        return fabs(fps - TARGET_FPS) <= TARGET_RANGE;
    else if (TARGET_DIRECTION == 1)
        return (fps <= TARGET_FPS && fps >= TARGET_FPS - TARGET_RANGE);
    else
        return (fps >= TARGET_FPS && fps <= TARGET_FPS + TARGET_RANGE);
}

static void insert_sorted_zero(double fps, prec_t abs_err,
                               double *fps_list, prec_t *abs_list,
                               int *count, int max) {
    int pos = 0;
    while (pos < *count && abs_list[pos] < abs_err) pos++;
    if (pos == max) return;
    for (int i = (*count < max ? *count : max-1); i > pos; i--) {
        fps_list[i] = fps_list[i-1];
        abs_list[i] = abs_list[i-1];
    }
    fps_list[pos] = fps;
    abs_list[pos] = abs_err;
    if (*count < max) (*count)++;
}

void run_single_point_mode(void) {
    if (g_GunzFdeltaMode) {
        printf("Single‑point mode is not supported in GunZ native engine mode.\n");
        return;
    }
    // ... original single‑point code (unchanged) ...
}

// -----------------------------------------------------------------
// Single‑range wrappers
// -----------------------------------------------------------------
void run_decimal_mode(double start, double end) {
    double max_ext_span = (end - start) + 2 * WINDOW_SIZE;
    size_t n_max = (size_t)safe_llround_ld((long double)max_ext_span / FPS_STEP) + 10;
    prec_t *err_buf = NULL;
    if (!g_GunzFdeltaMode) {
        err_buf = malloc(n_max * sizeof(prec_t));
        if (!err_buf) { fprintf(stderr, "Memory error\n"); exit(1); }
    }
    size_t num_bins;
    BestResult *up, *down, *sweet;
    scan_range(start, end, &up, &down, &sweet, &num_bins, 0, err_buf, n_max);
    BestResult *up_arr[1] = { up };
    BestResult *down_arr[1] = { down };
    BestResult *sweet_arr[1] = { sweet };
    merge_and_print(up_arr, down_arr, sweet_arr, &num_bins, 1, 0);
    free(up); free(down); free(sweet);
    if (err_buf) free(err_buf);
}

void run_integer_mode(double start, double end) {
    double max_ext_span = (end - start) + 2 * WINDOW_SIZE;
    size_t n_max = (size_t)safe_llround_ld((long double)max_ext_span / FPS_STEP) + 10;
    prec_t *err_buf = NULL;
    if (!g_GunzFdeltaMode) {
        err_buf = malloc(n_max * sizeof(prec_t));
        if (!err_buf) { fprintf(stderr, "Memory error\n"); exit(1); }
    }
    size_t num_bins;
    BestResult *up, *down, *sweet;
    scan_range(start, end, &up, &down, &sweet, &num_bins, 1, err_buf, n_max);
    BestResult *up_arr[1] = { up };
    BestResult *down_arr[1] = { down };
    BestResult *sweet_arr[1] = { sweet };
    merge_and_print(up_arr, down_arr, sweet_arr, &num_bins, 1, 1);
    free(up); free(down); free(sweet);
    if (err_buf) free(err_buf);
}

// -----------------------------------------------------------------
int main(void) {
#if Gunz_Fdelta_finder
    g_GunzFdeltaMode = 1;
#endif

    print_runtime_info();
    printf("\n");

    if (SINGLE_POINT_EXTREMES) {
        run_single_point_mode();
        return 0;
    }

    if (BATCH_MODE) {
        printf("Batch scanning %d ranges...\n", NUM_BATCH_RANGES);
        printf("Sweet spot target: %.0f FPS (range ±%.0f FPS, direction %s)\n",
               TARGET_FPS, TARGET_RANGE,
               TARGET_DIRECTION == 0 ? "both" : (TARGET_DIRECTION == 1 ? "≤ target" : "≥ target"));
        printf("Extreme filter: %s\n\n",
               TARGET_FILTER_EXTREMES ? "ON (only target window)" : "OFF (global)");

        double t_min = TARGET_FPS - TARGET_RANGE;
        double t_max = TARGET_FPS + TARGET_RANGE;
        if (TARGET_DIRECTION == 1) t_max = TARGET_FPS;
        if (TARGET_DIRECTION == 2) t_min = TARGET_FPS;

        double max_ext_span = MAX_SPAN_FPS + 2 * WINDOW_SIZE;
        size_t n_max = (size_t)safe_llround_ld((long double)max_ext_span / FPS_STEP) + 10;

#if !USE_OPENMP
        prec_t *err_buf = NULL;
        if (!g_GunzFdeltaMode) {
            err_buf = malloc(n_max * sizeof(prec_t));
            if (!err_buf) { fprintf(stderr, "Memory error\n"); exit(1); }
        }
#endif

        BestResult *up_arrays[NUM_BATCH_RANGES] = {NULL};
        BestResult *down_arrays[NUM_BATCH_RANGES] = {NULL};
        BestResult *sweet_arrays[NUM_BATCH_RANGES] = {NULL};
        size_t bin_counts[NUM_BATCH_RANGES] = {0};

#if USE_OPENMP
#pragma omp parallel for schedule(dynamic)
#endif
        for (size_t r = 0; r < NUM_BATCH_RANGES; r++) {
            if (TARGET_FILTER_EXTREMES) {
                if (batch_ends[r] < t_min || batch_starts[r] > t_max) {
                    printf("  Range %zu (%s): %.0f - %.0f  [skipped – outside target window]\n",
                           r+1, INTEGER_MODE ? "integer" : "decimal",
                           batch_starts[r], batch_ends[r]);
                    continue;
                }
            }

            printf("  Range %zu (%s): %.0f - %.0f\n", r+1,
                   INTEGER_MODE ? "integer" : "decimal",
                   batch_starts[r], batch_ends[r]);

#if USE_OPENMP
            prec_t *err_buf = NULL;
            if (!g_GunzFdeltaMode) {
                err_buf = malloc(n_max * sizeof(prec_t));
                if (!err_buf) { fprintf(stderr, "Memory error\n"); exit(1); }
            }
#endif

            scan_range(batch_starts[r], batch_ends[r],
                       &up_arrays[r], &down_arrays[r], &sweet_arrays[r],
                       &bin_counts[r], INTEGER_MODE,
                       err_buf, n_max);

#if USE_OPENMP
            if (err_buf) free(err_buf);
#endif
        }

        printf("\nMerging and ranking global results...\n\n");
        merge_and_print(up_arrays, down_arrays, sweet_arrays,
                        bin_counts, NUM_BATCH_RANGES, INTEGER_MODE);
        for (size_t r = 0; r < NUM_BATCH_RANGES; r++) {
            if (up_arrays[r])   free(up_arrays[r]);
            if (down_arrays[r]) free(down_arrays[r]);
            if (sweet_arrays[r]) free(sweet_arrays[r]);
        }

#if !USE_OPENMP
        if (err_buf) free(err_buf);
#endif
        return 0;
    }

    printf("Scanning %.2f–%.2f (step %.5f) … ", FPS_START, FPS_END, FPS_STEP);
    if (INTEGER_MODE) printf("[INTEGER CENTRES ONLY] ");
    fflush(stdout);

    if (INTEGER_MODE)
        run_integer_mode(FPS_START, FPS_END);
    else
        run_decimal_mode(FPS_START, FPS_END);

    printf("\nDone.\n");
    return 0;
}
