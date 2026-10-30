# Karp–Flatt Practice Lab — Results

Student ID: 230109011

## Hardware profile

- OS: Windows
- Python: 3.13
- `os.cpu_count()` = **16** logical cores

## Summary table (median of repeated runs)

| Experiment | Mode | T1 (s) | T2 (s) | T4 (s) | e(2) | e(4) | Architectural Diagnosis |
|---|---|---|---|---|---|---|---|
| A: Serial Tail | 16 chunks x 2M, 0.40s sleep (median of 4 runs) | 4.169 | 2.453 | 1.556 | 0.177 | 0.164 | Roughly constant e: fixed Amdahl serial path (the sleep) |
| B: Lock Contention | 2M locked increments (1 run) | 1.992 | 10.271 | 11.036 | 9.310 | 7.052 | Huge e and negative speedup: lock thrashing |
| C: Privatized State | 100M private increments (median of 3 runs) | 2.646 | 1.401 | 0.837 | 0.059 | 0.088 | Small e: scalability restored (E4 = 79%) |

Speedups from the medians: A: Sp(2) = 1.70x, Sp(4) = 2.68x. B: Sp(2) = 0.19x, Sp(4) = 0.18x. C: Sp(2) = 1.89x, Sp(4) = 3.16x.

## Raw runs

**Experiment A** (T1 / T2 / T4, then e(2), e(4), e(4)/e(2))

| Run | T1 | T2 | T4 | e(2) | e(4) | e(4)/e(2) |
|---|---|---|---|---|---|---|
| 1 | 4.170 | 2.536 | 1.503 | 0.217 | 0.147 | 0.68 |
| 2 | 4.152 | 2.396 | 1.600 | 0.154 | 0.180 | 1.17 |
| 3 | 4.168 | 2.381 | 1.533 | 0.143 | 0.157 | 1.10 |
| 4 | 4.359 | 2.510 | 1.578 | 0.152 | 0.149 | 0.98 |

**Experiment B** (1 run): T1 = 1.992, T2 = 10.271, T4 = 11.036, e(2) = 9.310, e(4) = 7.052.

**Experiment C** (100M increments)

| Run | T1 | T2 | T4 | e(2) | e(4) | E4 |
|---|---|---|---|---|---|---|
| 1 | 2.721 | 1.411 | 0.877 | 0.037 | 0.096 | 78% |
| 2 | 2.563 | 1.401 | 0.805 | 0.094 | 0.086 | 80% |
| 3 | 2.646 | 1.359 | 0.837 | 0.027 | 0.088 | 79% |

With the initial 20M increments, runs took under 0.7s and process start-up on Windows dominated: E4 was only 56–67% and e jumped between runs. The load was raised to 100M to make the start-up cost a small share of the runtime.

## Observations

- **A:** The 0.40s sleep is about 9.6% of T1, a fixed serial fraction. e(2) and e(4) stay at similar values (0.18 and 0.16 in the medians), so the loss is algorithmic and not a growing contention effect. The values are a bit higher than the handout's ~0.10–0.14, most likely because starting worker processes on Windows costs extra time. The first run gave a ratio of 0.68, later runs 0.98–1.17, so a single short run is noisy.
- **B:** Adding processes made the program about 5x slower than one process (Sp < 1), and e is far above 1.50. This is the lock-contention signature. One deviation from the handout: e(4) (7.05) came out lower than e(2) (9.31), so the "e(4) >> e(2)" expectation is not reproduced in this run. The slowdown already appears at P=2 and does not get much worse at P=4.
- **C:** With private counters and a single reduction at the end, P=2 reaches about 95% efficiency and P=4 about 79%. e stays below 0.1. It is slightly above the handout's < 0.05 target, mostly due to process start-up cost on Windows.

## Rubric self-check

| Check | Result |
|---|---|
| A: e(4)/e(2) in [0.80, 1.25] | PASS on the median (0.93, from e(4)=0.164 / e(2)=0.177) and on 3 of 4 runs |
| B: T4 > T1 and e(4) >> e(2) | T4 > T1 PASS; e(4) >> e(2) NOT met (7.05 vs 9.31) |
| C: T4 < T1 and E4 > 70% | PASS (T4 = 0.837 < T1 = 2.646, E4 = 79%) |
