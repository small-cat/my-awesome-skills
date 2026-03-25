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
Desired Compute to Memory Ratio (OP/B)

```
Compute to Memory Ratio = Operations(OP) / Bytes Transferred (B)
```

- If OP/B is low: The system is handling heavy computational tasks, but the computational power is limited. This leads to a compute-bound scenario.
- If OP/B is high: The system cannot supply enough data for processing, leading to data starvation or a memory-bound scenario.

Compute-bound: Occurs when a computer’s performance is limited by its computational capacity. This is common when executing complex calculations.

Memory-bound: Occurs when performance is limited by the ability to access data from memory. This usually happens when handling large amounts of data load/store operations.