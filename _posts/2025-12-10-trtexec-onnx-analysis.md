---
title: 记录一次对 onnx 模型的性能分析(使用 trtexec )
date: 2025-12-10 08:00:00 +0800
categories: [技术]
tags: [TensorRT]
---

## trtexec Performance summary 解析

```bash
[12/09/2025-17:00:23] [I] === Performance summary ===
[12/09/2025-17:00:23] [I] Throughput: 120.031 qps
[12/09/2025-17:00:23] [I] Latency: min = 7.55737 ms, max = 9.15381 ms, mean = 8.18562 ms, median = 8.22565 ms, percentile(90%) = 8.668 ms, percentile(95%) = 8.9481 ms, percentile(99%) = 9.1152 ms
[12/09/2025-17:00:23] [I] Enqueue Time: min = 1.55408 ms, max = 5.7403 ms, mean = 2.12265 ms, median = 2.07227 ms, percentile(90%) = 2.39771 ms, percentile(95%) = 2.57336 ms, percentile(99%) = 2.94116 ms
[12/09/2025-17:00:23] [I] H2D Latency: min = 0.738525 ms, max = 0.76123 ms, mean = 0.740346 ms, median = 0.739258 ms, percentile(90%) = 0.742432 ms, percentile(95%) = 0.744629 ms, percentile(99%) = 0.76001 ms
[12/09/2025-17:00:23] [I] GPU Compute Time: min = 6.12354 ms, max = 7.72314 ms, mean = 6.75204 ms, median = 6.78809 ms, percentile(90%) = 7.2366 ms, percentile(95%) = 7.51614 ms, percentile(99%) = 7.68307 ms
[12/09/2025-17:00:23] [I] D2H Latency: min = 0.691162 ms, max = 0.850952 ms, mean = 0.69323 ms, median = 0.691895 ms, percentile(90%) = 0.693359 ms, percentile(95%) = 0.694397 ms, percentile(99%) = 0.715271 ms
[12/09/2025-17:00:23] [I] Total Host Walltime: 3.00755 s
[12/09/2025-17:00:23] [I] Total GPU Compute Time: 2.43749 s
[12/09/2025-17:00:23] [W] * GPU compute time is unstable, with coefficient of variance = 5.58977%.
[12/09/2025-17:00:23] [W]   If not already in use, locking GPU clock frequency or adding --useSpinWait may improve the stability.
[12/09/2025-17:00:23] [I] Explanations of the performance metrics are printed in the verbose logs.
[12/09/2025-17:00:23] [I]
&&&& PASSED TensorRT.trtexec [TensorRT v101000] [b31] # trtexec.exe --onnx=D:/Self/my-cv-portfolio-project/notebooks/assets/water-meter-normal-pointer-seg-s-640-static.onnx
```

* `Throughput`：吞吐量，这是整体层面的数据，显示了该模型在当前环境运行时每秒可以执行 120.031 次推理 (Query Per Second)。从这个角度来看，假设我们的输入(假设是视频流)是30fps，那么这台机器(GPU)就可以支持4路视频并发。也就是这个参数可以用来规划机器的配置。在实际生产环境下，提升 `Throughput` 就是我们的重大目标，我们可以通过 Stream 技术（推理上一个 batch 时传输下一个 batch）和 Batching （提升单次推理的 batch 大小，如果显存足够的话）技术来提升吞吐量，当然这会增加单次的 `Latency`。

* `Latency`：延迟，代表处理一张图需要多少时间。最重要的指标是P99，这里为9.1152ms，对比mean的8.18562ms，略微高11%，说明99%的请求延迟和平均延迟比较接近，没有出现严重的资源争抢。

* `Enqueue Time`：CPU 将推理任务发送给 GPU 的时间。如果 `Enqueue Time > GPU Compute Time`，说明 CPU 是瓶颈，即CPU 发送任务的速度赶不上 GPU 干活的速度，这时候就需要考虑去提升 CPU 发送任务的能力。而现在 2.94116ms 远小于 7.68307ms，说明 CPU 还有余量去处理其它任务，GPU 是瓶颈，这是比较健康的状态。需要特别说明，对于小模型，在fp16下，可能会出现 False Alarm，就是看着 `Enqueue Time > GPU Compute Time`，乃至于 `Enqueue Time > Latency`(简直违背物理定律，因为 `Latency = Enqueue Time + GPU Compute Time + H2D Latency & D2H Latency`)，具体分析见 `fp32 vs fp16` 小节。

* `H2D Latency & D2H Latency`：Host (CPU) 和 Device (GPU) 之间的数据拷贝操作所带来的延迟，这种传输走的是 PCIe 总线。目前 0.76001ms & 0.715271ms 占用总延迟 8.18562 ms 差不多在 18%，数据传输占比已经有一点大了，需要进行分析与优化，也可以考虑引入 Pinned Memory （锁定内存上的数据，实现 GPU 直读内存，而不通过 CPU 中转）或 Zero Copy （CPU 和 GPU 共用一根内存条，因此不涉及拷贝，只涉及指针指向）等技术。

* `GPU Compute Time`：这是 GPU 里 CUDA Core 和 Tensor Core 真正满载计算的时间。要想缩短这个时间，要么通过剪枝（减少参数），要么量化（通过降低精度减少计算复杂度），要么就是升级显卡本身。

* `Total Host Walltime`：这个代表这次 trtexec 测试所花费的总时间。

* `coefficient of variance`：代表 GPU 计算时间的变异度，这里过高可能是因为 GPU 降频（功耗调度/温度墙）导致的。生产服务器可以通过锁频或者上面所说的 `--useSpinWait` 通过消耗 CPU 的算力来降低 GPU 等待的时间（相当于一直去主动监控 GPU 的运行状态）。


## fp32 vs fp16

```bash
trtexec --onnx=D:/Self/my-cv-portfolio-project/notebooks/assets/water-meter-normal-pointer-seg-s-640-static.onnx
# Throughput: 117.206 qps
# Latency: percentile(99%) = 11.345 ms
# Enqueue Time: percentile(99%) = 3.94824 ms
# H2D Latency: percentile(99%) = 0.762817 ms
# GPU Compute Time: percentile(99%) = 9.91345 ms
# D2H Latency: percentile(99%) = 0.698608 ms

trtexec --onnx=D:/Self/my-cv-portfolio-project/notebooks/assets/water-meter-normal-pointer-seg-s-640-static.onnx --fp16
# Throughput: 201.027 qps
# Latency: percentile(99%) =  6.88574 ms
# Enqueue Time: percentile(99%) = 8.91077 ms
# H2D Latency: percentile(99%) = 0.7771 ms
# GPU Compute Time: percentile(99%) = 5.45386 ms
# D2H Latency: percentile(99%) = 0.712891 ms
```

一切都很符合直觉, Throughput 接近翻倍，但是 Enqueue Time 很奇怪，出现了很异常的抖动。我们看完整数据。

```bash
[12/10/2025-14:10:18] [I] Enqueue Time: min = 1.36401 ms, max = 10.8906 ms, mean = 3.53569 ms, median = 3.34161 ms, percentile(90%) = 4.78271 ms, percentile(95%) = 5.72949 ms, percentile(99%) = 8.91077 ms
```

我们可以看到 Enqueue Time 的 mean 和 median，乃至 p95，都不大，但是出现过一些特别大的，导致 p99 巨大。事实上，这是由于 trtexec 的压测性质引起的，在小模型或者fp16模式下，GPU 计算快，而压测目标是，缺口有多大，就补多少缺口，这是为了测试出极限的 Throughput。这种情况，cpu 向 CUDA Driver(也就是 CUDA 驱动，负责管理 GPU 中的计算指令，而 CPU 往指令队列里面塞新指令，所以这两个之间构成竞争和互锁的关系) 发送任务的频率极高，cpu 线程在调用 enqueue 时会触发流控机制，被迫阻塞。另外 qps 翻倍，意味着 CPU 和 CUDA Driver 之间的同步锁竞争加剧，导致部分请求出现长尾（p99飙升）。

在生产环境中，如果 CPU 抖动影响了整体的稳定性，我会引入 CUDA Graph，把一连串的小任务合并成一个静态图，CPU 只要发一个指令就好，把数百次的 CPU - GPU 交互合并成一次，彻底消除 Enqueue Time 的抖动。

## 静态输入 VS 动态输入