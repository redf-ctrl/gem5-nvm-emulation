# Cache Memory and NVM Emulation in Ruby Gem5

## Overview
This repository contains source code, log files and simulation report for hardware emulation and cache profiling of Phase Change Memory (PCM) within the Gem5 simulator. 

The project modifies the Ruby memory model (MESI Two-Level protocol) to benchmark the performance degradation of emulated PCM against a baseline DDR3 DRAM architecture. 

## Technical Stack
* **Simulator:** Gem5 (Ruby Memory Model, MESI_Two_Level)
* **Languages:** C++ (Cache logic), Python (Hardware timing)
* **Workload:** SPEC CPU 2017 (`505.mcf_r` - Route planning)

## Implementation

### C++ Cache Profiling
Modified `CacheMemory.cc` and `CacheMemory.hh` to extract windowed performance metrics without altering standard execution flow.
* Intercepted `profileDemandHit()` and `profileDemandMiss()` to track access statistics.
* Restricted profiling to a 1,000,000 clock-cycle observation window (2.75B to 3.25B ticks).
* Leveraged `registerDumpCallback` to dump L1-I metrics at simulation teardown.

### Python Hardware Emulation
Defined a custom memory profile in `DRAMInterface.py` to simulate the physical constraints of PCM, specifically addressing its asymmetric read/write latency compared to volatile DRAM.
* **Baseline:** `DDR3_2133_8x8`
* **Read Penalty (3x):** Scaled `tRCD`, `tCL`, `tRP`, and `tRAS` (e.g., `tRCD` modified from 13.09ns to 39.27ns).
* **Write Penalty (10x):** Scaled `tWR` and `tRTP` (e.g., `tWR` modified from 15.0ns to 150.0ns).

## Results

Simulations ran with a 1,000,000 instruction fast-forward followed by a 10,000,000 instruction execution limit. 

### L1-I Cache Statistics (1M Cycle Window)
| Metric | Value |
| :--- | :--- |
| Window Hits | 328,607 |
| Window Misses | 1,294 |
| **Miss Rate** | **0.3922%** |

### Execution Cycle Latency
| Memory Topology | Total Execution Cycles |
| :--- | :--- |
| Baseline (`DDR3_2133`) | 43,216,439 |
| Emulated PCM (`NVM_PCM`) | 46,523,040 |
| **Cycle Penalty** | **+3,306,601** |
| **Performance Degradation**| **7.65%** |

## Analysis
Transitioning from DDR3 to PCM introduced a 7.65% performance degradation. While the L1-I cache maintained a stable ~0.39% miss rate, the memory-bound `505.mcf_r` workload forced frequent main memory fetches upon L2 misses. The 3x/10x read/write latency scaling of the PCM model amplified the cost of these L2 misses, extending CPU stall durations and adding 3.3 million cycles to total execution time.

