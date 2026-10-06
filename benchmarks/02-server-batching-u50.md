# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` · `--parallel 4` · 15 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 4.00 of 4 slots (100%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 203 |

Highest sampled value was **4.00 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation
Peak batch width đo được đạt mức tối đa **4.00** (100% công suất), hoàn toàn khớp với giới hạn `effective concurrency` là 4 (do cấu hình `--parallel 4` của server). 

Điều này cung cấp bằng chứng rõ ràng nhất cho thấy scheduler của Continuous Batching đã hoạt động chính xác: nó liên tục gộp đủ 4 requests vào chung một bước decode để tận dụng tối đa tài nguyên. Đồng thời, chỉ số `requests_deferred` vọt lên mức **46** phản ánh đúng thực tế khi có 50 users cùng truy cập nhưng server chỉ có 4 slots, khiến 46 requests bị dồn vào hàng đợi (queue). Chính khoảng thời gian xếp hàng (queue time) này là thủ phạm chính làm độ trễ ở P95 tăng vọt trong bài test chịu tải.
