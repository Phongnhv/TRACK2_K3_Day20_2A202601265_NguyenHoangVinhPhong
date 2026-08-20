# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` � host `Windows-AMD64` � llama.cpp `b10488`
CPU: **24 physical � 32 logical** cores � `ngl=99` � metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 95.6 | 99% |
| 12 | 95.7 | 100% |
| 24 | 95.8 | 100% |
| 32 | 95.7 | 100% |
| 64 | 96.1 | 100% |

**Best**: `-t 64` at 96.1 tok/s
**Slowest tested**: `-t 1` at 95.6 tok/s (1.01x spread)
**Against the physical-core default** (`-t 24`, 95.8 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=64 make bench
```

## Your explanation

There is no sharp knee at the 24 physical or 32 logical cores: the curve is almost
flat from 1 through 64 threads (95.6--96.1 tok/s). With `ngl=99`, most layers are
offloaded to the RTX 4060, so decode is dominated by GPU execution and memory
traffic rather than host thread parallelism. The 64-thread result is only 1.00x
the 24-thread baseline; extra CPU threads have little useful work and could add
scheduling overhead, so I keep the physical-core default for serving.
