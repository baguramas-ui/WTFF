WTFF is a tool to compare IEEE 754 32 bit float errors in framerate/time variance
compared to 64 BIT RTSS /80/128 
thoses errors might induce jitter, the tool gives the most stable result

It was created to get an architecture adapted to the analysis of the realspace 2 
(Gunz engine) bugged frame distribution

It runs On https://www.onlinegdb.com/online_c_compiler#


Note, it's meant to show the stable framerate aroud your own target range, 
you CHOOSE the range then pick a fps proposed by the tool
that is close and stable. 
Dont use the first value simply because it's higher in ranking.


here is a sample of the IEEE 754 float mode

=== EXTREME + STABLE DOWN (best decimal per integer, filtered by target) ===
  1.  181.10323 FPS    avg=        -2.015704e-11  stddev=         2.002312e-10
  2.  180.93543 FPS    avg=        -1.992100e-11  stddev=         2.003844e-10
  3.   90.60160 FPS    avg=        -1.981682e-11  stddev=         3.904531e-10
  4.  362.10646 FPS    avg=        -1.819676e-11  stddev=         1.042515e-10

and here is a sample of the gunz mode.

=== BEST ACCURACY (closest to 237.5 ms repeat interval, native engine) ===
(Lower score = interval dev. + 4.0 * jitter)
  1.  199.98000 FPS    block= 247.509 ms   jitter=   0.23 ms   score=   10.9
  2.  200.00000 FPS    block= 247.506 ms   jitter=   0.24 ms   score=   10.9
  3.  200.00001 FPS    block= 247.506 ms   jitter=   0.24 ms   score=   10.9
  4.  250.00000 FPS    block= 248.000 ms   jitter=   0.24 ms   score=   11.4
  5.  249.99999 FPS    block= 248.000 ms   jitter=   0.24 ms   score=   11.4
  6.  333.33333 FPS    block= 250.286 ms   jitter=   0.21 ms   score=   13.6
