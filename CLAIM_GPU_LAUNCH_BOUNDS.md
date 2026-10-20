# RTX 3070 GPU-kernel bounty candidate: launch occupancy

Base: `4bde6441f8f39764f5ab9b64a60e4cb613f72014` from the upstream repository.
License: `LICENSE` (MIT). The only code change is in `gpu/mine.py`:

```diff
-extern "C" __global__ void __launch_bounds__(B, 2) ring_mine(
+extern "C" __global__ void __launch_bounds__(B, 1) ring_mine(
```

This relaxes the compiler's minimum resident-block requirement. On the local RTX 4060 Laptop GPU, the compiled kernel uses 76 registers per thread instead of 40. It does not change the BRAM-ring function, tables, nonce handling, or benchmark entry point.

## Correctness

`python gpu/mine.py vectors | python host/check_vectors.py` reports **PASS: 8/8 vectors bit-exact** for digest, fold and full state. `python gpu/mine.py` independently reports 8/8 golden digests and folds.

A separate epoch check with `BRAM_EPOCH=codex-check-20261020` and nonce 42 compared the candidate GPU output with `spec.run(seed, nonce)`: complete state and fold matched. SHA-256 of the resulting state was `8698585fd77d98079999091379e9e7500af7c6c3328489cebfd5b8e21b85cd13`.

The three frozen specification files are unchanged in the commit. Their Git blob SHA-256 hashes match `BOUNTY.md`:

- `spec.py`: `5596327c5fc89b4d87eaf64df0b25e8430a9b25247ebae4c3e1037c19cd2f292`
- `generate.py`: `20f78b63ff10f7b4fee681355190e91eaaeb527a733d2701d928eb0ac6534dab`
- `vectors.txt`: `d406e2319f21e641dc77fcf79693c88062cc312cd1d75f0c286fa4ba85cf88f1`

## Local benchmark, 20 October 2026

Hardware: NVIDIA GeForce RTX 4060 Laptop GPU (8,188 MiB), driver 617.42. CuPy 14.2.0, CUDA runtime 12.9. On Windows, five interleaved 920-nonce runs, 460 blocks per run, with one warmup for each implementation:

| Run | Original H/s | Candidate H/s |
|---:|---:|---:|
| 1 | 897.4 | 1206.8 |
| 2 | 922.9 | 1218.0 |
| 3 | 947.5 | 1223.6 |
| 4 | 917.4 | 1218.2 |
| 5 | 950.9 | 1221.3 |
| Mean | 927.2 | 1217.6 |

Local gain: **31.3%**. A separate invocation of the repository's standard benchmark (`python gpu/mine.py`) reported 1,158 H/s for 920 nonces at 460 blocks; the baseline invocation reported 923 H/s. The repeated interleaved comparison is the stronger local estimate.

This is **not yet an award result**. The rules require the organizer to measure the candidate against the current champion on the reference RTX 3070, back to back. Architecture and software differences may change the result. No payment has been claimed or received.

## Reproduce

Install the dependencies in `README.md` and run:

```sh
python3 gpu/mine.py vectors | python3 host/check_vectors.py
python3 gpu/mine.py
```

The shipped `gpu/mine.py` remains the benchmark entry point, including its `blocks=460` line. Compare with the parent commit under the same protocol and machine. The organizer's fresh epoch seed can be supplied with `BRAM_EPOCH=<seed>` after regenerating artifacts in a scratch clone, as described in `BOUNTY.md`.
