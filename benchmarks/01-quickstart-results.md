# 01 - Measure: latency baseline

Model `Gemma 4 E2B` � host `Windows-AMD64` � llama.cpp `b10488`
Settings: `threads=24` `ngl=99` `ctx=2048`
`max_tokens=64` � warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 � `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 3792 | 206 / 371 | 10.6 / 10.9 | 880 / 1038 / 1038 | 94.1 |
| UD-Q2_K_XL | 2.24 | 3193 | 214 / 374 | 9.8 / 10.2 | 833 / 1014 / 1014 | 101.7 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.08x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation

UD-Q2_K_XL is worthwhile on this machine: decode improved from 94.1 to 101.7 tok/s
(1.08x), while disk usage fell by 0.73 GB (about 25%). Load time also dropped from
3792 to 3193 ms. I asked the same question through Q4 and Q2 servers; both answers
were coherent and technically equivalent, with only minor wording differences, so
the smaller quantization is the better default unless maximum answer quality is
more important than this modest speedup.
