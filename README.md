# Qingming Foundation

## Positioning

**Qingming Foundation** is the central foundation repository for completed Qingming-series primitive artifacts.

The Qingming series establishes both a common language for measurement and compelling performance evidence.

It collects only artifacts that have reached a Qingming-style level: explicit contract, independent validation, benchmark boundary, implementation, and repository-level reproducibility.

Qingming is not a leaderboard and not a collection of demos.
It is a foundation layer for turning low-level primitive performance claims into clearly defined, validated, and comparable new baselines.

---

## Scope

Qingming Foundation only lists completed Qingming-style primitive artifacts.

This repository does not list demos, experiments, roadmaps, internal prototypes, or ordinary implementations. A project may enter Qingming Foundation only after it reaches the minimum Qingming-style bar: it must define the structure of a primitive benchmark, provide a validated implementation, report a clear performance boundary, and establish a new baseline that can be used as a foundation for future work.

A listed artifact must provide:

* a clear primitive definition
* an explicit benchmark structure
* an explicit benchmark contract
* an implementation
* correctness or validation checks
* benchmark results
* hardware environment
* build / run path
* clear supported and unsupported boundaries
* repository-level reproducibility

The minimum bar for inclusion is not implementation alone.

A Qingming Foundation entry should be a foundation-moving artifact in its own primitive domain: it should clarify how the primitive is structured, how it should be measured, what boundary the implementation reaches, and how others can reproduce or challenge the result.

Unreleased, experimental, internal, not-yet-restructured, or non-Qingming-style projects are not listed here.

---

## Principles

```text
No contract, no comparison.
No correctness, no benchmark.
No validation, no foundation entry.
```

A Qingming-style artifact should make the following explicit:

* mathematical object
* input / output contract
* layout contract
* timing region
* correctness checks
* hardware environment
* benchmark boundary
* supported / unsupported scale
* reproduction path

---

## Artifact Status Matrix

| Artifact              | Domain                              | Contract | Implementation | Benchmark | Validation | Status |
| --------------------- | ----------------------------------- | -------: | -------------: | --------: | ---------: | ------ |
| Qingming-G64-NTT-CUDA | Native Goldilocks/G64 STARK-LDE NTT |      Yes |    NVIDIA CUDA |       Yes |        Yes | Public |
| Qingming-G64-NTT      | Native Goldilocks/G64 STARK-LDE NTT |      Yes | AMD HIP / ROCm |       Yes |        Yes | Public |

`Validation` means that the artifact has passed author-side independent validation, such as correctness checks, CPU reference comparison, roundtrip tests, fixed benchmark runs, or release-suite checks.

External third-party reproduction is a separate future status and is not required for initial inclusion in Qingming Foundation.

---

# Work Progress

## 1. G64-NTT Series

G64-NTT is the first completed Qingming primitive line.

It focuses on native Goldilocks/G64 STARK-LDE NTT and defines benchmark contracts around field semantics, domain size, layout, timing region, correctness, hardware boundary, and reproducibility.

---

### 1.1 Qingming-G64-NTT-CUDA

**Domain.**
Native Goldilocks/G64 STARK-LDE NTT on NVIDIA CUDA.

**What this work does.**
This artifact defines two CUDA benchmark contracts:

* `qingming_fast`: device-native tiled input to mapped output
* `qingming_standard`: natural-order input to standard materialized output

The fast path treats device layout as part of the algebraic operator and exposes mapped output as the proof-facing result. The standard path provides natural-order compatibility and includes layout transition cost.

**Boundary.**
RTX4090-24G covers `domain_logn=20..30`.

Representative reference points:

```text
domain_logn=24 fast      ~1.26 ms
domain_logn=27 fast      ~9.18 ms
domain_logn=30 fast      ~97 ms

domain_logn=30 standard  ~325 ms
```

`domain_logn=31` is capacity-gated on RTX4090-24G.
`domain_logn>=32` is unsupported.

**Repository.**
https://github.com/uulong950/qingming-g64-ntt-cuda

---

### 1.2 Qingming-G64-NTT

**Domain.**
Native Goldilocks/G64 STARK-LDE NTT on AMD HIP / ROCm.

**What this work does.**
This artifact provides an AMD-native G64 NTT implementation with fast mapped output, standard compatibility path, correctness checks, CPU reference validation, and large-domain scaling.

**Boundary.**
RX 7900 XTX reaches `domain_logn=27` with reported fast-path effective logical bandwidth around `559 GB/s` under the Qingming G64/STARK-LDE benchmark contract.

**Repository.**
https://github.com/uulong950/qingming-g64-ntt

---

## Layering

Qingming Foundation is the primitive foundation layer.

```text
Qingming = primitive contract + benchmark + boundary implementation
```

Application-layer projects may use Qingming primitives to build products, PoCs, or workflows.

```text
Application layer = product / PoC / workflow built on Qingming capabilities
```

Application-layer results should not replace primitive-level benchmark contracts.

---

## Goal

Qingming Foundation aims to organize completed Qingming-series artifacts that do three things at once:

1. define the benchmark structure of a primitive
2. implement the primitive near the hardware boundary
3. provide an independently validated baseline for future work

The goal is not to claim isolated performance numbers.

The goal is to establish new ground truth.
