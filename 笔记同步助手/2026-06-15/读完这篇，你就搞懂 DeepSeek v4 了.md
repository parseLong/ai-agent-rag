---
author: dorian（腾讯程序员）
source: 微信公众号 · 腾讯技术工程
url: https://mp.weixin.qq.com/s/MamimcCQj_Hd12T8iFVmKg?scene=1&click_id=1403356983
saved: 2026-06-15 19:32:47
tags:
  - 笔记同步助手
  - DeepSeek
id: f7e0b716-a744-4d30-8adc-ce36a19e22a6
---

# 读完这篇，你就搞懂 DeepSeek v4 了

> 2026-04-28 发布，对 DeepSeek V4 技术报告的深度解读。已深度提取到 **10 个原子笔记**，本文仅保留索引。

![[V4_header.gif]]

## 🔗 核心内容索引

### 总览与系统级闭环
| 主题 | 原子笔记 | 核心内容 |
|------|---------|---------|
| V4 总览 | [[DeepSeek V4]] | 模型规格、benchmark数据、三项创新互锁逻辑、V4 vs V3.2演进 |
| 上下文 | [[上下文窗口]] | 1M上下文三大消耗场景+Prefill/Decode成本拆分 |

### 三项架构创新（互锁关系）
| 主题 | 原子笔记 | 核心内容 |
|------|---------|---------|
| ① 残差机制 | [[mHC（多流约束残差连接）]] | 标准残差→HC→mHC演进、双随机矩阵约束、非对称约束、表达能力 |
| ② 注意力机制 | [[注意力机制]] | Prefill/Decode成本拆分、CSA三步+HCA两步+混合三层、闪电索引 |
| ③ 训练稳定性 | [[Muon优化器]] | 梯度正交化数学直觉、RMSNorm前置vs LayerNorm、与分布式训练冲突 |
| 对比方案 | [[注意力残差机制]] | Kimi Attn-Res / Block Attn-Res vs mHC |

### 模型架构
| 主题 | 原子笔记 | 核心内容 |
|------|---------|---------|
| MoE架构 | [[MoE架构]] | 路由/门控机制、稀疏激活原理、V4细粒度计算通信重叠、MoE vs Dense对比 |

### Infra 层优化（拆为5个原子笔记）
| 主题 | 原子笔记 | 核心内容 |
|------|---------|---------|
| 算子开发 | [[TileLang]] | 数据流逻辑与调度策略解耦、可移植多硬件 |
| 训练可复现 | [[批无关性与计算确定性]] | bit级一致、DeepGEMM、加法括号结构钉死 |
| 部署效率 | [[FP4量化感知训练]] | 分模块精准量化、QAT vs PTQ对比 |
| 训练/推理框架 | [[DeepSeek V4 训练与推理框架优化]] | Muon兼容分布式、mHC调优(6.7%)、跨机器压缩、单tensor级重算 |
| KV Cache | [[KV 缓存]] | Prefill/Decode角色、V4异构架构、SWA三档取舍、分类持久化 |

## 参考论文

- DeepSeek-V4 技术报告：[DeepSeek\_V4.pdf](https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro/blob/main/DeepSeek_V4.pdf)
- DeepSeek-V3.2：[arxiv.org/pdf/2512.02556](https://arxiv.org/pdf/2512.02556)
- mHC 论文：[arxiv.org/pdf/2512.24880](https://arxiv.org/pdf/2512.24880)
- Attention Residuals：[arxiv.org/pdf/2603.15031](https://arxiv.org/pdf/2603.15031)
- Kimi Linear：[arxiv.org/pdf/2510.26692](https://arxiv.org/pdf/2510.26692)
- Qwen3.5-Omni：[arxiv.org/pdf/2604.15804v2](https://arxiv.org/pdf/2604.15804v2)
