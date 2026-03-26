---
name: triton-ascend-kernels
description: "Provide guidance for writing and benchmarking optimized triton kernels for ascend npus(910B, 910C). Support models like Qwen, Deepseek and so on. The triton kernels could be integrated into vllm inference engine or used separately with torch. This skill should be used when the user asks to \"write/optimize triton kernels\" for Ascend NPUs."
---

# Triton Kernel for Ascend NPU

The skill provides patterns and guidance for developing and optimizing triton kernels targeting Ascend NPUs(910B, 910C) for models like Qwen, Deepseek and so on.

## Key Differences from NVIDIA GPU

**Important**: NPU kernels are fundamentally different from CUDA kernels. Key differences:

- **Device Type**: Use `npu` instead of `cuda`. Always check `torch.npu.is_available()` instead of `torch.cuda.is_available()`
- **Memory Model**: NPU uses explicit on-chip buffers (UB, L1, L0A, L0B, L0C) instead of GPU's unified shared memory
- **Alignment Requirements**: NPU buffers require strict alignment (UB: 32B, L0A/L0B/L0C: 512B, etc.)
- **Data Flow**: GM-UB-Vector or GM-L1-L0A/L0B-Cube-L0C-FixPipe-GM patterns

## When This Skill Applies
Use this skill when:
- benchmarking kernel performance against baseline implementations
- writing new triton kernels for ascend npus
- optimizing existing triton kernels for ascend npus
- debugging triton kernel performance issues on ascend npus

## Principles
- **Correctness First**: After each modification, correctness verification and performance measurement must be conducted.
- **Goal-Oriented**: Continue optimization until performance improvement reaches the target, without stopping iteration.
- **Iterative Optimization**: Repeatedly modify, test, and iterate until the target is achieved. Before modifying Triton operator source code, be sure to back up to restore if needed.
- **Precise Modification**: Pursue "surgical-level" precise modifications to avoid introducing new issues.

## Workflow
**Step 1**:
Analyze the existing code for optimizing the triton kernel or understand the user's requirement for writing a new kernel. If write a new kernel, clarify the input/output specifications, data types, and performance goals and goto step 3 directly.

**Step 2**:
Thoroughly analyze the operator's input parameters, data types, shape ranges, functional logic, and computation flow. Refer to [memory bound and compute bound analysis](./reference/memory-bound-vs-compute-bound.md) to evaluate the operator's compute intensity and memory access patterns, determining whether it is compute-bound or memory-bound to guide subsequent design and optimization. Wait for user confirmation after analysis is complete.

**Step 3**:
If writing a new operator, design the operator implementation flow, referring to [ascend npu hardware spec](./reference/ascend-npu-hardware-spec-constraints.md) and [triton npu best practices](./tutorial/README.zh.md). Proceed to subsequent steps after user confirms the design. If optimizing an existing operator, first run the functional test:
```shell
python -m pytest test_<op_name>.py
```
to verify correctness and precision. Then run the performance test:
```shell
msprof op --output=<output_dir> --kernel-name="<op_name>_kernel" --warm-up=20 --launch-count=20 python test_<op_name>_perf.py
```
The `Task Duration(us)` in the output is the key performance metric; record it as the baseline performance.

**Step 4**:
If writing a new operator, implement the operator in `op_name.py` and create two test files — `test_op_name_verify.py` for functional verification and `test_op_name_perf.py` for performance testing — by referring to [triton benchmark guidance](./reference/kernel-benchmark-guidance.md). Run the commands above to verify functional correctness and performance metrics.

If optimizing an existing operator, perform targeted optimization on `<op_name>.py` based on the baseline analysis results. Ensure the performance improvement reaches at least x times (user's target), and the higher the better. Run the following tests:
- Performance test (vs baseline): `msprof op --output=<user-specified-path> --kernel-name="<op_name>_kernel" --warm-up=20 --launch-count=20 python test_<op_name>_perf.py`
- Correctness verification: `python -m pytest test_<op_name>.py`

**IMPORTANT**: Reference documents for iterative tuning
- reference/ascend-npu-hardware-spec-constraints.md
- reference/troubleshooting.md
- tutorial/README.zh.md
- reference/ascend-npu-hardware-spec-constraints.md
- reference/troubleshooting.md
- tutorial/README.zh.md

## Project structure
```
.
├── SKILL.md                              # this file
└── reference
├   ├── ascend-npu-hardware-spec.md       # ascend npu hardware spec and restrictions for kernels
├   ├── kernel-benchmark-guidance.md      # kernel benchmark guidance
├   └── memory-bound-vs-compute-bound.md  # memory bound and compute bound analysis
└── tutorial                              # the best practices for triton kernels on ascend npus
    ├── README.zh.md                      # introduction to the best practices
    ├── basic
    │   ├── 001-vector_add.py                   # 矢量运算单元不支持int64数据类型，不影响精度的情况下，使用int32
    │   ├── 001-vector_add.zh.md                # 矢量运算单元不支持int64数据类型，不影响精度的情况下，使用int32
    │   ├── 002-vector_cmp.py                   # Cmp Op不支持int32/int64数据类型矢量运算，不影响精度的情况下，转换为FP32
    │   ├── 002-vector_cmp.zh.md                # Cmp Op不支持int32/int64数据类型矢量运算，不影响精度的情况下，转换为FP32
    │   ├── 003-ub_overflow.py                  # 通过tiling解决UB内存消耗，避免UB溢出
    │   ├── 003-ub_overflow.zh.md               # 通过tiling解决UB内存消耗，避免UB溢出
    │   ├── 004-discrete_memory_access.py       # 小数据量的离散访存，全部读取到UB后使用Gather语义筛选，替代从GM直接单个读取
    │   ├── 004-discrete_memory_access.zh.md    # 小数据量的离散访存，全部读取到UB后使用Gather语义筛选，替代从GM直接单个读取
    │   ├── 005-load_order.py                   # 将与计算没有依赖的数据加载语句提前，让MTE2指令并行下发
    │   ├── 005-load_order.zh.md                # 将与计算没有依赖的数据加载语句提前，让MTE2指令并行下发
    │   ├── 006-tiling.py                       # 基于昇腾硬件的实际物理核数，划分grid，并在kernel内做tiling
    │   └── 006-tiling.zh.md                    # 基于昇腾硬件的实际物理核数，划分grid，并在kernel内做tiling
    ├── best_practice
    │   ├── 001-assign_req_to_token_pool.py
    │   ├── 002-decode_grouped_attention.py     # 需要加载的Tensor在高维连续，低维离散时，转置实现向量化加载
    │   ├── 002-decode_grouped_attention.zh.md  # 需要加载的Tensor在高维连续，低维离散时，转置实现向量化加载
    │   ├── 003-fused-cat-slice-conv1d.py       # causal_conv1d_update 算子优化实例
    │   ├── 003-fused-cat-slice-conv1d.zh.md    # causal_conv1d_update 算子优化实例
    │   ├── 004-gather_scatter.py
    │   ├── 005-binned_gather_scatter.py
    │   ├── 006-padded_gather_scatter.py
    │   └── utils.py
    └── op_extension
        ├── 001-insert_slice.py                 # 多个数据合并到一起，批量处理，提升数据处理效率
        ├── 001-insert_slice.zh.md              # 多个数据合并到一起，批量处理，提升数据处理效率
        ├── 002-extract_slice.py                # 一次批量读取，截取部分数据处理，提升数据处理效率
        ├── 002-extract_slice.zh.md             # 一次批量读取，截取部分数据处理，提升数据处理效率
        └── 003-load_care_padding.zh.md         # 带Mask的数据加载，被Mask掉部分如果不需要默认值，显示指定，提升MTE2与Vector的并行
```

## Best practices for triton kernels on Ascend NPUs
The best practices for triton kernels on Ascend NPUs could see [triton npu best practices](./tutorial/README.zh.md)

## Question and Answer
The question and answer for triton kernels on Ascend NPUs could see [troubleshooting](./reference/troubleshooting.md)
