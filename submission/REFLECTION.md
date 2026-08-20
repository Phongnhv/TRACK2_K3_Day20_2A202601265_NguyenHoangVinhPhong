# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** _Nguyen Hoang Vinh Phong_
**Cohort:** _A20-K3_
**Ngày submit:** _2026-08-20_

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** _Windows 11_
- **CPU:** _13th Gen Intel(R) Core(TM) i9-13900HX_
- **Cores:** _24 physical / 32 logical_
- **CPU extensions:** _AVX2_
- **RAM:** _15.7 GB_
- **Accelerator:** _NVIDIA GeForce RTX 4060 Laptop GPU, CUDA (Vulkan also detected)_
- **llama.cpp asset đã tải:** _llama-b10488-bin-win-cuda-12.4-x64.zip_
- **Model đã dùng:** _Gemma 4 E2B_ (`LAB_MODEL=`_gemma4-e2b_)
- **Quantization:** _UD-Q4_K_XL_ + _UD-Q2_K_XL_ (từ `models/active.json`)

**Chạy ở đâu:** _laptop của tôi_
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Setup chạy local với Python 3.12, runtime CUDA prebuilt b10488 và hai model GGUF. Probe
lần đầu gặp lỗi mã hóa console Windows; đặt `PYTHONIOENCODING=utf-8` rồi chạy lại thành
công. Không cần cloud hay compiler.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 3792 | 206 / 371 | 10.6 / 10.9 | 880 / 1038 / 1038 | 94.1 |
| UD-Q2_K_XL | 2.24 | 3193 | 214 / 374 | 9.8 / 10.2 | 833 / 1014 / 1014 | 101.7 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 decode nhanh hơn 1.08x (101.7 so với 94.1 tok/s), nhỏ hơn 0.73 GB và load nhanh hơn.
Tôi hỏi cùng một prompt trên hai server; cả hai trả lời mạch lạc, khác biệt chỉ ở cách diễn đạt.
Vì vậy Q2 đáng dùng trên máy này trừ khi ưu tiên chất lượng tối đa.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 3.30 | 2000 | 3200 | 4200 | 6.9 | 0.0% |
| 50 | 3.34 | 13000 | 15000 | 16000 | 41.4 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** _1.01×_
- **P95 tăng:** _4.69×_
- **Effective concurrency ở 50 users:** _41.4_ so với `--parallel` = _4_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _3.94_ / _4_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hòa ở mức không quá 50 users: RPS chỉ tăng 1.01x (3.30 lên 3.34) nhưng P95 tăng
4.69x. Effective concurrency 41.4 vượt xa 4 slots, đồng thời có 45 request deferred và
3.94/4 busy slots; vì vậy latency thêm chủ yếu là queue time. Tôi sẽ dùng Q2 và giảm output/context
trước, rồi mới thử tăng `--parallel` để nâng goodput@SLO mà không đẩy queue lên thêm.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost-only seam | stub |
| N17 Data pipeline | in-memory toy list | stub |
| N18 Lakehouse | toy dictionary | stub |
| N19 Vector + features | TOY_DOCS + keyword overlap | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: _0.0 ms_
- retrieve: _0.0 ms_
- llm: _2550.8 ms_
- **stage chiếm nhiều nhất:** _llm_ (_100%_ của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck đúng như kỳ vọng: trung bình 2550.8 ms, trong khi embed và retrieve đều 0 ms
vì dùng fallback toy. Để giảm latency 2x, tôi sẽ giảm số token/context hoặc dùng Q2; tối ưu retrieval
không giúp đáng kể ở pipeline hiện tại.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _đổi quantization từ UD-Q4_K_XL sang UD-Q2_K_XL_

```
before:  94.1 tok/s (UD-Q4_K_XL)
after:   101.7 tok/s (UD-Q2_K_XL)
speedup: 1.08×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Q2 giảm lượng dữ liệu trọng số phải đọc trong mỗi bước decode nên giảm áp lực memory bandwidth;
điều này phù hợp với TPOT giảm từ 10.6 xuống 9.8 ms. Trên máy này phần lớn layer chạy trên RTX
4060 (`ngl=99`), nên thread sweep CPU gần như phẳng (chỉ 1.01x), trong khi quantization vẫn tạo
ra khác biệt đo được.

Đổi lại, Q2 có thể giảm chất lượng biểu diễn vì dùng ít bit hơn. Tôi đã hỏi cùng một câu trên hai
server và nhận được câu trả lời mạch lạc, gần tương đương, nên speed/size benefit đáng đánh đổi cho
serving thông thường; tác vụ cần độ chính xác cao vẫn nên dùng Q4.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [x] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
