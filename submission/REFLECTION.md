# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** _Dương Hữu Đạt_
**MSSV:** _2A202602544_
**Cohort:** _A20-K4_
**Ngày submit:** _6/10/2026_

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** _Windows 11_
- **CPU:** _Intel(R) Core(TM) i7-10510U CPU @ 1.80GHz_
- **Cores:** _4 physical / 8 logical_
- **CPU extensions:** _AVX2_
- **RAM:** _15.8 GB_
- **Accelerator:** _Vulkan_
- **llama.cpp asset đã tải:** _llama-b10488-bin-win-vulkan-x64.zip_
- **Model đã dùng:** _Gemma 4 E2B_ (`LAB_MODEL=`_gemma4-e2b_)
- **Quantization:** _UD-Q4_K_XL_ + _UD-Q2_K_XL_ (từ `models/active.json`)

**Chạy ở đâu:** _laptop của tôi_
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

_Phải chạy thêm flag `--vulkan` để dùng card on-board. Đặc biệt trên Windows, tôi phải vá file `serve.py` thay lệnh `os.execv` bằng `subprocess.run` do hàm gốc gây lỗi Unicode ở thư mục chứa khoảng trắng._

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 14581 | 3977 / 6257 | 158.0 / 165.7 | 13938 / 16167 / 16167 | 6.3 |
| UD-Q2_K_XL | 2.24 | 19124 | 5884 / 17460 | 1776.8 / 2432.1 | 117590 / 160490 / 160490 | 0.6 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

_Bản 2-bit chậm hơn bản 4-bit tới 10.5 lần (0.6 vs 6.3 tok/s), hoàn toàn không đáng dùng vì trên máy này chi phí giải nén dequantize trên CPU quá lớn so với mức tiết kiệm băng thông bộ nhớ. Chất lượng câu trả lời của 2-bit cũng kém và giật cục._

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.14 | 28000 | 28000 | 28000 | 4.0 | 0.0% |
| 50 | 0.15 | 27000 | 27000 | 27000 | 4.0 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** _1.03×_
- **P95 tăng:** _0.96×_
- **Effective concurrency ở 50 users:** _4.0_ so với `--parallel` = _4_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _4.00_ / _4_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

_Server bão hoà ở mức dưới 50 users vì throughput đi ngang (tăng 1.03x dù tải tăng 5x) và effective concurrency kịch trần 4.0. Độ trễ tăng thêm hoàn toàn là queue time vì server không còn rảnh lúc nào để xử lý. Để tăng goodput, đổi knob sang phần cứng mạnh hơn (hoặc cluster thêm node) vì --parallel đã kịch trần khả năng của RAM._

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | N/A | stub |
| N17 Data pipeline | N/A | stub |
| N18 Lakehouse | N/A | stub |
| N19 Vector + features | N/A | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: _0.0 ms_
- retrieve: _0.1 ms_
- llm: _17599.3 ms_
- **stage chiếm nhiều nhất:** _llm_ (_100%_ của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

_Bottleneck hoàn toàn nằm ở LLM, đúng như kỳ vọng vì các khâu khác đã bị stub. Để giảm latency 2x, bắt buộc phải nâng cấp phần cứng cho server inference, dùng model bé hơn (Qwen 0.8B) hoặc tối ưu cache để LLM không phải tính lại._

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _Hạ -t từ 4 xuống 16_

```
before:  6.3 tok/s
after:   4.7 tok/s
speedup: 0.74×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Dù tăng số luồng CPU từ 4 lên 16, tốc độ lại bị giảm. Lý do là CPU i7-10510U chỉ có 4 nhân vật lý, việc ép nó chạy 16 luồng sinh ra chi phí context switch khổng lồ. Hơn nữa, memory bandwidth bị kịch trần từ sớm, nên số core có tăng lên thì CPU cũng chỉ ngồi đợi RAM trả data về (memory bound). Bằng chứng là tốc độ decode giảm 26% khi cố nhồi thêm thread._

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _B2 sweep-ctx_

**Numbers:**

```
before:  24.5 tok/s (ở 256 tokens)
after:   12.0 tok/s (ở 8192 tokens)
speedup: 0.49× (hiệu năng giảm mạnh do độ dài O(N^2))
```

**Điều này nói lên gì mà deck chưa nói:**

Ở context length ngắn (< 2048), tốc độ prefill đi ngang (linear), nhưng từ 4096 đến 8192, prefill time tăng gấp đôi so với linear scaling do hàm O(N^2) của thuật toán attention bắt đầu chiếm ưu thế, chứng minh rằng không bao giờ nên lạm dụng ngữ cảnh quá lớn cho RAG vì nó sẽ kéo sập TTFT của server.

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
- [x] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [x] Repo GitHub ở chế độ **public**
- [x] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

Có sử dụng Gemini (Google Antigravity AI Assistant) trong suốt quá trình làm bài. Các việc AI đã làm:
- Sửa lỗi đường dẫn `subprocess` cho file `serve.py` và các script đo đạc để chạy trên Windows
- Chạy các lệnh benchmark (tuning, load testing, sweep-ctx) và phân tích các log
- Phân tích số liệu hiệu năng và hỗ trợ viết các nhận xét trong file `benchmarks/*.md` và `REFLECTION.md`
