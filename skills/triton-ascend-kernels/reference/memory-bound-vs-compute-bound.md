# Compute-bound and Memory-bound
computation: Refers to computational power, often measured using a popular metric called the FLOPS rate (floating point operations per second). It quantifies a computer’s performance in executing floating-point calculations in one second

```
Flop Rate = Total Floating-Point Operations / Execution Time
```

Memory: This doesn’t refer to the total memory used but rather the memory bandwidth (GB/s), which is the rate at which data can be loaded or stored between memory and processing components.

```
Memory Bandwidth = Bus Width x Clock Speed x Efficiency Factor
```

# How do we determine a good FLOPS rate or Memory Bandwidth?
```
Arithmetic Intensity (AI) = Total Compute (FLOPs) / Total Data Access (Bytes)
                            Unit: FLOPs/Byte

Hardware Balance Point = Peak Compute (FLOPs/s) / Peak Bandwidth (Bytes/s)
                         Unit: FLOPs/Byte

if AI < Balance Point:  Memory-Bound
if AI > Balance Point:  Compute-Bound
```
Compute-bound: Occurs when a computer’s performance is limited by its computational capacity. This is common when executing complex calculations.

Memory-bound: Occurs when performance is limited by the ability to access data from memory. This usually happens when handling large amounts of data load/store operations.

# Example
Ascend 910B

| Parameter | Value |
|-----------|-------|
| FP16 Peak Compute | 256 ~ 376 TFLOPS |
| HBM Memory Bandwidth | 900 GB/s ~ 1.2 TB/s |
| Balance Point | ≈ 213 ~ 313 FLOPs/Byte |

```
Balance Point = Peak Compute / Peak Bandwidth 
              = 256 TFLOPS / 1.2 TB/s 
              ≈ 213 FLOPs/Byte
```
Judgment Rule:
- Operator AI < 213 FLOPs/Byte → Memory-Bound
- Operator AI > 213 FLOPs/Byte → Compute-Bound

Arithmetic Intensity Calculation Method
```
Arithmetic Intensity (AI) = Total Compute (FLOPs) / Total Data Access (Bytes)

Data Access = Read + Write
            = All Input Tensor Sizes + All Output Tensor Sizes
```

## Example 1: Matrix Multiplication (MatMul / GEMM) Compute-Bound
Scenario: QKV projection in Transformer, shape [B, S, H] × [H, H']
```
Input: A [B, S, H], B [H, H']
Output: C [B, S, H']

Assume: B=1, S=4096, H=4096, H'=4096

Compute (FLOPs):
FLOPs = 2 × B × S × H × H'  (multiply + add)
      = 2 × 1 × 4096 × 4096 × 4096
      = 137,438,953,472 FLOPs
      ≈ 137.4 GFLOPs

Data Access (Bytes):
Input A: B × S × H × 2 Bytes (FP16) = 1 × 4096 × 4096 × 2 = 33,554,432 Bytes
Input B: H × H' × 2 Bytes           = 4096 × 4096 × 2 = 33,554,432 Bytes
Output C: B × S × H' × 2 Bytes      = 1 × 4096 × 4096 × 2 = 33,554,432 Bytes

Total Access = 33,554,432 × 3 = 100,663,296 Bytes ≈ 96 MB

Arithmetic Intensity:
AI = 137,438,953,472 FLOPs / 100,663,296 Bytes
   ≈ 1,365 FLOPs/Byte

Judgment:
1,365 FLOPs/Byte > 213 FLOPs/Byte (Balance Point)
→ Compute-Bound ✅
```
Conclusion: MatMul is a typical Compute-Bound operator. Optimization focus: improve compute utilization (e.g., Tensor Core/3D Cube).