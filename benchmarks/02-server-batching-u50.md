# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` � `--parallel 4` � 15 samples over
60s at 2.0s intervals � raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.94 of 4 slots (99%) |
| `requests_processing` | 4 |
| `requests_deferred` | 45 |
| `kv_cache_usage_ratio` | n/a � not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 22772 |

Highest sampled value was **3.94 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation

The peak batch width was 3.94 of 4 slots (99%), so the scheduler was continuously
batching requests under load. This does not equal the 41.4 effective concurrency in
the load report: that value includes requests waiting in the queue, whereas the
llama.cpp gauge measures useful work per decode step. I therefore use 3.94/4 as the
batching evidence and 41.4 as the saturation/queueing evidence.
