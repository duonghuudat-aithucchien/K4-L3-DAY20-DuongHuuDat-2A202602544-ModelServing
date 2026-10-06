# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **4 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 3.8 | 79% |
| 2 | 4.5 | 96% |
| 4 | 4.1 | 87% |
| 8 | 4.5 | 94% |
| 16 | 4.7 | 100% |

**Best**: `-t 16` at 4.7 tok/s
**Slowest tested**: `-t 1` at 3.8 tok/s (1.26x spread)
**Against the physical-core default** (`-t 4`, 4.1 tok/s): 1.15x

Use this in your run:

```bash
LAB_N_THREADS=16 make bench
```

## Your explanation

Đường cong hiệu năng gần như đi ngang từ 2 luồng (4.5 tok/s) lên 16 luồng (4.7 tok/s). Điểm tối ưu (-t 16) không mang lại sự khác biệt đáng kể so với -t 2. Điều này cho thấy hệ thống (Intel i7-10510U, Vulkan GPU Offload) đang bị giới hạn (bottleneck) ở băng thông bộ nhớ (memory bandwidth) và I/O của GPU, chứ không phải do thiếu khả năng tính toán của CPU. Việc nhồi thêm thread (lên 16) chỉ làm tăng overhead quản lý thread mà không giải quyết được nút thắt cổ chai về dữ liệu, nên tốc độ không cải thiện thêm.
