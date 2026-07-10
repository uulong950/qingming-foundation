# Qingming Foundation

## 定位

**Qingming Foundation** 是 Qingming 系列已完成底层 primitive artifact 的中心地基仓库。

Qingming 系列既建立一套共同的衡量语言，也提供有力的性能证据。

本仓库只收录已经达到 Qingming-style 标准的 artifact：明确的 contract、独立验证、benchmark 边界、实现，以及仓库级可复现路径。

Qingming 不是排行榜，也不是 demo 集合。
它是一个地基层，用于将底层 primitive 的性能声明，转化为定义清晰、经过验证、可比较的新 baseline。

---

## Scope

Qingming Foundation 只收录已经完成的 Qingming-style primitive artifact。

本仓库不收录 demo、实验草稿、路线图、内部原型或普通实现。一个项目只有在达到 Qingming-style 的最低门槛后，才可以进入 Qingming Foundation：它必须定义 primitive benchmark 的结构，提供经过验证的实现，报告清晰的性能边界，并建立一个可作为后续工作地基的新 baseline。

一个被收录的 artifact 必须提供：

* 清晰的 primitive 定义
* 明确的 benchmark 结构
* 明确的 benchmark contract
* 实现
* 正确性或验证检查
* benchmark 结果
* 硬件环境
* build / run 路径
* 清晰的支持与不支持边界
* 仓库级可复现路径

进入 Foundation 的最低门槛不是“有实现”。

一个 Qingming Foundation 条目，应当是其所在 primitive 领域的地基推进型 artifact：它需要说明该 primitive 如何被结构化、应当如何被测量、实现推进到了什么边界，以及其他人如何复现或挑战这个结果。

未发布的、实验性的、内部的、尚未重构完成的，或不符合 Qingming-style 的项目，不会被收录在这里。

---

## 原则

```text
No contract, no comparison.
No correctness, no benchmark.
No validation, no foundation entry.
```

一个 Qingming-style artifact 应当明确说明：

* 数学对象
* 输入 / 输出 contract
* layout contract
* timing region
* 正确性检查
* 硬件环境
* benchmark 边界
* 支持 / 不支持的规模
* 复现路径

---

## Artifact Status Matrix

| Artifact              | Domain                              | Contract | Implementation | Benchmark | Validation | Status |
| --------------------- | ----------------------------------- | -------: | -------------: | --------: | ---------: | ------ |
| Qingming-G64-NTT-CUDA | Native Goldilocks/G64 STARK-LDE NTT |      Yes |    NVIDIA CUDA |       Yes |        Yes | Public |
| Qingming-G64-NTT      | Native Goldilocks/G64 STARK-LDE NTT |      Yes | AMD HIP / ROCm |       Yes |        Yes | Public |
| Qingming-STARK-G64 | Goldilocks/G64 STARK proving backend | Yes | AMD HIP / ROCm | Yes | Yes | Public |

`Validation` 表示该 artifact 已经通过作者侧的独立验证，例如正确性检查、CPU reference 对比、roundtrip 测试、固定 benchmark 运行，或 release-suite 检查。

外部第三方复现是未来单独记录的状态，不是初始进入 Qingming Foundation 的必要条件。

---

# 工作进展

## 1. G64-NTT Series

G64-NTT 是第一个已经完成的 Qingming primitive line。

它聚焦于 native Goldilocks/G64 STARK-LDE NTT，并围绕 field semantics、domain size、layout、timing region、correctness、hardware boundary 和 reproducibility 定义 benchmark contract。

---

### 1.1 Qingming-G64-NTT-CUDA

**Domain.**
NVIDIA CUDA 上的 native Goldilocks/G64 STARK-LDE NTT。

**What this work does.**
该 artifact 定义了两个 CUDA benchmark contract：

* `qingming_fast`：device-native tiled input 到 mapped output
* `qingming_standard`：natural-order input 到 standard materialized output

fast path 将 device layout 视为代数算子的一部分，并将 mapped output 作为面向 proof 的 fast result。standard path 提供 natural-order 兼容性，并包含 layout transition cost。

**Boundary.**
RTX4090-24G 覆盖 `domain_logn=20..30`。

代表性参考结果：

```text
domain_logn=24 fast      ~1.26 ms
domain_logn=27 fast      ~9.18 ms
domain_logn=30 fast      ~97 ms

domain_logn=30 standard  ~325 ms
```

`domain_logn=31` 在 RTX4090-24G 上属于容量受限边界。
`domain_logn>=32` 不支持。

**Repository.**
https://github.com/uulong950/qingming-g64-ntt-cuda

---

### 1.2 Qingming-G64-NTT

**Domain.**
AMD HIP / ROCm 上的 native Goldilocks/G64 STARK-LDE NTT。

**What this work does.**
该 artifact 提供了 AMD-native G64 NTT 实现，包含 fast mapped output、standard compatibility path、正确性检查、CPU reference validation 和 large-domain scaling。

**Boundary.**
RX 7900 XTX 在 Qingming G64/STARK-LDE benchmark contract 下达到 `domain_logn=27`，并报告 fast-path effective logical bandwidth 约为 `559 GB/s`。

**Repository.**
https://github.com/uulong950/qingming-g64-ntt

---
## 2. G64-STARK Series

G64-STARK 是位于 G64-NTT primitive layer 之上的下一条 Qingming foundation line。

它聚焦于一个可复现的 Goldilocks/G64 STARK proving 与 verification backend，并明确 proof-file boundary、deterministic protocol contract、standalone verifier 和仓库级复现路径。

---

### 2.1 Qingming-STARK-G64

**Domain.** AMD HIP / ROCm 上的 Goldilocks/G64 STARK backend。

**What this work does.** 该 artifact 提供了一个 source-visible、CLI-based 的 QSPG64 STARK proving 与 verification backend。它将官方 integration surface 有意保持为很小的边界：

* CLI prover
* QSPG64 `.qsp` proof file
* standalone verifier

该 backend 将 public inputs、statement digest、trace commitment、quotient commitment、Merkle openings、FRI proof material 和 local AIR verifier material 绑定到完整的 QSPG64 proof boundary 中。

**Contract.** 该 artifact 明确固定了 Goldilocks/G64 field contract、Poseidon2-G64 hash contract、compiled AIR profile、proof format、retained-Merkle-tree proof-builder 行为，以及 standalone verification checks。

**Boundary.** RX 7900 XTX 24GB 是已验证的 backend target。retained-tree proving matrix 已验证 `SCALE20` 到 `SCALE27`，其中 `SCALE27` 是 primary FAST XYZ benchmark。

**Repository.** https://github.com/uulong950/qingming-stark-g64

---

## Layering

Qingming Foundation 是 primitive foundation layer。

```text
Qingming = primitive contract + benchmark + boundary implementation
```

应用层项目可以基于 Qingming primitives 构建产品、PoC 或 workflow。

```text
Application layer = product / PoC / workflow built on Qingming capabilities
```

应用层结果不应替代 primitive-level benchmark contract。

---

## Goal

Qingming Foundation 旨在组织已经完成的 Qingming-series artifacts。这些 artifact 需要同时完成三件事：

1. 定义一个 primitive 的 benchmark 结构
2. 在接近硬件边界的位置实现该 primitive
3. 为后续工作提供一个经过独立验证的 baseline

目标不是声称孤立的性能数字。

目标是建立新的 ground truth。
