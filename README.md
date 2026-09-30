# LLM-Inference-Engineering-Serving

**From GPU profiling and inference internals to quantized serving and capacity benchmarking.**

This project documents a practical exploration of **LLM inference systems**, focusing on what happens between loading a model onto a GPU and operating it as a measurable serving workload.

The work progresses from low-level inference measurements to serving-engine behavior, model quantization, and repeatable performance/capacity benchmarking.

![Architecture](architecture/inference-engineering-architecture.png)

---

## Overview

The project follows an end-to-end inference engineering workflow:

```text
GPU Profiling
     ↓
Inference Anatomy
     ↓
Serving Engine
     ↓
Quantization & Model Lock
     ↓
Benchmark & Capacity Analysis
```

The objective was not simply to run an LLM, but to understand and measure the systems-level factors that determine inference performance, memory usage, serving efficiency, and operating capacity.

---

## Engineering Workflow

### 1. GPU Inference Profiling

The first stage established a baseline by profiling LLM inference on a real GPU.

Focus areas included:

* GPU memory behavior
* inference latency
* token generation performance
* prompt/output characteristics
* relationship between workload and hardware utilization

This established the baseline for the experiments that followed.

---

### 2. Inference Anatomy

The second stage examined the mechanics behind autoregressive inference.

Key areas:

* Time to First Token (TTFT)
* Time Per Output Token (TPOT)
* KV-cache memory
* activation memory
* memory growth with sequence length
* static batching
* batch straggler effects
* paged KV-cache allocation

The goal was to connect observed serving behavior with the underlying memory and scheduling mechanisms.

---

### 3. Serving Engine Comparison

The next stage moved from manually controlled inference behavior to an optimized serving engine.

The same workload was evaluated across serving approaches to study:

* batching behavior
* concurrency
* throughput
* latency
* request scheduling
* engine-level efficiency

The experiments demonstrated how an inference engine can change system-level throughput without changing the underlying model workload.

---

### 4. Quantization & Model Lock

The serving configuration was then stabilized around a quantized model.

The locked serving model was:

```text
Qwen/Qwen2.5-1.5B-Instruct-AWQ
```

The quantized configuration was subjected to a functional smoke test covering tool-calling behavior and distractor prompts.

The final smoke test recorded:

```text
10 / 10 correct behaviours
Distractor call-free: yes
Regression detected: no
```

The purpose of this stage was to establish a reproducible model/configuration baseline before performance benchmarking.

---

### 5. Benchmark & Capacity Analysis

The final stage built a repeatable benchmark workflow around the locked serving configuration.

The benchmark evaluates:

* concurrency
* tokens/second
* TTFT p50
* TTFT p95
* end-to-end latency p95
* request errors
* sustainable operating capacity

Rather than selecting the highest raw throughput point, the analysis identifies the **capacity knee** under a defined latency target.

For the benchmark configuration, the target p95 latency was:

```text
2.0 seconds
```

The capacity analysis identified the highest tested concurrency that remained within that target and used it as the operating point for the capacity assessment.

---

## Key Results

Selected results from the experiments include:

| Area                 | Result                                       |
| -------------------- | -------------------------------------------- |
| Locked model         | Qwen/Qwen2.5-1.5B-Instruct-AWQ               |
| Quantization         | AWQ                                          |
| Smoke-test behaviour | 10/10                                        |
| Benchmark metric set | TTFT, latency, throughput, errors            |
| Capacity target      | 2.0 s p95                                    |
| Capacity analysis    | Knee-based rather than peak-throughput based |

Additional benchmark results and experiment summaries are provided in the `results/` and `docs/` directories.

---

## Architecture

```text
                    ┌──────────────────────┐
                    │     LLM Workload     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │   GPU Profiling      │
                    │ Latency / Throughput │
                    │ Memory Behavior      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Inference Anatomy   │
                    │ KV Cache / Batching  │
                    │ Scheduling Concepts  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Serving Engine       │
                    │ Request Scheduling   │
                    │ Continuous Batching  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Quantized Model      │
                    │ AWQ + Model Lock     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Benchmark Harness    │
                    │ Concurrency Sweep    │
                    │ TTFT / Latency       │
                    │ Throughput / Errors  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │ Capacity Analysis    │
                    │ SLO / Knee Detection │
                    └──────────────────────┘
```

---

## Technical Focus

The project focuses on practical LLM systems engineering across several layers:

**Inference**

* autoregressive generation
* TTFT and TPOT
* KV-cache behavior
* GPU memory

**Serving**

* batching
* request concurrency
* scheduling
* inference engines
* throughput/latency tradeoffs

**Optimization**

* model quantization
* memory efficiency
* serving configuration
* model validation

**Performance Engineering**

* benchmark harnesses
* percentile latency
* throughput measurement
* capacity analysis
* SLO-aware operating points

---

## What I Learned

The experiments reinforced several important inference-engineering concepts:

* GPU memory is not determined by model weights alone; KV-cache and runtime state can become significant.
* Increasing concurrency can improve throughput while simultaneously increasing latency.
* Serving-engine scheduling can materially affect utilization and throughput.
* Quantization is not only a memory optimization; the resulting model still needs functional validation.
* Peak throughput is not necessarily the correct production operating point.
* Capacity should be evaluated against an explicit latency target rather than throughput alone.

---

## Environment

The experiments were performed using GPU-based LLM inference with an OpenAI-compatible serving interface.

Core technologies included:

* Python
* PyTorch
* Transformers
* vLLM
* NVIDIA GPU inference
* AWQ quantization
* OpenAI-compatible APIs
* GPU/performance benchmarking

Exact experiment environments and configurations varied between stages.

---

## Repository Scope

This repository intentionally contains **documentation, architecture, selected results, and engineering observations rather than the complete laboratory implementation**.

The original experiments included notebooks, benchmark code, serving scripts, validation utilities, and infrastructure-specific configuration that are not included here.

The repository is therefore intended to communicate the **engineering process, system design, and measured outcomes** without publishing the complete internal implementation.

---

## Disclaimer

This project is a technical learning and portfolio project built around practical LLM inference and serving experiments.

Reported measurements are specific to the tested hardware, model configuration, workload, and serving environment and should not be interpreted as universal performance characteristics.
