# Bonus - Context-length sweep (prefill cost)

Host `Windows-AMD64` · llama.cpp `b10488` ·
`threads=4` `ngl=99` · RAM 15.8 GB

| Prompt tokens | Prefill (tok/s) | TTFT contribution (ms) | vs linear scaling |
|:--|--:|--:|--:|
| 256 | 24.5 | 10449.0 | 1.00x |
| 1024 | 24.3 | 42139.9 | 1.01x |
| 2048 | 23.0 | 89043.5 | 1.07x |
| 4096 | 18.5 | 221405.4 | 1.32x |
| 8192 | 12.0 | 682666.7 | 2.04x |

At 8192 tokens, prefill costs **682666 ms** --
2.04x what linear scaling from the smallest point would predict. That excess
is attention's O(N^2) term becoming visible, and every millisecond of it lands in TTFT
before the user sees a single token.

Either way, this is the number to remember when someone proposes stuffing more retrieved
context into a RAG prompt "because the context window allows it". Prefill is paid in full,
on every request, before the first token appears.

## Your finding
Ở khoảng 4096-8192 tokens, prefill bắt đầu bẻ cong rõ rệt theo đường cong quadratic (vs linear scaling = 2.04x ở 8192 tokens), cho thấy chi phí của O(N^2) Attention đang áp đảo O(N) MLP. Điều này cho thấy pipeline RAG chỉ nên nhét dưới 4000 tokens context (khoảng 3-4 chunks) để tránh bị nghẽn cổ chai ở khâu đọc hiểu của LLM.
