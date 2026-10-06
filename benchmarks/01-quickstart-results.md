# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=4` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 14581 | 3977 / 6257 | 158.0 / 165.7 | 13938 / 16167 / 16167 | 6.3 |
| UD-Q2_K_XL | 2.24 | 19124 | 5884 / 17460 | 1776.8 / 2432.1 | 117590 / 160490 / 160490 | 0.6 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **10.50x SLOWER** than `UD-Q4_K_XL` here, despite being 0.73 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Your observation
Dựa trên kết quả đo đạc trên máy của tôi:
1. **Kích thước**: Bản `UD-Q2_K_XL` (2.24 GB) nhẹ hơn bản `UD-Q4_K_XL` (2.97 GB) khoảng 0.73 GB.
2. **Tốc độ**: Bản 2-bit (0.6 tok/s) chạy **chậm hơn tới 10.5 lần** so với bản 4-bit (6.3 tok/s), đồng thời thời gian chờ token đầu tiên (TTFT) và tổng thời gian (E2E) cũng tăng vọt.
3. **Kết luận**: Trên máy của tôi (Compute-limited - chạy thuần CPU), chi phí xử lý giải nén (dequantize) cho định dạng 2-bit lớn hơn rất nhiều so với băng thông bộ nhớ tiết kiệm được. Do đó, tốc độ của bản 2-bit bị suy giảm quá mức (chỉ còn 0.6 tok/s - không thể sử dụng thực tế). **Hoàn toàn không đáng dùng bản 2-bit trên máy này**, bản 4-bit vẫn là sự lựa chọn duy nhất và tốt nhất để cân bằng giữa RAM và tốc độ.
