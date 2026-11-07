# N7 serve on trn1 (27 Oct 2026, <redacted>): fusezap serve before/after

Branch nki-n7-serve 5085f75; head v2:1024:1. validate short (128 in / 128 out), 2 repeats; every server passed the gate;
probe texts after vs before 16/16 identical; PROMOTE; accuracy top-1 1.0, greedy exact 1.0.

| served, trn1 fp32 1 core | TPOT c1 | tok/s c1 | TPOT c4 | tok/s c4 [CI] | TTFT p99 c4 |
|---|---|---|---|---|---|
| before (LIFO blocks, no prefill-first, engine in process) | 16.77 ms | 58.7 | 20.04 ms | 190.7 [188.6, 192.8] | 243 ms |
| after (lowest-first blocks, prefill-first 4, engine process) | **14.88 ms** | **66.3** | 19.55 ms | 198.6 [186.1, 211.2] | 147 ms |
| after, engine in process | 14.98 ms | 65.9 | 19.65 ms | 197.5 [196.2, 198.8] | 149 ms |
| AWS vLLM + vllm-neuron (N6b) | 16.30 ms | 61.1 | 16.42 ms | 238.4 | 94 ms |

In process floors: 512-slot prefix c1 14.76 ms, c4 17.04 ms / 234.7 tok/s; 1024-slot prefix c1 16.61 ms, c4 18.81 ms / 212.7 tok/s.
Served c4 has to run at the 1024 prefix (4 x 256 slots), so its floor is ~213 tok/s; the rest is prefill interleaving.
The engine process makes no measurable difference on trn1.
