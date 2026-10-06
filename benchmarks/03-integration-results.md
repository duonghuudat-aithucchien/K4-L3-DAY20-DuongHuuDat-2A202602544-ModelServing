# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.1 | 21273.6 | 21273.7 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 15738.5 | 15738.7 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.2 | 15785.9 | 15786.2 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **17599.3** · total **17599.5**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real
- **N16 (Data Pipeline / Document Embedding):** Stubbed (Dùng bộ docs tĩnh `TOY_DOCS`).
- **N17 (Embedding Model/Endpoint):** Stubbed (Trả về 0.0 ms do không gọi embedding model thật).
- **N18 (Vector Database / Index):** Stubbed (Dùng keyword overlap đơn giản trên RAM thay vì Vector DB thật).
- **N19 (LLM Inference Endpoint):** **Real** (Thực sự gọi vào `llama-server` đang chạy ở `localhost:8080`).

**Dominant stage có đúng như kỳ vọng không?** 
Hoàn toàn đúng như kỳ vọng. Do N16-N18 đều bị stub và chạy fake ngay trên RAM (tốn 0.1ms), nên toàn bộ thời gian của pipeline đều dồn vào khâu N19 (LLM), chiếm 100% tổng thời gian (gần 18 giây mỗi câu).

**Làm sao để giảm một nửa latency?**
Để giảm một nửa latency, bắt buộc phải tối ưu stage **`llm`** vì nó chiếm 100% thời gian. Các phương án khả thi nhất:
1. **Chuyển sang dùng GPU** (thay vì CPU) để tăng tốc độ decode.
2. Dùng một model nhỏ hơn (như Qwen 0.8B).
3. Áp dụng Semantic Caching hoặc Prompt Caching để bỏ qua bước tính toán nếu câu hỏi bị lặp lại. Tối ưu khâu retrieve hay embed lúc này là hoàn toàn vô nghĩa.
