# benchmark example
write a benchmark for triton kernel in ascend npu.
The following is an example for a vector add kernel.

**Important**: Always use `torch.npu.is_available()` for device detection and `device='npu'` for tensor allocation. Never use `cuda`.

define a python file named vector_add/vector_add.py
```python
import torch
import triton
import triton.language as tl
import triton.experimental.tle as tle

# Check device availability - use npu, not cuda!
if not torch.npu.is_available():
    raise RuntimeError("NPU not available")

# define the kerneliton kernel for vector add
@triton.jit
def vector_add_kernel(x_ptr,  # *Pointer* to first input vector.
               y_ptr,  # *Pointer* to second input vector.
               output_ptr,  # *Pointer* to output vector.
               n_elements,  # Size of the vector.
               BLOCK_SIZE: tl.constexpr,  # Number of elements each program should process.
               # NOTE: `constexpr` so it can be used as a shape value.
               ):
    pid = tl.program_id(axis=0)
    block_start = pid * BLOCK_SIZE
    offsets = block_start + tl.arange(0, BLOCK_SIZE)

    # mem_addr_space is language.extra.cann.core.ascend_address_space
    # Use DSA API for NPU - explicit buffer allocation
    a_ub = tle.dsa.alloc([BLOCK_SIZE], dtype=tl.float32, mem_addr_space=tle.dsa.ascend.UB)
    b_ub = tle.dsa.alloc([BLOCK_SIZE], dtype=tl.float32, mem_addr_space=tle.dsa.ascend.UB)
    c_ub = tle.dsa.alloc([BLOCK_SIZE], dtype=tl.float32, mem_addr_space=tle.dsa.ascend.UB)

    t0 = n_elements - block_start
    tail_size = tl.minimum(t0, BLOCK_SIZE)

    tle.dsa.copy(x_ptr + offsets, a_ub, [tail_size])
    tle.dsa.copy(y_ptr + offsets, b_ub, [tail_size])

    tle.dsa.add(a_ub, b_ub, c_ub)
    tle.dsa.copy(c_ub, output_ptr + offsets, [tail_size])


# wrap a python function to call triton kernel
def custom_func(x: torch.Tensor, y: torch.Tensor):
    output = torch.empty_like(x)
    n_elements = output.numel()
    grid = lambda meta: (triton.cdiv(n_elements, meta['BLOCK_SIZE']), )
    vector_add_kernel[grid](x, y, output, n_elements, BLOCK_SIZE=128)
    return output
```

create a python file named vector_add/test_vector_add_perf.py
```python
from vector_add import custom_func

# define the benchmark_function to compare the performance of torch and triton kernel, use do_bench_npu to measure the performance
def benchmark_function():
    torch.manual_seed(0)
    size = 1024
    # IMPORTANT: Use device='npu', not 'cuda'!
    x = torch.rand(size, device='npu', dtype=torch.float)
    y = torch.rand(size, device='npu', dtype=torch.float)

    from triton.backends.ascend.testing import do_bench_npu
    bench_torch = do_bench_npu(lambda: x + y, clear_l2_cache=True, keep_res=True, collect_prof=False)
    bench_triton = do_bench_npu(lambda: custom_func(x, y), clear_l2_cache=True, keep_res=True, collect_prof=False)
    # 保留两位小数输出
    print(f"torch time : {bench_torch:.2f}")
    print(f"triton time: {bench_triton:.2f}")

if __name__ == "__main__":
    benchmark_function()
```
create a python file named vector_add/test_vector_add_verify.py
```python
from vector_add import custom_func
# define the verify_correctness_function to verify the correctness of triton kernel
def verify_correctness():
    torch.manual_seed(0)
    size = 1024
    # IMPORTANT: Use device='npu', not 'cuda'!
    x = torch.rand(size, device='npu', dtype=torch.float)
    y = torch.rand(size, device='npu', dtype=torch.float)
    output_torch = x + y
    output_triton = custom_func(x, y)
    print(f'The maximum difference between torch and triton is '
          f'{torch.max(torch.abs(output_torch - output_triton))}')

if __name__ == "__main__":
    verify_correctness()
```

# profiler guidance
昇腾 Ascend NPU kernel 可以使用 msprof 工具对算子性能进行分析。

## usage
```shell
# --output - 收集到的性能数据的存放路径，默认在当前目录下保存性能数据
# --application - 应用执行命令
msprof --output=xxx --application=""
# msprof --output=./output-profile --application="python vector_add_benchmark.py"

# 单算子上板调优
# --output - 收集到的性能数据的存放路径，默认在当前目录下保存性能数据
# --application - 单算子执行命令
# --kernel-name - 指定要采集的算子名称，支持使用算子名前缀进行模糊匹配
# --aic-metrics - 使能算子性能指标的采集能力和算子采集能力指标（Roofline/Occupancy/MemoryDetail等）
msprof op --output=xxx --application="" --kernel-name=xxx --aic-metrics=xxx
```