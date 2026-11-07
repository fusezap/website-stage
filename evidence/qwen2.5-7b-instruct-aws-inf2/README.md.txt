# I78 confirm: inf2.8xlarge, Qwen2.5-7B bf16 TP=2, 1 Nov 2026

main 5a870a6 (prefill size by the oldest pending prompt, 128,512 default on NEURON, cost 128:53 / 512:195) + the
aws-multi-ip runner fix (27e3f04). `nki_n8_prefill_inf2.sh --pfk2`, 0.8 h. Two earlier launches timed out waiting for sshd
(the operator's router egresses from two public IPs; <redacted> each), fixed by aws-multi-ip.

Served via `fusezap serve --tp 2`, Validate perf (1 repeat, 8 requests per point); TTFT p50 / p99, ms:

| prompt | users | a: 128 rows only | b: 128,512, per-prompt rule | b on 1 Nov (burst rule) |
|---|---|---|---|---|
| 128 | 1 | 54.7 / 54.9 | 55.4 / 57.4 | 55.9 / 57.1 |
| 128 | 4 | 141.5 / 237.9 | 141.9 / 230.3 | 204.1 / 234.5 |
| 40 | 1 | 54.1 / 61.5 | 55.1 / 55.5 | 55.0 / 55.3 |
| 40 | 4 | 86.4 / 114.8 | 87.4 / 114.4 | 88.6 / 117.5 |
| 200 | 1 | 110.9 / 112.7 | 109.7 / 113.1 | 110.0 / 114.3 |
| 200 | 4 | 302.1 / 505.1 | 296.2 / 494.7 | 303.9 / 464.8 |
| 512 | 1 | 226.6 / 229.1 | 199.3 / 200.6 | 199.9 / 201.1 |

TPOT p50 38.4-42.7 ms both; c4 short tok/s 98.7 (a) / 99.3 (b); 0 errors; Validate PROMOTE on perf and accuracy for both
(top-1 0.9987, greedy exact 0.9668, both: the 1 Nov greedy difference came with the bundled bursts).
Chunk check: the last prompt token's logits bit-identical across auto / all-128 / one-512 and v1 at 200, 400, 512 tokens.
Job summary: RECOMMEND (measured cost 128:52, 512:194; the table's 53 / 195 kept).
