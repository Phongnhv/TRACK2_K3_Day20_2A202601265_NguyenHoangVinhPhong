# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` � llama.cpp `b10488` �
retrieval backend: **keyword overlap** � 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 2672.8 | 2672.9 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 2502.9 | 2503.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 2476.6 | 2476.6 |

Mean per stage (ms): embed **0.0** � retrieve **0.0** �
llm **2550.8** � total **2550.8**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, which removes the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real

N16 is a localhost-only serving seam (stub), N17 is an in-memory list (stub), N18
is represented by the toy dictionary rather than a lakehouse (stub), and N19 uses
`TOY_DOCS` plus keyword-overlap retrieval rather than a vector index (stub). N20
`llama-server` is real. The LLM stage taking 100% of measured time was expected;
to halve latency I would reduce generated tokens/context or use Q2, because embed
and retrieve are effectively free in this toy pipeline.
