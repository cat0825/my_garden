---
{"dg-publish":true,"dg-permalink":"/大语言模型学习/Common-Models常见模型/Qwen系列/Qwen1.5","dg-home":false,"dg-description":"在此输入笔记的描述","dg-hide":false,"dg-hide-title":false,"dg-show-backlinks":true,"dg-show-local-graph":true,"dg-show-inline-title":true,"dg-pinned":false,"dg-passphrase":"在此输入访问密码","dg-enable-mathjax":false,"dg-enable-mermaid":false,"dg-enable-uml":false,"dg-note-icon":0,"dg-enable-dataview":false,"tags":["NLP"],"permalink":"/大语言模型学习/Common-Models常见模型/Qwen系列/Qwen1.5/","dgShowBacklinks":true,"dgShowLocalGraph":true,"dgShowInlineTitle":true,"dgPassFrontmatter":true,"noteIcon":0,"created":"2025-04-25T11:13:40.534+08:00","updated":"2025-04-25T11:15:43.034+08:00"}
---



# Qwen1.5 模型解析与技术创新

## 元数据
- 分类：人工智能
- 标签：Qwen1.5, 模型结构, MoE, 深度学习, 自然语言处理
- 日期：2025年4月12日


## 内容总结
Qwen1.5 是一种先进的自然语言处理模型，其结构和训练方法在多个方面进行了创新和优化。本文将详细解析该模型的关键技术点及其实现细节。


## 模型结构
Qwen1.5 使用了多种优化技术来提升性能：

- **Tokenizer 和模型组件**：使用了 BPE、PreRMSNorm、SwiGLU、RoPE 作为基础组件，形成了所谓的“黄金四件套”。
- **注意力机制**：继续使用 Flash Attention，并通过 `torch.nn.functional.scaled_dot_product_attention` 实现 SDPA Attention。值得注意的是，Qwen1.5-32B 版本引入了 GQA（Gated Query Attention）。
- **其他结构改动**：模型的 Embedding 和输出层没有进行参数共享（`tie_word_embedding=false`），而是仅在 QKV 参数中添加了偏置。

💡 **启发点**：Qwen1.5 的设计中，细粒度专家机制和不共享参数的策略可能为模型性能带来了显著提升。


## 细粒度专家机制
Qwen1.5-MoE-A2.7B 引入了一种细粒度专家机制：

- **专家划分**：将 FFN（前馈神经网络）切分为多个部分，每个部分作为一个独立的专家。
- **初始化策略**：利用 Qwen-1.8B 进行初始化改造，并引入随机性以加速收敛。
- **路由机制**：设置四个共享专家和 60 个路由专家，每次激活四个专家。


## 模型训练
训练数据量未公布，但在偏好对齐部分中，使用了 PPO 和 DPO 方法。Qwen1.5 支持 32K 上下文，并提供了 AWQ 和 GPTQ 的量化模型，包括 int4 和 int8 格式。


## 代码演示
**PreTrainedModel.from_pretrained调用tie_weights方法**

```python
# fromhttps://github.com/huggingface/transformers/blob/ee339bad01bf09266eba665c5f063f0ab7474dad/src/transformers/modeling_utils.py#L1264
def tie_weights(self):
    """
    Tie the weights between the input embeddings and the output embeddings.
    
    If the `torchscript` flag is set in the configuration, can't handle parameter sharing so we are cloning the
    weights instead.
    """
    if getattr(self.config, "tie_word_embeddings", True):
        output_embeddings = self.get_output_embeddings()
        if output_embeddings is not None:
            self._tie_or_clone_weights(output_embeddings, self.get_input_embeddings())
    
    if getattr(self.config, "is_encoder_decoder", False) and getattr(self.config, "tie_encoder_decoder", False):
        if hasattr(self, self.base_model_prefix):
            self = getattr(self, self.base_model_prefix)
        self._tie_encoder_decoder_weights(self.encoder, self.decoder, self.base_model_prefix)
    
    for module in self.modules():
        if hasattr(module, "_tie_weights"):
            module._tie_weights()

```

### 常见错误
> 在实现模型时，注意不要混淆参数共享设置，确保 `tie_word_embedding` 参数正确配置。


### 操作步骤
1. ✅ 初始化 Qwen1.5 使用 Qwen-1.8B。
2. ⚠ 分割 FFN 为多个独立专家。
3. ❗ 使用随机性加速收敛。
4. ✅ 配置路由机制以激活适当数量的专家。


### 数据表格
| 参数              | 描述                   |
|-----------------|----------------------|
| Tokenizer       | BPE, PreRMSNorm, SwiGLU, RoPE |
| 注意力机制         | Flash Attention, SDPA Attention |
| 模型版本          | Qwen1.5-32B 使用 GQA  |


### 行动清单
- 探索 Qwen1.5 在不同任务上的性能表现。
- 深入研究细粒度专家机制对模型效率的影响。
- 尝试在其他模型中应用不共享参数的策略。

> 原始出处：[Qwen1.5 官方博客](https://qwenlm.github.io/zh/blog/qwen1.5/)
