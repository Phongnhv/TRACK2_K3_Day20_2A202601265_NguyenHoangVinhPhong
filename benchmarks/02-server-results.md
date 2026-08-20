# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` � llama.cpp `b10488` �
`--parallel 4` � `ctx=2048` � `threads=24` �
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 195 | 3.30 | 2000 | 3200 | 4200 | 6.9 | 0.0% |
| 50 | 198 | 3.34 | 13000 | 15000 | 16000 | 41.4 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.01x** (20% of linear) |
| P95 latency | **4.69x** |
| Effective concurrency at 50 users | 41.4 vs `--parallel 4` slots (occupancy/slot ratio 10.34) |

**Saturated.** Throughput delivered only 1.01x for 5x the offered load, and effective concurrency (41.4) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.01x while P95 moved 4.69x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

The server is already saturated by the 50-user run: offered load rose 5x, but RPS
only moved from 3.30 to 3.34 (1.01x), while P95 grew from 3200 to 15000 ms (4.69x).
The strongest evidence is effective concurrency 41.4 versus only four decode slots,
plus 45 deferred requests and a 3.94/4 busy-slot peak. The added latency is queue
time. To improve goodput@SLO I would first use the Q2 quantization (and limit output
tokens/context) to shorten each decode; then retest `--parallel`, rather than adding
slots blindly to a GPU-bound workload.
