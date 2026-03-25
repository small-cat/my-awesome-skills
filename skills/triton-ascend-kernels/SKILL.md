---
name: triton-ascend-kernels
description: "Provide guidance for writing and benchmarking optimized triton kernels for ascend npus(910B, 910C). Support models like Qwen, Deepseek and so on. The triton kernels could be integrated into vllm inference engine or used separately with torch. This skill should be used when the user asks to \"write/optimize triton kernels\"."
---

# Triton Kernel for Ascend NPU

The skill provides patterns and guidance for developing and optimizing triton kernels targeting Ascend NPUs(910B, 910C) for models like Qwen, Deepseek and so on. 

## When This Skill Applies
Use this skill when:
- benchmarking kernel performance against baseline implementations
- writing new triton kernels for ascend npus
- optimizing existing triton kernels for ascend npus
- debugging triton kernel performance issues on ascend npus

## Workflow
**Step 1**:
Analyze the existing code for optimizing the triton kernel or understand the user's requirement for writing a new kernel.

**Step 2**:
Read the [Ascend NPU Hardware Spec](./reference/ascend-npu-hardware-spec.md) to understand the hardware architecture of ascend npus.

**Step 3**:
Read the [memory bound and compute bound analysis](./reference/memory-bound-vs-compute-bound.md) to compute the maximum compute capability and memory bandwidth, determine whether the kernel is compute-bound or memory-bound, present the detailed computation process and conclusion, and await user confirmation.
 
**Step 4**:
Based on the analysis from step 3, design the triton kernel architecture and show the details of the design and wait for the user's confirmation.

**Step 5**:
Read the [kernel benchmark guidance](./reference/kernel-benchmark-guidance.md) and implement the benchmarking test cases.

**Step 6**:
Implement the triton kernel in ascend npu.

**Step 7**:
Run the benchmark test cases, record the performance metrics. Read [kernel optimize guidance](./reference/kernel-optimize-guidance.md) and analyze the performance metrics, give the optimization suggestions and wait for the user's confirmation.

**Step 8**:
Iterate the steps from step 4 to step 7, until the kernel performance satisfies the requirement.

## Project structure
```
.
├── SKILL.md                              # this file
└── reference
    ├── ascend-npu-hardware-spec.md       # ascend npu hardware spec
    ├── kernel-benchmark-guidance.md      # kernel benchmark guidance
    ├── kernel-optimize-guidance.md       # kernel optimize guidance
    └── memory-bound-vs-compute-bound.md  # memory bound and compute bound analysis
```