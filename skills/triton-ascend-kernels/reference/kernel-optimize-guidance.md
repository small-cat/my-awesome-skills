# kernel optimization guidance
## optimization checklist
### Essential Optimizations (Apply First)
- [ ] Memory Coalescing: Consecutive threads access consecutive addresses
- [ ] Kernel Fusion: Combine operations to reduce memory traffic
- [ ] Shared Memory: Cache frequently accessed data
- [ ] Boundary Checks: Validate all array accesses (tid < size)

### Performance Optimizations (Apply as Needed)
- [ ] Vectorized Memory: Use float2/float4 for higher throughput
- [ ] Loop Unrolling: Increase instruction-level parallelism
- [ ] Mixed Precision: FP16/TF32 where appropriate
- [ ] Double Buffering: Overlap computation with memory transfers

### 针对 npu 的优化方式


### best practices
1. 参考 [triton ascend 最佳实践](https://ascendnpu-ir.gitcode.com/zh_cn/sources/user_guide/best_practice_zh.html)