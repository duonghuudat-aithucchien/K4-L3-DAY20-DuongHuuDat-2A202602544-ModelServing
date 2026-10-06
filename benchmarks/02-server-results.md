# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=4` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 4 | 0.14 | 28000 | 28000 | 28000 | 4.0 | 0.0% |
| 50 | 4 | 0.15 | 27000 | 27000 | 27000 | 4.0 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.03x** (21% of linear) |
| P95 latency | **0.96x** |
| Effective concurrency at 50 users | 4.0 vs `--parallel 4` slots (occupancy/slot ratio 1.00) |

**Saturated.** Throughput delivered only 1.03x for 5x the offered load, and effective concurrency (4.0) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

P95 grew no faster than throughput (0.96x vs 1.03x), so this server still has headroom at 50 users.

> **Small sample.** Only 4 requests completed in the
> shorter run, so these percentiles are indicative rather than solid. Note also that
> locust averages only *completed* requests: when the run ends with requests still
> queued, effective concurrency is an **under**-estimate. Trust the throughput-scaling
> row over the concurrency row here, and run longer (`-t 3m`) if you want firmer numbers.

## Your reading
Server của tôi đã đạt **điểm bão hoà (Saturated)** ở mức dưới 50 users. Bằng chứng rõ ràng nhất là: khi tăng số lượng user gấp 5 lần (từ 10 lên 50), tốc độ xử lý thực tế (Throughput delivered) gần như đi ngang, chỉ tăng **1.03x**. Đồng thời, chỉ số Effective Concurrency đạt chính xác **4.0** - lấp đầy hoàn toàn 4 slot decode (`--parallel 4`) được cấu hình. Do máy chạy chậm và bài test ngắt ở 60s, Locust chưa kịp ghi nhận những request phải xếp hàng, nhưng việc throughput đi ngang chứng tỏ các request phụ thêm đã biến thành queue time thay vì compute time.

Để nâng cao Goodput ở một mức SLO nhất định, thay đổi đáng giá nhất là **nâng cấp phần cứng có băng thông bộ nhớ (memory bandwidth) lớn hơn**, hoặc chạy thêm nhiều server (horizontal scaling). Việc tăng tham số `--parallel` trên máy tính này không có ý nghĩa vì máy đã chạm trần băng thông RAM ngay từ 1 thread (đã chứng minh ở phần Tune thread count), nên việc nhét thêm request vào chung một batch cũng không làm throughput tổng thể tăng lên được.
