---
{"dg-publish":true,"dg-permalink":"/大语言模型学习/训练推理优化/训练框架/X-ray","dg-home":false,"dg-description":"在此输入笔记的描述","dg-hide":false,"dg-hide-title":false,"dg-show-backlinks":true,"dg-show-local-graph":true,"dg-show-inline-title":true,"dg-pinned":false,"dg-passphrase":"在此输入访问密码","dg-enable-mathjax":false,"dg-enable-mermaid":false,"dg-enable-uml":false,"dg-note-icon":0,"dg-enable-dataview":false,"tags":["NLP"],"permalink":"/大语言模型学习/训练推理优化/训练框架/X-ray/","dgShowBacklinks":true,"dgShowLocalGraph":true,"dgShowInlineTitle":true,"dgPassFrontmatter":true,"noteIcon":0,"created":"2025-04-29T22:54:30.587+08:00","updated":"2025-04-29T22:55:13.808+08:00"}
---



# X-ray: 大语言模型分布式强化学习框架

## 构建思想
在强化学习中，特别是基于人类反馈的强化学习（RLHF），通常需要多个模型之间进行交互通信。这种通信具有数据量较小、交互模式灵活性要求高的特点，因此非常适合使用 Ray 的 Actor Programming 编程范式来实现模型间通信（inter model communication）。

大模型的训练通常采用 3D 并行技术，其前向、反向和生成过程涉及到大量数据通信。为了高效地实现这些通信，通常采用 GPU NCCL 的集合通信（collective communication）。开源工具如 Deepspeed 和 Megatron 已经提供了模型内通信（intra model communication）的高效解决方案。

通过构建模型间通信和模型内通信的层次化通信平面，我们可以实现以下目标：
- 模型间：基于 Ray 的 Actor Programming 编程范式，实现高效的异步通信。
- 模型内：集成 Megatron 的 TP/PP/DP 并行算法，集成 Deepspeed 的 DP（Zero）并行算法，具备完整的 3D 并行加速方案。

同时，借鉴 trlX 对 PPO 等算法的实现（该算法已经过实战测试），我们采用了 trlX 的 PPO 算法流程。

此外，引入 Deepspeed Inference 和 vLLM 推理加速方案，加速 Actor 的 token 生成效率，支持两种后端组合：Megatron 训练 + vLLM 推理，以及 Deepspeed Zero3 训练 + Deepspeed Inference 推理。


## 大模型并行训练
在训练大模型时，必然伴随着大数据的处理。并行训练主要涉及以下几个技术：

### 数据并行（Data Parallel，DP）
在数据并行中，数据在模型副本之间分摊，在反向传播时副本之间交换梯度。更多细节可以参考 [PyTorch 的教程](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html)。


### 张量并行（Tensor Parallel，TP）
张量并行涉及 Intra Layer 内的拆分。例如，在 Transformer 结构上按照 Attention Heads 进行 $N$ 等分，同时也会对 Feed Forward Network 进行 $N$ 等分。关于这方面的详细信息，可以参考 [Megatron-LM: Training Multi-Billion Parameter Language Models Using Model Parallelism]


### 流水线并行（Pipeline Parallel，PP）
流水线并行涉及 Inter Layer 间的拆分。例如，[0，$N$) 层在 GPU 0，[N, 2N) 层在 GPU 1。关于这方面的详细信息，可以参考 GPipe: Efficient Training of Giant Neural Networks using Pipeline Parallelism。
