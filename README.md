//   WTFF — Windowed Timing Frame Finder 0.3.0
//   By Grimnismal & a bit of AI, vibe coded.
/ This tool measures IEEE‑754 rounding error in 32‑bit float vs. 64‑bit double
// and how that error affects FPS timing.
// It has also a native mode for gunz the duel which has broken timers.
/// Plain version: FPS is imprecise and fluctuates; error leans one way or the
/// other — unless you pick a good value.
// Here are the good values.


 What It Actually Finds

GunZ runs on the RealSpace2 engine, which uses timeGetTime() for all timing. That timer has 1 ms resolution, so the game never sees fractional milliseconds.
The engine’s animation speed is driven by a constant m_fSpeed = 4.8 – every frame, the animation counter advances by floor( delta_ms × 4.8 ).
This simple rule creates a hidden, FPS‑dependent landscape where frame time, animation speed, and input responsiveness all interact in non‑obvious ways.

The WTFF tool simulates this exact behaviour, including truncation, overshoot, and the ±0.1 FPS jitter that every real limiter produces.
Its purpose is to find FPS values that give you maximum stability, maximum speed (or slowness), and the smoothest visual/animation experience.
What the Engine Actually Does

    At any given FPS, the target frame time is T = 1000 / fps ms.

    Because the timer only returns integers, the actual per‑frame deltas alternate between floor(T) and ceil(T) in a pattern that matches T on average.

    Each frame, the animation advances by floor( delta × 4.8 ) ticks (internal animation units).

    This means the per‑frame advance can be two very different values if T is not an integer, causing the animation to stutter microscopically.

    A block (or any animation) ends when m_nFrame ≥ 1140. The exact finish time depends on the whole sequence of integer deltas, including any overshoot on the last frame.

The tool simulates all of that – including the ±0.1 FPS jitter window – and calculates the real block completion time for each candidate FPS.
Two Kinds of Jitter – and Why You Must Avoid Them

Block‑time jitter
The standard deviation of the total block time across the 21 jitter samples.
If your FPS limiter fluctuates by ±0.1 FPS, this tells you how much the action’s total duration will vary.
High block‑time jitter means your block (or dash, or slash) will sometimes end a few ms early, sometimes late – killing your timing consistency.

Motion jitter
The standard deviation of the per‑frame animation advance (in ticks) during the block.
This is what makes animations feel “choppy” or “jittery” – the character appears to speed up and slow down randomly within a single move.
High motion jitter means your own animations and those of opponents visibly stutter, and it can even affect muscle memory because the visual feedback is irregular.

The BEST ACCURACY list in the tool combines both jitter types with a penalty for deviation from the theoretical reference (237.5 ms).
A score of 10.5 with zero jitter means you’re only 10 ms off the ideal, with absolutely perfect stability and smoothness – that’s the real sweet spot.
The Real‑World Impact: 7 ms Matters

In the playable range (~190–390 FPS), the difference between the fastest and slowest block is roughly 7 ms.
That may sound tiny, but it applies to every animation in the game – dash, slash, massive, reload, everything.

    A faster dash moves you a shorter distance; a slower dash moves you further.

    Fast block recovery means you can act sooner; slow block means longer invincibility.

    Even a 5 ms shift changes the feel of combos and the frame‑perfect windows for cancels.

The tool lets you deliberately pick FPS values that speed up or slow down specific actions to match your playstyle.
Reading the Numbers

FASTEST BLOCK / SLOWEST BLOCK
These are the extremes of total block time. Use them to gain speed or to extend animation duration for tactical advantage.

BEST ACCURACY
This is the FPS that balances minimal block‑time jitter, minimal motion jitter, and closeness to the theoretical reference.
At 250 FPS (or nearby integer), motion jitter is zero – every frame advances the animation by exactly 19 ticks. That’s why the community has always felt 250 was “the best”. The tool confirms it mathematically.

HIGHEST JITTER / HIGHEST MOTION JITTER
These show the most unstable FPS values – avoid them.
Decimal Values and Rounding

You will see many entries like 209.99999 or 210.42631.

    Values ending in .99999 or .00001 can safely be rounded to the nearest integer – the integer’s performance is virtually identical (usually less than 0.02 ms difference).

    However, some decimal values like 210.42631 are genuinely faster than the nearest integer, due to the precise alignment of the frame‑time pattern with the input hold and the game’s logic.
    If you use RTSS, you can cap at these exact decimals and gain that last fraction of a millisecond.
    For NVIDIA/AMD limiters (which only accept integers), round to the nearest integer – you’ll still be within a hair’s width of optimal.

Sweet spot target: 256 FPS (range ±64 FPS, direction both)

=== FASTEST BLOCK (lower ms, native engine) ===
  1.  210.42631 FPS    block= 247.505 ms   jitter=   0.00 ms
  2.  209.99999 FPS    block= 247.524 ms   jitter=   0.00 ms
  3.  230.50796 FPS    block= 247.676 ms   jitter=   0.00 ms
  4.  229.99999 FPS    block= 247.696 ms   jitter=   0.00 ms
  5.  206.21578 FPS    block= 247.699 ms   jitter=   0.00 ms
  6.  205.99999 FPS    block= 247.709 ms   jitter=   0.00 ms
  7.  226.31509 FPS    block= 247.837 ms   jitter=   0.00 ms
  8.  225.99999 FPS    block= 247.850 ms   jitter=   0.00 ms
  9.  202.00526 FPS    block= 247.901 ms   jitter=   0.00 ms
 10.  201.99999 FPS    block= 247.901 ms   jitter=   0.00 ms
 11.  250.42192 FPS    block= 247.987 ms   jitter=   0.00 ms
 12.  249.99999 FPS    block= 248.000 ms   jitter=   0.00 ms
 13.  222.12222 FPS    block= 248.004 ms   jitter=   0.00 ms
 14.  221.99999 FPS    block= 248.009 ms   jitter=   0.00 ms
 15.  197.79473 FPS    block= 248.111 ms   jitter=   0.00 ms
 16.  246.24655 FPS    block= 248.122 ms   jitter=   0.00 ms
 17.  245.99999 FPS    block= 248.130 ms   jitter=   0.00 ms
 18.  217.92935 FPS    block= 248.177 ms   jitter=   0.00 ms
 19.  242.07118 FPS    block= 248.262 ms   jitter=   0.00 ms
 20.  241.99999 FPS    block= 248.264 ms   jitter=   0.00 ms
 21.  213.73647 FPS    block= 248.357 ms   jitter=   0.00 ms
 22.  270.17027 FPS    block= 248.403 ms   jitter=   0.00 ms
 23.  237.89582 FPS    block= 248.407 ms   jitter=   0.00 ms
 24.  269.99999 FPS    block= 248.407 ms   jitter=   0.00 ms
 25.  266.01226 FPS    block= 248.518 ms   jitter=   0.00 ms
 26.  265.99999 FPS    block= 248.519 ms   jitter=   0.00 ms
 27.  233.72045 FPS    block= 248.557 ms   jitter=   0.00 ms
 28.  208.99999 FPS    block= 248.569 ms   jitter=   0.00 ms
 29.  232.99999 FPS    block= 248.584 ms   jitter=   0.00 ms
 30.  212.99999 FPS    block= 248.628 ms   jitter=   0.43 ms
 31.  261.85426 FPS    block= 248.638 ms   jitter=   0.00 ms
 32.  260.99999 FPS    block= 248.663 ms   jitter=   0.00 ms
 33.  228.99999 FPS    block= 248.734 ms   jitter=   0.00 ms
 34.  204.99999 FPS    block= 248.756 ms   jitter=   0.00 ms
 35.  257.69625 FPS    block= 248.761 ms   jitter=   0.00 ms
 36.  256.99999 FPS    block= 248.782 ms   jitter=   0.00 ms
 37.  253.53825 FPS    block= 248.888 ms   jitter=   0.00 ms
 38.  224.99999 FPS    block= 248.889 ms   jitter=   0.00 ms
 39.  289.75507 FPS    block= 248.902 ms   jitter=   0.00 ms
 40.  252.99999 FPS    block= 248.905 ms   jitter=   0.00 ms
 41.  288.99999 FPS    block= 248.920 ms   jitter=   0.00 ms
 42.  200.99999 FPS    block= 248.950 ms   jitter=   0.00 ms
 43.  196.99999 FPS    block= 248.962 ms   jitter=   0.39 ms
 44.  236.99999 FPS    block= 248.963 ms   jitter=   0.50 ms
 45.  285.61428 FPS    block= 249.002 ms   jitter=   0.00 ms
 46.  284.99999 FPS    block= 249.018 ms   jitter=   0.00 ms
 47.  248.99999 FPS    block= 249.032 ms   jitter=   0.00 ms
 48.  220.99999 FPS    block= 249.050 ms   jitter=   0.00 ms
 49.  281.47349 FPS    block= 249.105 ms   jitter=   0.00 ms
 50.  280.99999 FPS    block= 249.117 ms   jitter=   0.00 ms
 51.  264.99999 FPS    block= 249.119 ms   jitter=   0.50 ms
 52.  244.99999 FPS    block= 249.163 ms   jitter=   0.00 ms
 53.  277.33271 FPS    block= 249.212 ms   jitter=   0.00 ms
 54.  216.99999 FPS    block= 249.217 ms   jitter=   0.00 ms
 55.  276.99999 FPS    block= 249.220 ms   jitter=   0.00 ms
 56.  240.99999 FPS    block= 249.299 ms   jitter=   0.00 ms
 57.  273.19192 FPS    block= 249.321 ms   jitter=   0.00 ms
 58.  272.99999 FPS    block= 249.326 ms   jitter=   0.00 ms
 59.  192.77211 FPS    block= 249.375 ms   jitter=   0.00 ms
 60.  268.99999 FPS    block= 249.435 ms   jitter=   0.00 ms
 61.  309.17835 FPS    block= 249.469 ms   jitter=   0.00 ms
 62.  308.99999 FPS    block= 249.472 ms   jitter=   0.00 ms
 63.  305.05463 FPS    block= 249.556 ms   jitter=   0.00 ms
 64.  304.99999 FPS    block= 249.557 ms   jitter=   0.00 ms

=== SLOWEST BLOCK (higher ms, native engine) ===
  1.  192.97212 FPS    block= 254.078 ms   jitter=   0.45 ms
  2.  193.00000 FPS    block= 253.934 ms   jitter=   0.50 ms
  3.  312.21499 FPS    block= 253.406 ms   jitter=   0.00 ms
  4.  316.32177 FPS    block= 253.323 ms   jitter=   0.00 ms
  5.  292.88351 FPS    block= 252.829 ms   jitter=   0.00 ms
  6.  293.00000 FPS    block= 252.826 ms   jitter=   0.00 ms
  7.  297.00722 FPS    block= 252.734 ms   jitter=   0.00 ms
  8.  301.13093 FPS    block= 252.642 ms   jitter=   0.00 ms
  9.  296.99722 FPS    block= 252.591 ms   jitter=   0.64 ms
 10.  305.25464 FPS    block= 252.552 ms   jitter=   0.00 ms
 11.  233.92046 FPS    block= 252.550 ms   jitter=   0.00 ms
 12.  309.37836 FPS    block= 252.465 ms   jitter=   0.00 ms
 13.  234.00000 FPS    block= 252.452 ms   jitter=   0.29 ms
 14.  313.00000 FPS    block= 252.390 ms   jitter=   0.00 ms
 15.  238.09583 FPS    block= 252.352 ms   jitter=   0.21 ms
 16.  213.93648 FPS    block= 252.349 ms   jitter=   0.00 ms
 17.  214.00000 FPS    block= 252.346 ms   jitter=   0.00 ms
 18.  197.99474 FPS    block= 252.339 ms   jitter=   0.43 ms
 19.  273.39193 FPS    block= 252.316 ms   jitter=   0.00 ms
 20.  194.00000 FPS    block= 252.309 ms   jitter=   0.00 ms
 21.  317.00000 FPS    block= 252.309 ms   jitter=   0.00 ms
 22.  198.00000 FPS    block= 252.291 ms   jitter=   0.39 ms
 23.  277.53272 FPS    block= 252.206 ms   jitter=   0.00 ms
 24.  278.00000 FPS    block= 252.194 ms   jitter=   0.00 ms
 25.  218.12936 FPS    block= 252.169 ms   jitter=   0.00 ms
 26.  281.67350 FPS    block= 252.100 ms   jitter=   0.00 ms
 27.  282.00000 FPS    block= 252.092 ms   jitter=   0.00 ms
 28.  285.81429 FPS    block= 251.998 ms   jitter=   0.00 ms
 29.  222.32223 FPS    block= 251.996 ms   jitter=   0.00 ms
 30.  286.00000 FPS    block= 251.993 ms   jitter=   0.00 ms
 31.  274.00000 FPS    block= 251.918 ms   jitter=   0.49 ms
 32.  289.95508 FPS    block= 251.898 ms   jitter=   0.00 ms
 33.  290.00000 FPS    block= 251.897 ms   jitter=   0.00 ms
 34.  202.20527 FPS    block= 251.891 ms   jitter=   0.00 ms
 35.  253.73826 FPS    block= 251.882 ms   jitter=   0.00 ms
 36.  242.27119 FPS    block= 251.874 ms   jitter=   0.49 ms
 37.  254.00000 FPS    block= 251.874 ms   jitter=   0.00 ms
 38.  226.51510 FPS    block= 251.829 ms   jitter=   0.00 ms
 39.  294.00000 FPS    block= 251.803 ms   jitter=   0.00 ms
 40.  257.89626 FPS    block= 251.755 ms   jitter=   0.00 ms
 41.  258.00000 FPS    block= 251.752 ms   jitter=   0.00 ms
 42.  298.00000 FPS    block= 251.711 ms   jitter=   0.00 ms
 43.  206.41579 FPS    block= 251.689 ms   jitter=   0.00 ms
 44.  230.70797 FPS    block= 251.669 ms   jitter=   0.00 ms
 45.  262.05427 FPS    block= 251.632 ms   jitter=   0.00 ms
 46.  302.00000 FPS    block= 251.623 ms   jitter=   0.00 ms
 47.  306.00000 FPS    block= 251.536 ms   jitter=   0.00 ms
 48.  266.21227 FPS    block= 251.513 ms   jitter=   0.00 ms
 49.  210.62632 FPS    block= 251.495 ms   jitter=   0.00 ms
 50.  211.00000 FPS    block= 251.479 ms   jitter=   0.00 ms
 51.  310.00000 FPS    block= 251.452 ms   jitter=   0.00 ms
 52.  246.44656 FPS    block= 251.449 ms   jitter=   0.47 ms
 53.  270.37028 FPS    block= 251.397 ms   jitter=   0.00 ms
 54.  314.00000 FPS    block= 251.369 ms   jitter=   0.00 ms
 55.  239.00000 FPS    block= 251.368 ms   jitter=   0.00 ms
 56.  207.00000 FPS    block= 251.329 ms   jitter=   0.47 ms
 57.  215.00000 FPS    block= 251.302 ms   jitter=   0.00 ms
 58.  318.00000 FPS    block= 251.289 ms   jitter=   0.00 ms
 59.  275.00000 FPS    block= 251.273 ms   jitter=   0.00 ms
 60.  235.00000 FPS    block= 251.273 ms   jitter=   0.43 ms
 61.  195.00000 FPS    block= 251.256 ms   jitter=   0.00 ms
 62.  243.00000 FPS    block= 251.230 ms   jitter=   0.00 ms
 63.  279.00000 FPS    block= 251.168 ms   jitter=   0.00 ms
 64.  219.00000 FPS    block= 251.132 ms   jitter=   0.00 ms

=== BEST ACCURACY (closest to 237.5 ms repeat interval, native engine) ===
(Lower score = interval dev. + 4.0 * block_jitter + 1.0 * motion_jitter)
  1.  250.42192 FPS    block= 247.987 ms   jitter=   0.00 ms   motion_jitter=   0.00 ticks   score=   10.5
  2.  249.99999 FPS    block= 248.000 ms   jitter=   0.00 ms   motion_jitter=   0.00 ticks   score=   10.5
  3.  202.00526 FPS    block= 247.901 ms   jitter=   0.00 ms   motion_jitter=   1.00 ticks   score=   11.4
  4.  201.99999 FPS    block= 247.901 ms   jitter=   0.00 ms   motion_jitter=   1.00 ticks   score=   11.4
  5.  197.79473 FPS    block= 248.111 ms   jitter=   0.00 ms   motion_jitter=   0.98 ticks   score=   11.6
  6.  246.24655 FPS    block= 248.122 ms   jitter=   0.00 ms   motion_jitter=   1.26 ticks   score=   11.9
  7.  245.99999 FPS    block= 248.130 ms   jitter=   0.00 ms   motion_jitter=   1.26 ticks   score=   11.9
  8.  206.21578 FPS    block= 247.699 ms   jitter=   0.00 ms   motion_jitter=   1.75 ticks   score=   12.0
  9.  205.99999 FPS    block= 247.709 ms   jitter=   0.00 ms   motion_jitter=   1.75 ticks   score=   12.0
 10.  210.42631 FPS    block= 247.505 ms   jitter=   0.00 ms   motion_jitter=   2.14 ticks   score=   12.2
 11.  209.99999 FPS    block= 247.524 ms   jitter=   0.00 ms   motion_jitter=   2.14 ticks   score=   12.2
 12.  200.99999 FPS    block= 248.950 ms   jitter=   0.00 ms   motion_jitter=   0.71 ticks   score=   12.2
 13.  248.99999 FPS    block= 249.032 ms   jitter=   0.00 ms   motion_jitter=   0.64 ticks   score=   12.2
 14.  253.53825 FPS    block= 248.888 ms   jitter=   0.00 ms   motion_jitter=   1.08 ticks   score=   12.5
 15.  252.99999 FPS    block= 248.905 ms   jitter=   0.00 ms   motion_jitter=   1.08 ticks   score=   12.5
 16.  242.07118 FPS    block= 248.262 ms   jitter=   0.00 ms   motion_jitter=   1.72 ticks   score=   12.5
 17.  241.99999 FPS    block= 248.264 ms   jitter=   0.00 ms   motion_jitter=   1.72 ticks   score=   12.5
 18.  199.99999 FPS    block= 250.000 ms   jitter=   0.00 ms   motion_jitter=   0.00 ticks   score=   12.5
 19.  230.50796 FPS    block= 247.676 ms   jitter=   0.00 ms   motion_jitter=   2.38 ticks   score=   12.6
 20.  229.99999 FPS    block= 247.696 ms   jitter=   0.00 ms   motion_jitter=   2.38 ticks   score=   12.6
 21.  196.96498 FPS    block= 249.154 ms   jitter=   0.00 ms   motion_jitter=   1.12 ticks   score=   12.8
 22.  226.31509 FPS    block= 247.837 ms   jitter=   0.00 ms   motion_jitter=   2.47 ticks   score=   12.8
 23.  225.99999 FPS    block= 247.850 ms   jitter=   0.00 ms   motion_jitter=   2.47 ticks   score=   12.8
 24.  257.69625 FPS    block= 248.761 ms   jitter=   0.00 ms   motion_jitter=   1.58 ticks   score=   12.9
 25.  256.99999 FPS    block= 248.782 ms   jitter=   0.00 ms   motion_jitter=   1.58 ticks   score=   12.9
 26.  204.99999 FPS    block= 248.756 ms   jitter=   0.00 ms   motion_jitter=   1.64 ticks   score=   12.9
 27.  237.89582 FPS    block= 248.407 ms   jitter=   0.00 ms   motion_jitter=   2.04 ticks   score=   13.0
 28.  222.12222 FPS    block= 248.004 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   13.0
 29.  221.99999 FPS    block= 248.009 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   13.0
 30.  261.85426 FPS    block= 248.638 ms   jitter=   0.00 ms   motion_jitter=   1.90 ticks   score=   13.0
 31.  244.99999 FPS    block= 249.163 ms   jitter=   0.00 ms   motion_jitter=   1.39 ticks   score=   13.1
 32.  260.99999 FPS    block= 248.663 ms   jitter=   0.00 ms   motion_jitter=   1.90 ticks   score=   13.1
 33.  217.92935 FPS    block= 248.177 ms   jitter=   0.00 ms   motion_jitter=   2.45 ticks   score=   13.1
 34.  266.01226 FPS    block= 248.518 ms   jitter=   0.00 ms   motion_jitter=   2.12 ticks   score=   13.1
 35.  265.99999 FPS    block= 248.519 ms   jitter=   0.00 ms   motion_jitter=   2.12 ticks   score=   13.1
 36.  208.99999 FPS    block= 248.569 ms   jitter=   0.00 ms   motion_jitter=   2.07 ticks   score=   13.2
 37.  270.17027 FPS    block= 248.403 ms   jitter=   0.00 ms   motion_jitter=   2.27 ticks   score=   13.2
 38.  213.73647 FPS    block= 248.357 ms   jitter=   0.00 ms   motion_jitter=   2.32 ticks   score=   13.2
 39.  269.99999 FPS    block= 248.407 ms   jitter=   0.00 ms   motion_jitter=   2.27 ticks   score=   13.2
 40.  233.72045 FPS    block= 248.557 ms   jitter=   0.00 ms   motion_jitter=   2.26 ticks   score=   13.3
 41.  251.99999 FPS    block= 249.937 ms   jitter=   0.00 ms   motion_jitter=   0.89 ticks   score=   13.3
 42.  232.99999 FPS    block= 248.584 ms   jitter=   0.00 ms   motion_jitter=   2.26 ticks   score=   13.4
 43.  247.99999 FPS    block= 250.065 ms   jitter=   0.00 ms   motion_jitter=   0.90 ticks   score=   13.5
 44.  192.77211 FPS    block= 249.375 ms   jitter=   0.00 ms   motion_jitter=   1.59 ticks   score=   13.5
 45.  240.99999 FPS    block= 249.299 ms   jitter=   0.00 ms   motion_jitter=   1.81 ticks   score=   13.6
 46.  228.99999 FPS    block= 248.734 ms   jitter=   0.00 ms   motion_jitter=   2.41 ticks   score=   13.6
 47.  255.99999 FPS    block= 249.813 ms   jitter=   0.00 ms   motion_jitter=   1.48 ticks   score=   13.8
 48.  203.99999 FPS    block= 249.804 ms   jitter=   0.00 ms   motion_jitter=   1.51 ticks   score=   13.8
 49.  224.99999 FPS    block= 248.889 ms   jitter=   0.00 ms   motion_jitter=   2.48 ticks   score=   13.9
 50.  289.75507 FPS    block= 248.902 ms   jitter=   0.00 ms   motion_jitter=   2.49 ticks   score=   13.9
 51.  288.99999 FPS    block= 248.920 ms   jitter=   0.00 ms   motion_jitter=   2.49 ticks   score=   13.9
 52.  195.99999 FPS    block= 250.204 ms   jitter=   0.00 ms   motion_jitter=   1.23 ticks   score=   14.0
 53.  285.61428 FPS    block= 249.002 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   14.0
 54.  284.99999 FPS    block= 249.018 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   14.0
 55.  259.99999 FPS    block= 249.692 ms   jitter=   0.00 ms   motion_jitter=   1.83 ticks   score=   14.0
 56.  236.90623 FPS    block= 249.442 ms   jitter=   0.00 ms   motion_jitter=   2.10 ticks   score=   14.0
 57.  220.99999 FPS    block= 249.050 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   14.1
 58.  281.47349 FPS    block= 249.105 ms   jitter=   0.00 ms   motion_jitter=   2.48 ticks   score=   14.1
 59.  280.99999 FPS    block= 249.117 ms   jitter=   0.00 ms   motion_jitter=   2.48 ticks   score=   14.1
 60.  309.17835 FPS    block= 249.469 ms   jitter=   0.00 ms   motion_jitter=   2.14 ticks   score=   14.1
 61.  308.99999 FPS    block= 249.472 ms   jitter=   0.00 ms   motion_jitter=   2.14 ticks   score=   14.1
 62.  264.91035 FPS    block= 249.550 ms   jitter=   0.00 ms   motion_jitter=   2.07 ticks   score=   14.1
 63.  198.99999 FPS    block= 251.050 ms   jitter=   0.00 ms   motion_jitter=   0.57 ticks   score=   14.1
 64.  277.33271 FPS    block= 249.212 ms   jitter=   0.00 ms   motion_jitter=   2.44 ticks   score=   14.2

=== HIGHEST JITTER (ms, native engine) ===
  1.  192.88211 FPS    jitter=   2.49 ms   block= 251.988 ms
  2.  197.87814 FPS    jitter=   2.28 ms   block= 250.060 ms
  3.  242.18118 FPS    jitter=   2.00 ms   block= 250.354 ms
  4.  238.00582 FPS    jitter=   2.00 ms   block= 250.498 ms
  5.  237.99999 FPS    jitter=   2.00 ms   block= 250.499 ms
  6.  233.83045 FPS    jitter=   2.00 ms   block= 250.648 ms
  7.  230.61796 FPS    jitter=   2.00 ms   block= 249.768 ms
  8.  226.42509 FPS    jitter=   2.00 ms   block= 249.928 ms
  9.  222.23222 FPS    jitter=   2.00 ms   block= 250.095 ms
 10.  218.03935 FPS    jitter=   2.00 ms   block= 250.268 ms
 11.  213.84647 FPS    jitter=   2.00 ms   block= 250.448 ms
 12.  210.53631 FPS    jitter=   2.00 ms   block= 249.595 ms
 13.  206.32578 FPS    jitter=   2.00 ms   block= 249.789 ms
 14.  202.11526 FPS    jitter=   2.00 ms   block= 249.991 ms
 15.  217.99999 FPS    jitter=   1.94 ms   block= 249.698 ms
 16.  246.31657 FPS    jitter=   1.89 ms   block= 249.596 ms
 17.  316.23176 FPS    jitter=   1.50 ms   block= 251.896 ms
 18.  312.12498 FPS    jitter=   1.50 ms   block= 251.979 ms
 19.  309.28835 FPS    jitter=   1.50 ms   block= 251.038 ms
 20.  305.16463 FPS    jitter=   1.50 ms   block= 251.125 ms
 21.  301.04092 FPS    jitter=   1.50 ms   block= 251.215 ms
 22.  296.91721 FPS    jitter=   1.50 ms   block= 251.307 ms
 23.  292.79350 FPS    jitter=   1.50 ms   block= 251.402 ms
 24.  289.86507 FPS    jitter=   1.50 ms   block= 250.471 ms
 25.  285.72428 FPS    jitter=   1.50 ms   block= 250.571 ms
 26.  281.58349 FPS    jitter=   1.50 ms   block= 250.674 ms
 27.  277.44271 FPS    jitter=   1.50 ms   block= 250.780 ms
 28.  273.30192 FPS    jitter=   1.50 ms   block= 250.889 ms
 29.  270.28027 FPS    jitter=   1.50 ms   block= 249.971 ms
 30.  266.12226 FPS    jitter=   1.50 ms   block= 250.087 ms
 31.  261.96426 FPS    jitter=   1.50 ms   block= 250.206 ms
 32.  257.80625 FPS    jitter=   1.50 ms   block= 250.329 ms
 33.  253.64825 FPS    jitter=   1.50 ms   block= 250.456 ms
 34.  250.53192 FPS    jitter=   1.50 ms   block= 249.554 ms
 35.  300.99999 FPS    jitter=   1.41 ms   block= 250.645 ms
 36.  262.00426 FPS    jitter=   1.35 ms   block= 250.776 ms
 37.  297.00721 FPS    jitter=   0.64 ms   block= 252.591 ms
 38.  193.00849 FPS    jitter=   0.50 ms   block= 253.886 ms
 39.  194.60698 FPS    jitter=   0.50 ms   block= 251.801 ms
 40.  195.41620 FPS    jitter=   0.50 ms   block= 250.758 ms
 41.  196.23218 FPS    jitter=   0.50 ms   block= 249.716 ms
 42.  198.74777 FPS    jitter=   0.50 ms   block= 251.587 ms
 43.  199.57420 FPS    jitter=   0.50 ms   block= 250.545 ms
 44.  200.40754 FPS    jitter=   0.50 ms   block= 249.503 ms
 45.  201.24787 FPS    jitter=   0.50 ms   block= 248.462 ms
 46.  203.73221 FPS    jitter=   0.50 ms   block= 250.341 ms
 47.  204.58291 FPS    jitter=   0.50 ms   block= 249.300 ms
 48.  205.44074 FPS    jitter=   0.50 ms   block= 248.259 ms
 49.  207.02934 FPS    jitter=   0.50 ms   block= 251.184 ms
 50.  208.75827 FPS    jitter=   0.50 ms   block= 249.104 ms
 51.  209.63361 FPS    jitter=   0.50 ms   block= 248.064 ms
 52.  211.17013 FPS    jitter=   0.50 ms   block= 250.995 ms
 53.  212.04822 FPS    jitter=   0.50 ms   block= 249.956 ms
 54.  214.42299 FPS    jitter=   0.50 ms   block= 251.851 ms
 55.  215.31092 FPS    jitter=   0.50 ms   block= 250.813 ms
 56.  216.20622 FPS    jitter=   0.50 ms   block= 249.774 ms
 57.  219.45170 FPS    jitter=   0.50 ms   block= 250.637 ms
 58.  220.36423 FPS    jitter=   0.50 ms   block= 249.600 ms
 59.  221.28437 FPS    jitter=   0.50 ms   block= 248.562 ms
 60.  223.59249 FPS    jitter=   0.50 ms   block= 250.469 ms
 61.  224.52223 FPS    jitter=   0.50 ms   block= 249.432 ms
 62.  225.45973 FPS    jitter=   0.50 ms   block= 248.395 ms
 63.  227.73328 FPS    jitter=   0.50 ms   block= 250.306 ms
 64.  228.68023 FPS    jitter=   0.50 ms   block= 249.270 ms

=== HIGHEST MOTION JITTER (ticks, native engine) ===
  1.  285.72428 FPS    motion_jitter=   2.50 ticks   block= 250.571 ms
  2.  222.23222 FPS    motion_jitter=   2.50 ticks   block= 250.095 ms
  3.  286.28857 FPS    motion_jitter=   2.50 ticks   block= 251.510 ms
  4.  284.52609 FPS    motion_jitter=   2.50 ticks   block= 249.553 ms
  5.  221.28437 FPS    motion_jitter=   2.50 ticks   block= 248.562 ms
  6.  287.46434 FPS    motion_jitter=   2.50 ticks   block= 250.481 ms
  7.  223.59249 FPS    motion_jitter=   2.50 ticks   block= 250.469 ms
  8.  283.35756 FPS    motion_jitter=   2.50 ticks   block= 250.582 ms
  9.  288.64980 FPS    motion_jitter=   2.49 ticks   block= 249.453 ms
 10.  220.36423 FPS    motion_jitter=   2.49 ticks   block= 249.600 ms
 11.  282.19859 FPS    motion_jitter=   2.49 ticks   block= 251.611 ms
 12.  289.86507 FPS    motion_jitter=   2.49 ticks   block= 250.471 ms
 13.  224.52223 FPS    motion_jitter=   2.49 ticks   block= 249.432 ms
 14.  281.58349 FPS    motion_jitter=   2.48 ticks   block= 250.674 ms
 15.  290.37855 FPS    motion_jitter=   2.48 ticks   block= 251.411 ms
 16.  219.45170 FPS    motion_jitter=   2.48 ticks   block= 250.637 ms
 17.  225.45973 FPS    motion_jitter=   2.48 ticks   block= 248.395 ms
 18.  280.40238 FPS    motion_jitter=   2.48 ticks   block= 249.656 ms
 19.  291.57111 FPS    motion_jitter=   2.48 ticks   block= 250.383 ms
 20.  226.42509 FPS    motion_jitter=   2.47 ticks   block= 249.928 ms
 21.  292.79350 FPS    motion_jitter=   2.47 ticks   block= 251.402 ms
 22.  279.25079 FPS    motion_jitter=   2.47 ticks   block= 250.686 ms
 23.  293.26903 FPS    motion_jitter=   2.46 ticks   block= 252.343 ms
 24.  218.03935 FPS    motion_jitter=   2.46 ticks   block= 250.268 ms
 25.  217.99999 FPS    motion_jitter=   2.46 ticks   block= 249.698 ms
 26.  278.10861 FPS    motion_jitter=   2.45 ticks   block= 251.715 ms
 27.  294.46853 FPS    motion_jitter=   2.45 ticks   block= 251.316 ms
 28.  277.44271 FPS    motion_jitter=   2.44 ticks   block= 250.780 ms
 29.  227.73328 FPS    motion_jitter=   2.44 ticks   block= 250.306 ms
 30.  295.67789 FPS    motion_jitter=   2.43 ticks   block= 250.288 ms
 31.  276.27866 FPS    motion_jitter=   2.43 ticks   block= 249.763 ms
 32.  216.20622 FPS    motion_jitter=   2.42 ticks   block= 249.774 ms
 33.  228.68023 FPS    motion_jitter=   2.42 ticks   block= 249.270 ms
 34.  296.91721 FPS    motion_jitter=   2.42 ticks   block= 251.307 ms
 35.  297.00721 FPS    motion_jitter=   2.41 ticks   block= 252.591 ms
 36.  275.14401 FPS    motion_jitter=   2.41 ticks   block= 250.793 ms
 37.  215.31092 FPS    motion_jitter=   2.39 ticks   block= 250.813 ms
 38.  229.63510 FPS    motion_jitter=   2.39 ticks   block= 248.233 ms
 39.  274.01863 FPS    motion_jitter=   2.39 ticks   block= 251.823 ms
 40.  298.55851 FPS    motion_jitter=   2.38 ticks   block= 251.223 ms
 41.  230.61796 FPS    motion_jitter=   2.37 ticks   block= 249.768 ms
 42.  273.30192 FPS    motion_jitter=   2.37 ticks   block= 250.889 ms
 43.  299.78467 FPS    motion_jitter=   2.36 ticks   block= 250.195 ms
 44.  214.42299 FPS    motion_jitter=   2.36 ticks   block= 251.851 ms
 45.  300.00000 FPS    motion_jitter=   2.35 ticks   block= 249.667 ms
 46.  301.04092 FPS    motion_jitter=   2.34 ticks   block= 251.215 ms
 47.  272.15495 FPS    motion_jitter=   2.34 ticks   block= 249.873 ms
 48.  213.84647 FPS    motion_jitter=   2.33 ticks   block= 250.448 ms
 49.  231.87406 FPS    motion_jitter=   2.32 ticks   block= 250.149 ms
 50.  271.03723 FPS    motion_jitter=   2.31 ticks   block= 250.903 ms
 51.  302.64849 FPS    motion_jitter=   2.30 ticks   block= 251.132 ms
 52.  270.28027 FPS    motion_jitter=   2.29 ticks   block= 249.971 ms
 53.  232.83824 FPS    motion_jitter=   2.28 ticks   block= 249.113 ms
 54.  303.89144 FPS    motion_jitter=   2.27 ticks   block= 250.105 ms
 55.  304.00001 FPS    motion_jitter=   2.26 ticks   block= 249.627 ms
 56.  269.14114 FPS    motion_jitter=   2.25 ticks   block= 248.955 ms
 57.  212.04822 FPS    motion_jitter=   2.25 ticks   block= 249.956 ms
 58.  233.83045 FPS    motion_jitter=   2.25 ticks   block= 250.648 ms
 59.  305.16463 FPS    motion_jitter=   2.25 ticks   block= 251.125 ms
 60.  234.07625 FPS    motion_jitter=   2.23 ticks   block= 252.068 ms
 61.  268.03124 FPS    motion_jitter=   2.22 ticks   block= 249.986 ms
 62.  267.99124 FPS    motion_jitter=   2.21 ticks   block= 250.177 ms
 63.  211.17013 FPS    motion_jitter=   2.20 ticks   block= 250.995 ms
 64.  306.73847 FPS    motion_jitter=   2.19 ticks   block= 251.044 ms


Sweet spot target: 320 FPS (range ±64 FPS, direction both)

=== FASTEST BLOCK (lower ms, native engine) ===
  1.  270.17027 FPS    block= 248.403 ms   jitter=   0.00 ms
  2.  269.99999 FPS    block= 248.407 ms   jitter=   0.00 ms
  3.  266.01226 FPS    block= 248.518 ms   jitter=   0.00 ms
  4.  265.99999 FPS    block= 248.519 ms   jitter=   0.00 ms
  5.  261.85426 FPS    block= 248.638 ms   jitter=   0.00 ms
  6.  260.99999 FPS    block= 248.663 ms   jitter=   0.00 ms
  7.  257.69625 FPS    block= 248.761 ms   jitter=   0.00 ms
  8.  256.99999 FPS    block= 248.782 ms   jitter=   0.00 ms
  9.  289.75507 FPS    block= 248.902 ms   jitter=   0.00 ms
 10.  288.99999 FPS    block= 248.920 ms   jitter=   0.00 ms
 11.  285.61428 FPS    block= 249.002 ms   jitter=   0.00 ms
 12.  284.99999 FPS    block= 249.018 ms   jitter=   0.00 ms
 13.  281.47349 FPS    block= 249.105 ms   jitter=   0.00 ms
 14.  280.99999 FPS    block= 249.117 ms   jitter=   0.00 ms
 15.  264.99999 FPS    block= 249.119 ms   jitter=   0.50 ms
 16.  277.33271 FPS    block= 249.212 ms   jitter=   0.00 ms
 17.  276.99999 FPS    block= 249.220 ms   jitter=   0.00 ms
 18.  273.19192 FPS    block= 249.321 ms   jitter=   0.00 ms
 19.  272.99999 FPS    block= 249.326 ms   jitter=   0.00 ms
 20.  268.99999 FPS    block= 249.435 ms   jitter=   0.00 ms
 21.  309.17835 FPS    block= 249.469 ms   jitter=   0.00 ms
 22.  308.99999 FPS    block= 249.472 ms   jitter=   0.00 ms
 23.  305.05463 FPS    block= 249.556 ms   jitter=   0.00 ms
 24.  304.99999 FPS    block= 249.557 ms   jitter=   0.00 ms
 25.  303.99999 FPS    block= 249.627 ms   jitter=   0.21 ms
 26.  300.93092 FPS    block= 249.646 ms   jitter=   0.00 ms
 27.  299.99999 FPS    block= 249.667 ms   jitter=   0.00 ms
 28.  263.99999 FPS    block= 249.671 ms   jitter=   0.29 ms
 29.  259.99999 FPS    block= 249.692 ms   jitter=   0.00 ms
 30.  296.80721 FPS    block= 249.738 ms   jitter=   0.00 ms
 31.  295.99999 FPS    block= 249.757 ms   jitter=   0.00 ms
 32.  292.68350 FPS    block= 249.833 ms   jitter=   0.00 ms
 33.  291.99999 FPS    block= 249.849 ms   jitter=   0.00 ms
 34.  287.99999 FPS    block= 249.944 ms   jitter=   0.00 ms
 35.  307.99999 FPS    block= 250.017 ms   jitter=   0.50 ms
 36.  283.99999 FPS    block= 250.042 ms   jitter=   0.00 ms
 37.  328.44209 FPS    block= 250.089 ms   jitter=   0.00 ms
 38.  327.99999 FPS    block= 250.098 ms   jitter=   0.00 ms
 39.  279.99999 FPS    block= 250.143 ms   jitter=   0.00 ms
 40.  324.33531 FPS    block= 250.166 ms   jitter=   0.00 ms
 41.  323.99999 FPS    block= 250.173 ms   jitter=   0.00 ms
 42.  267.99999 FPS    block= 250.177 ms   jitter=   0.45 ms
 43.  320.22854 FPS    block= 250.246 ms   jitter=   0.00 ms
 44.  275.99999 FPS    block= 250.246 ms   jitter=   0.00 ms
 45.  319.99999 FPS    block= 250.250 ms   jitter=   0.00 ms
 46.  316.12176 FPS    block= 250.327 ms   jitter=   0.00 ms
 47.  315.99999 FPS    block= 250.329 ms   jitter=   0.00 ms
 48.  271.99999 FPS    block= 250.353 ms   jitter=   0.00 ms
 49.  312.01498 FPS    block= 250.410 ms   jitter=   0.00 ms
 50.  311.99999 FPS    block= 250.410 ms   jitter=   0.00 ms
 51.  310.99999 FPS    block= 250.431 ms   jitter=   0.00 ms
 52.  314.99999 FPS    block= 250.492 ms   jitter=   0.35 ms
 53.  306.99999 FPS    block= 250.515 ms   jitter=   0.00 ms
 54.  302.99999 FPS    block= 250.601 ms   jitter=   0.00 ms
 55.  262.99999 FPS    block= 250.605 ms   jitter=   0.00 ms
 56.  301.00092 FPS    block= 250.644 ms   jitter=   1.41 ms
 57.  298.99999 FPS    block= 250.689 ms   jitter=   0.00 ms
 58.  258.99999 FPS    block= 250.722 ms   jitter=   0.00 ms
 59.  347.54826 FPS    block= 250.755 ms   jitter=   0.00 ms
 60.  346.99999 FPS    block= 250.764 ms   jitter=   0.00 ms
 61.  294.99999 FPS    block= 250.780 ms   jitter=   0.00 ms
 62.  343.45828 FPS    block= 250.823 ms   jitter=   0.00 ms
 63.  342.99999 FPS    block= 250.831 ms   jitter=   0.00 ms
 64.  318.99999 FPS    block= 250.841 ms   jitter=   0.50 ms

=== SLOWEST BLOCK (higher ms, native engine) ===
  1.  369.26836 FPS    block= 254.416 ms   jitter=   0.00 ms
  2.  370.00000 FPS    block= 254.405 ms   jitter=   0.00 ms
  3.  373.32516 FPS    block= 254.357 ms   jitter=   0.00 ms
  4.  374.00000 FPS    block= 254.348 ms   jitter=   0.00 ms
  5.  377.38195 FPS    block= 254.300 ms   jitter=   0.00 ms
  6.  378.00000 FPS    block= 254.291 ms   jitter=   0.00 ms
  7.  381.43875 FPS    block= 254.243 ms   jitter=   0.00 ms
  8.  382.00000 FPS    block= 254.236 ms   jitter=   0.00 ms
  9.  350.40550 FPS    block= 253.708 ms   jitter=   0.00 ms
 10.  351.00000 FPS    block= 253.698 ms   jitter=   0.00 ms
 11.  354.47882 FPS    block= 253.642 ms   jitter=   0.00 ms
 12.  355.00000 FPS    block= 253.634 ms   jitter=   0.00 ms
 13.  358.55214 FPS    block= 253.578 ms   jitter=   0.00 ms
 14.  359.00000 FPS    block= 253.571 ms   jitter=   0.00 ms
 15.  362.62546 FPS    block= 253.515 ms   jitter=   0.00 ms
 16.  363.00000 FPS    block= 253.510 ms   jitter=   0.00 ms
 17.  366.69878 FPS    block= 253.454 ms   jitter=   0.00 ms
 18.  367.00000 FPS    block= 253.450 ms   jitter=   0.00 ms
 19.  312.21499 FPS    block= 253.406 ms   jitter=   0.00 ms
 20.  371.00000 FPS    block= 253.391 ms   jitter=   0.00 ms
 21.  375.00000 FPS    block= 253.333 ms   jitter=   0.00 ms
 22.  316.32177 FPS    block= 253.323 ms   jitter=   0.00 ms
 23.  379.00000 FPS    block= 253.277 ms   jitter=   0.00 ms
 24.  383.00000 FPS    block= 253.222 ms   jitter=   0.00 ms
 25.  331.38835 FPS    block= 253.178 ms   jitter=   0.35 ms
 26.  332.00000 FPS    block= 253.024 ms   jitter=   0.00 ms
 27.  320.42855 FPS    block= 253.004 ms   jitter=   0.43 ms
 28.  335.47833 FPS    block= 252.962 ms   jitter=   0.00 ms
 29.  336.00000 FPS    block= 252.952 ms   jitter=   0.00 ms
 30.  339.56831 FPS    block= 252.890 ms   jitter=   0.00 ms
 31.  340.00000 FPS    block= 252.882 ms   jitter=   0.00 ms
 32.  292.88351 FPS    block= 252.829 ms   jitter=   0.00 ms
 33.  293.00000 FPS    block= 252.826 ms   jitter=   0.00 ms
 34.  343.65829 FPS    block= 252.820 ms   jitter=   0.00 ms
 35.  344.00000 FPS    block= 252.814 ms   jitter=   0.00 ms
 36.  347.74827 FPS    block= 252.751 ms   jitter=   0.00 ms
 37.  348.00000 FPS    block= 252.747 ms   jitter=   0.00 ms
 38.  297.00722 FPS    block= 252.734 ms   jitter=   0.00 ms
 39.  324.53532 FPS    block= 252.686 ms   jitter=   0.50 ms
 40.  352.00000 FPS    block= 252.682 ms   jitter=   0.00 ms
 41.  301.13093 FPS    block= 252.642 ms   jitter=   0.00 ms
 42.  356.00000 FPS    block= 252.618 ms   jitter=   0.00 ms
 43.  296.99722 FPS    block= 252.591 ms   jitter=   0.64 ms
 44.  360.00000 FPS    block= 252.556 ms   jitter=   0.00 ms
 45.  305.25464 FPS    block= 252.552 ms   jitter=   0.00 ms
 46.  364.00000 FPS    block= 252.495 ms   jitter=   0.00 ms
 47.  365.00000 FPS    block= 252.479 ms   jitter=   0.00 ms
 48.  309.37836 FPS    block= 252.465 ms   jitter=   0.00 ms
 49.  368.00000 FPS    block= 252.435 ms   jitter=   0.00 ms
 50.  313.00000 FPS    block= 252.390 ms   jitter=   0.00 ms
 51.  372.00000 FPS    block= 252.376 ms   jitter=   0.00 ms
 52.  328.64210 FPS    block= 252.371 ms   jitter=   0.45 ms
 53.  376.00000 FPS    block= 252.319 ms   jitter=   0.00 ms
 54.  273.39193 FPS    block= 252.316 ms   jitter=   0.00 ms
 55.  317.00000 FPS    block= 252.309 ms   jitter=   0.00 ms
 56.  361.00000 FPS    block= 252.302 ms   jitter=   0.43 ms
 57.  380.00000 FPS    block= 252.263 ms   jitter=   0.00 ms
 58.  321.00000 FPS    block= 252.231 ms   jitter=   0.00 ms
 59.  384.00000 FPS    block= 252.208 ms   jitter=   0.00 ms
 60.  277.53272 FPS    block= 252.206 ms   jitter=   0.00 ms
 61.  278.00000 FPS    block= 252.194 ms   jitter=   0.00 ms
 62.  325.00000 FPS    block= 252.154 ms   jitter=   0.00 ms
 63.  281.67350 FPS    block= 252.100 ms   jitter=   0.00 ms
 64.  282.00000 FPS    block= 252.092 ms   jitter=   0.00 ms

=== BEST ACCURACY (closest to 237.5 ms repeat interval, native engine) ===
(Lower score = interval dev. + 4.0 * block_jitter + 1.0 * motion_jitter)
  1.  257.69625 FPS    block= 248.761 ms   jitter=   0.00 ms   motion_jitter=   1.58 ticks   score=   12.9
  2.  256.99999 FPS    block= 248.782 ms   jitter=   0.00 ms   motion_jitter=   1.58 ticks   score=   12.9
  3.  261.85426 FPS    block= 248.638 ms   jitter=   0.00 ms   motion_jitter=   1.90 ticks   score=   13.0
  4.  260.99999 FPS    block= 248.663 ms   jitter=   0.00 ms   motion_jitter=   1.90 ticks   score=   13.1
  5.  266.01226 FPS    block= 248.518 ms   jitter=   0.00 ms   motion_jitter=   2.12 ticks   score=   13.1
  6.  265.99999 FPS    block= 248.519 ms   jitter=   0.00 ms   motion_jitter=   2.12 ticks   score=   13.1
  7.  270.17027 FPS    block= 248.403 ms   jitter=   0.00 ms   motion_jitter=   2.27 ticks   score=   13.2
  8.  269.99999 FPS    block= 248.407 ms   jitter=   0.00 ms   motion_jitter=   2.27 ticks   score=   13.2
  9.  328.44209 FPS    block= 250.089 ms   jitter=   0.00 ms   motion_jitter=   1.09 ticks   score=   13.7
 10.  327.99999 FPS    block= 250.098 ms   jitter=   0.00 ms   motion_jitter=   1.09 ticks   score=   13.7
 11.  289.75507 FPS    block= 248.902 ms   jitter=   0.00 ms   motion_jitter=   2.49 ticks   score=   13.9
 12.  288.99999 FPS    block= 248.920 ms   jitter=   0.00 ms   motion_jitter=   2.49 ticks   score=   13.9
 13.  285.61428 FPS    block= 249.002 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   14.0
 14.  335.27832 FPS    block= 250.965 ms   jitter=   0.00 ms   motion_jitter=   0.55 ticks   score=   14.0
 15.  284.99999 FPS    block= 249.018 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   14.0
 16.  334.99999 FPS    block= 250.970 ms   jitter=   0.00 ms   motion_jitter=   0.55 ticks   score=   14.0
 17.  259.99999 FPS    block= 249.692 ms   jitter=   0.00 ms   motion_jitter=   1.83 ticks   score=   14.0
 18.  324.33531 FPS    block= 250.166 ms   jitter=   0.00 ms   motion_jitter=   1.42 ticks   score=   14.1
 19.  281.47349 FPS    block= 249.105 ms   jitter=   0.00 ms   motion_jitter=   2.48 ticks   score=   14.1
 20.  323.99999 FPS    block= 250.173 ms   jitter=   0.00 ms   motion_jitter=   1.42 ticks   score=   14.1
 21.  280.99999 FPS    block= 249.117 ms   jitter=   0.00 ms   motion_jitter=   2.48 ticks   score=   14.1
 22.  309.17835 FPS    block= 249.469 ms   jitter=   0.00 ms   motion_jitter=   2.14 ticks   score=   14.1
 23.  308.99999 FPS    block= 249.472 ms   jitter=   0.00 ms   motion_jitter=   2.14 ticks   score=   14.1
 24.  264.91035 FPS    block= 249.550 ms   jitter=   0.00 ms   motion_jitter=   2.07 ticks   score=   14.1
 25.  277.33271 FPS    block= 249.212 ms   jitter=   0.00 ms   motion_jitter=   2.44 ticks   score=   14.2
 26.  276.99999 FPS    block= 249.220 ms   jitter=   0.00 ms   motion_jitter=   2.44 ticks   score=   14.2
 27.  268.99999 FPS    block= 249.435 ms   jitter=   0.00 ms   motion_jitter=   2.24 ticks   score=   14.2
 28.  273.19192 FPS    block= 249.321 ms   jitter=   0.00 ms   motion_jitter=   2.36 ticks   score=   14.2
 29.  272.99999 FPS    block= 249.326 ms   jitter=   0.00 ms   motion_jitter=   2.36 ticks   score=   14.2
 30.  305.05463 FPS    block= 249.556 ms   jitter=   0.00 ms   motion_jitter=   2.25 ticks   score=   14.3
 31.  304.99999 FPS    block= 249.557 ms   jitter=   0.00 ms   motion_jitter=   2.25 ticks   score=   14.3
 32.  331.18834 FPS    block= 251.039 ms   jitter=   0.00 ms   motion_jitter=   0.78 ticks   score=   14.3
 33.  330.99999 FPS    block= 251.042 ms   jitter=   0.00 ms   motion_jitter=   0.78 ticks   score=   14.3
 34.  320.22854 FPS    block= 250.246 ms   jitter=   0.00 ms   motion_jitter=   1.67 ticks   score=   14.4
 35.  319.99999 FPS    block= 250.250 ms   jitter=   0.00 ms   motion_jitter=   1.67 ticks   score=   14.4
 36.  339.36830 FPS    block= 250.893 ms   jitter=   0.00 ms   motion_jitter=   1.07 ticks   score=   14.5
 37.  338.99999 FPS    block= 250.900 ms   jitter=   0.00 ms   motion_jitter=   1.07 ticks   score=   14.5
 38.  333.91221 FPS    block= 251.990 ms   jitter=   0.00 ms   motion_jitter=   0.00 ticks   score=   14.5
 39.  300.93092 FPS    block= 249.646 ms   jitter=   0.00 ms   motion_jitter=   2.35 ticks   score=   14.5
 40.  332.99999 FPS    block= 252.006 ms   jitter=   0.00 ms   motion_jitter=   0.00 ticks   score=   14.5
 41.  299.99999 FPS    block= 249.667 ms   jitter=   0.00 ms   motion_jitter=   2.35 ticks   score=   14.5
 42.  296.80721 FPS    block= 249.738 ms   jitter=   0.00 ms   motion_jitter=   2.42 ticks   score=   14.7
 43.  295.99999 FPS    block= 249.757 ms   jitter=   0.00 ms   motion_jitter=   2.42 ticks   score=   14.7
 44.  316.12176 FPS    block= 250.327 ms   jitter=   0.00 ms   motion_jitter=   1.87 ticks   score=   14.7
 45.  315.99999 FPS    block= 250.329 ms   jitter=   0.00 ms   motion_jitter=   1.87 ticks   score=   14.7
 46.  343.45828 FPS    block= 250.823 ms   jitter=   0.00 ms   motion_jitter=   1.38 ticks   score=   14.7
 47.  342.99999 FPS    block= 250.831 ms   jitter=   0.00 ms   motion_jitter=   1.38 ticks   score=   14.7
 48.  292.68350 FPS    block= 249.833 ms   jitter=   0.00 ms   motion_jitter=   2.47 ticks   score=   14.8
 49.  291.99999 FPS    block= 249.849 ms   jitter=   0.00 ms   motion_jitter=   2.47 ticks   score=   14.8
 50.  326.99999 FPS    block= 251.116 ms   jitter=   0.00 ms   motion_jitter=   1.21 ticks   score=   14.8
 51.  325.99999 FPS    block= 251.135 ms   jitter=   0.00 ms   motion_jitter=   1.21 ticks   score=   14.8
 52.  347.54826 FPS    block= 250.755 ms   jitter=   0.00 ms   motion_jitter=   1.61 ticks   score=   14.9
 53.  346.99999 FPS    block= 250.764 ms   jitter=   0.00 ms   motion_jitter=   1.61 ticks   score=   14.9
 54.  287.99999 FPS    block= 249.944 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   14.9
 55.  312.01498 FPS    block= 250.410 ms   jitter=   0.00 ms   motion_jitter=   2.04 ticks   score=   15.0
 56.  311.99999 FPS    block= 250.410 ms   jitter=   0.00 ms   motion_jitter=   2.04 ticks   score=   15.0
 57.  310.99999 FPS    block= 250.431 ms   jitter=   0.00 ms   motion_jitter=   2.04 ticks   score=   15.0
 58.  258.99999 FPS    block= 250.722 ms   jitter=   0.00 ms   motion_jitter=   1.75 ticks   score=   15.0
 59.  283.99999 FPS    block= 250.042 ms   jitter=   0.00 ms   motion_jitter=   2.50 ticks   score=   15.0
 60.  263.81752 FPS    block= 250.581 ms   jitter=   0.00 ms   motion_jitter=   2.01 ticks   score=   15.1
 61.  279.99999 FPS    block= 250.143 ms   jitter=   0.00 ms   motion_jitter=   2.47 ticks   score=   15.1
 62.  262.99999 FPS    block= 250.605 ms   jitter=   0.00 ms   motion_jitter=   2.01 ticks   score=   15.1
 63.  267.94123 FPS    block= 250.464 ms   jitter=   0.00 ms   motion_jitter=   2.20 ticks   score=   15.2
 64.  275.99999 FPS    block= 250.246 ms   jitter=   0.00 ms   motion_jitter=   2.42 ticks   score=   15.2

=== HIGHEST JITTER (ms, native engine) ===
  1.  324.44361 FPS    jitter=   1.50 ms   block= 251.736 ms
  2.  320.33854 FPS    jitter=   1.50 ms   block= 251.815 ms
  3.  316.23176 FPS    jitter=   1.50 ms   block= 251.896 ms
  4.  312.12498 FPS    jitter=   1.50 ms   block= 251.979 ms
  5.  309.28835 FPS    jitter=   1.50 ms   block= 251.038 ms
  6.  305.16463 FPS    jitter=   1.50 ms   block= 251.125 ms
  7.  301.04092 FPS    jitter=   1.50 ms   block= 251.215 ms
  8.  296.91721 FPS    jitter=   1.50 ms   block= 251.307 ms
  9.  292.79350 FPS    jitter=   1.50 ms   block= 251.402 ms
 10.  289.86507 FPS    jitter=   1.50 ms   block= 250.471 ms
 11.  285.72428 FPS    jitter=   1.50 ms   block= 250.571 ms
 12.  281.58349 FPS    jitter=   1.50 ms   block= 250.674 ms
 13.  277.44271 FPS    jitter=   1.50 ms   block= 250.780 ms
 14.  273.30192 FPS    jitter=   1.50 ms   block= 250.889 ms
 15.  270.28027 FPS    jitter=   1.50 ms   block= 249.971 ms
 16.  266.12226 FPS    jitter=   1.50 ms   block= 250.087 ms
 17.  261.96426 FPS    jitter=   1.50 ms   block= 250.206 ms
 18.  257.80625 FPS    jitter=   1.50 ms   block= 250.329 ms
 19.  300.99999 FPS    jitter=   1.41 ms   block= 250.645 ms
 20.  328.52040 FPS    jitter=   1.36 ms   block= 251.136 ms
 21.  262.00426 FPS    jitter=   1.35 ms   block= 250.776 ms
 22.  331.27313 FPS    jitter=   1.19 ms   block= 252.037 ms
 23.  381.34874 FPS    jitter=   1.00 ms   block= 253.292 ms
 24.  377.29194 FPS    jitter=   1.00 ms   block= 253.349 ms
 25.  373.23515 FPS    jitter=   1.00 ms   block= 253.406 ms
 26.  369.17835 FPS    jitter=   1.00 ms   block= 253.465 ms
 27.  366.60877 FPS    jitter=   1.00 ms   block= 252.503 ms
 28.  362.53545 FPS    jitter=   1.00 ms   block= 252.564 ms
 29.  358.46213 FPS    jitter=   1.00 ms   block= 252.627 ms
 30.  354.38881 FPS    jitter=   1.00 ms   block= 252.691 ms
 31.  350.31549 FPS    jitter=   1.00 ms   block= 252.757 ms
 32.  347.65826 FPS    jitter=   1.00 ms   block= 251.800 ms
 33.  343.56828 FPS    jitter=   1.00 ms   block= 251.869 ms
 34.  339.47830 FPS    jitter=   1.00 ms   block= 251.939 ms
 35.  335.38832 FPS    jitter=   1.00 ms   block= 252.011 ms
 36.  297.00721 FPS    jitter=   0.64 ms   block= 252.591 ms
 37.  256.71878 FPS    jitter=   0.50 ms   block= 249.314 ms
 38.  258.71690 FPS    jitter=   0.50 ms   block= 251.254 ms
 39.  259.78382 FPS    jitter=   0.50 ms   block= 250.223 ms
 40.  260.85957 FPS    jitter=   0.50 ms   block= 249.191 ms
 41.  263.90753 FPS    jitter=   0.50 ms   block= 250.102 ms
 42.  265.00036 FPS    jitter=   0.50 ms   block= 249.071 ms
 43.  268.03124 FPS    jitter=   0.50 ms   block= 249.986 ms
 44.  269.14114 FPS    jitter=   0.50 ms   block= 248.955 ms
 45.  271.03723 FPS    jitter=   0.50 ms   block= 250.903 ms
 46.  272.15495 FPS    jitter=   0.50 ms   block= 249.873 ms
 47.  274.01863 FPS    jitter=   0.50 ms   block= 251.823 ms
 48.  275.14401 FPS    jitter=   0.50 ms   block= 250.793 ms
 49.  276.27866 FPS    jitter=   0.50 ms   block= 249.763 ms
 50.  278.10861 FPS    jitter=   0.50 ms   block= 251.715 ms
 51.  279.25079 FPS    jitter=   0.50 ms   block= 250.686 ms
 52.  280.40238 FPS    jitter=   0.50 ms   block= 249.656 ms
 53.  282.19859 FPS    jitter=   0.50 ms   block= 251.611 ms
 54.  283.35756 FPS    jitter=   0.50 ms   block= 250.582 ms
 55.  284.52609 FPS    jitter=   0.50 ms   block= 249.553 ms
 56.  286.28857 FPS    jitter=   0.50 ms   block= 251.510 ms
 57.  287.46434 FPS    jitter=   0.50 ms   block= 250.481 ms
 58.  288.64980 FPS    jitter=   0.50 ms   block= 249.453 ms
 59.  290.37855 FPS    jitter=   0.50 ms   block= 251.411 ms
 60.  291.57111 FPS    jitter=   0.50 ms   block= 250.383 ms
 61.  293.26903 FPS    jitter=   0.50 ms   block= 252.343 ms
 62.  294.46853 FPS    jitter=   0.50 ms   block= 251.316 ms
 63.  295.67789 FPS    jitter=   0.50 ms   block= 250.288 ms
 64.  298.55851 FPS    jitter=   0.50 ms   block= 251.223 ms

=== HIGHEST MOTION JITTER (ticks, native engine) ===
  1.  285.72428 FPS    motion_jitter=   2.50 ticks   block= 250.571 ms
  2.  286.28857 FPS    motion_jitter=   2.50 ticks   block= 251.510 ms
  3.  284.52609 FPS    motion_jitter=   2.50 ticks   block= 249.553 ms
  4.  287.46434 FPS    motion_jitter=   2.50 ticks   block= 250.481 ms
  5.  283.35756 FPS    motion_jitter=   2.50 ticks   block= 250.582 ms
  6.  288.64980 FPS    motion_jitter=   2.49 ticks   block= 249.453 ms
  7.  282.19859 FPS    motion_jitter=   2.49 ticks   block= 251.611 ms
  8.  289.86507 FPS    motion_jitter=   2.49 ticks   block= 250.471 ms
  9.  281.58349 FPS    motion_jitter=   2.48 ticks   block= 250.674 ms
 10.  290.37855 FPS    motion_jitter=   2.48 ticks   block= 251.411 ms
 11.  280.40238 FPS    motion_jitter=   2.48 ticks   block= 249.656 ms
 12.  291.57111 FPS    motion_jitter=   2.48 ticks   block= 250.383 ms
 13.  292.79350 FPS    motion_jitter=   2.47 ticks   block= 251.402 ms
 14.  279.25079 FPS    motion_jitter=   2.47 ticks   block= 250.686 ms
 15.  293.26903 FPS    motion_jitter=   2.46 ticks   block= 252.343 ms
 16.  278.10861 FPS    motion_jitter=   2.45 ticks   block= 251.715 ms
 17.  384.00000 FPS    motion_jitter=   2.45 ticks   block= 252.208 ms
 18.  294.46853 FPS    motion_jitter=   2.45 ticks   block= 251.316 ms
 19.  383.82839 FPS    motion_jitter=   2.44 ticks   block= 252.734 ms
 20.  277.44271 FPS    motion_jitter=   2.44 ticks   block= 250.780 ms
 21.  382.28377 FPS    motion_jitter=   2.43 ticks   block= 253.756 ms
 22.  295.67789 FPS    motion_jitter=   2.43 ticks   block= 250.288 ms
 23.  276.27866 FPS    motion_jitter=   2.43 ticks   block= 249.763 ms
 24.  381.34874 FPS    motion_jitter=   2.42 ticks   block= 253.292 ms
 25.  380.00256 FPS    motion_jitter=   2.42 ticks   block= 252.263 ms
 26.  296.91721 FPS    motion_jitter=   2.42 ticks   block= 251.307 ms
 27.  297.00721 FPS    motion_jitter=   2.41 ticks   block= 252.591 ms
 28.  379.78798 FPS    motion_jitter=   2.41 ticks   block= 252.790 ms
 29.  275.14401 FPS    motion_jitter=   2.41 ticks   block= 250.793 ms
 30.  378.25962 FPS    motion_jitter=   2.39 ticks   block= 253.811 ms
 31.  274.01863 FPS    motion_jitter=   2.39 ticks   block= 251.823 ms
 32.  298.55851 FPS    motion_jitter=   2.38 ticks   block= 251.223 ms
 33.  377.29194 FPS    motion_jitter=   2.38 ticks   block= 253.349 ms
 34.  376.00075 FPS    motion_jitter=   2.38 ticks   block= 252.319 ms
 35.  273.30192 FPS    motion_jitter=   2.37 ticks   block= 250.889 ms
 36.  375.74758 FPS    motion_jitter=   2.37 ticks   block= 252.847 ms
 37.  299.78467 FPS    motion_jitter=   2.36 ticks   block= 250.195 ms
 38.  300.00000 FPS    motion_jitter=   2.35 ticks   block= 249.667 ms
 39.  374.23548 FPS    motion_jitter=   2.35 ticks   block= 253.868 ms
 40.  301.04092 FPS    motion_jitter=   2.34 ticks   block= 251.215 ms
 41.  272.15495 FPS    motion_jitter=   2.34 ticks   block= 249.873 ms
 42.  373.23515 FPS    motion_jitter=   2.33 ticks   block= 253.406 ms
 43.  372.00012 FPS    motion_jitter=   2.32 ticks   block= 252.376 ms
 44.  271.03723 FPS    motion_jitter=   2.31 ticks   block= 250.903 ms
 45.  371.70718 FPS    motion_jitter=   2.31 ticks   block= 252.904 ms
 46.  302.64849 FPS    motion_jitter=   2.30 ticks   block= 251.132 ms
 47.  370.21133 FPS    motion_jitter=   2.29 ticks   block= 253.926 ms
 48.  270.28027 FPS    motion_jitter=   2.29 ticks   block= 249.971 ms
 49.  303.89144 FPS    motion_jitter=   2.27 ticks   block= 250.105 ms
 50.  369.17835 FPS    motion_jitter=   2.27 ticks   block= 253.465 ms
 51.  368.00030 FPS    motion_jitter=   2.26 ticks   block= 252.435 ms
 52.  304.00001 FPS    motion_jitter=   2.26 ticks   block= 249.627 ms
 53.  269.14114 FPS    motion_jitter=   2.25 ticks   block= 248.955 ms
 54.  305.16463 FPS    motion_jitter=   2.25 ticks   block= 251.125 ms
 55.  367.66677 FPS    motion_jitter=   2.24 ticks   block= 252.964 ms
 56.  366.60877 FPS    motion_jitter=   2.22 ticks   block= 252.503 ms
 57.  268.03124 FPS    motion_jitter=   2.22 ticks   block= 249.986 ms
 58.  267.99124 FPS    motion_jitter=   2.21 ticks   block= 250.177 ms
 59.  365.10157 FPS    motion_jitter=   2.20 ticks   block= 252.002 ms
 60.  306.73847 FPS    motion_jitter=   2.19 ticks   block= 251.044 ms
 61.  364.00070 FPS    motion_jitter=   2.18 ticks   block= 252.494 ms
 62.  363.62638 FPS    motion_jitter=   2.16 ticks   block= 253.024 ms
 63.  308.00000 FPS    motion_jitter=   2.16 ticks   block= 250.017 ms
 64.  307.99822 FPS    motion_jitter=   2.16 ticks   block= 250.017 ms
















    
