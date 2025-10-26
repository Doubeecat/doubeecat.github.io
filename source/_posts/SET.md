---
title: SET
date: 2025-10-18 17:13:33
tags: CV
---

# Motivation

问题是：如何通过微调VFM来学习风格不变的表征？已知频谱分解可以有效分离处理风格和内容信息的方法，那么如何设计一种基于频率空间的方法，以有效地融合VFM特征中的风格与内容信息？这里作者选择使用 FFT，把特征分解成幅度和相位两个分量，相位包含更多低频成分，而幅度包含更多高频成分。由 nightadapter 可知，低频的成分在跨风格变化下稳定，高频不稳定。所以更要关注的就是幅度成分。

# Method

作者认为图像的风格与内容在频率域可分离：这个结论和 nightadapter 是差不多的

- 低频 → 场景结构、语义信息，较稳定；
- 高频 → 风格与纹理信息，易受域差异影响。

因此，作者使用快速傅里叶变换将中间特征分解为振幅和相位两部分：

$$
\alpha = \sqrt{(X_{real})^2 + (X_{img})^2}, \quad \rho = \arctan(X_{img}/X_{real})
$$

再通过逆变换 (IFFT)将增强后的频谱特征投影回空间域。

在每一层冻结的 VFM 特征上，作者引入两组可学习的 Token：
- 振幅 Token ($T_\alpha$)
- 相位 Token ($T_\rho$)

它们与对应的频谱特征通过注意力机制进行特征增强：

$$
M_\alpha = \text{Softmax}(\frac{\alpha_k T_\alpha}{\sqrt{d}}), \quad
\beta(\alpha_k) = M_\alpha \cdot MLP(T_\alpha)
$$

增强后的特征记为：

$$
\alpha' = \alpha + \beta(\alpha), \quad \rho' = \rho + \beta(\rho)
$$

最后再组合回去并送入下一层：

$$
X_{k+1} = V_{k+1}(\text{compose}(\alpha', \rho'))
$$

这里和rein是一样的。

由于振幅特征对风格敏感，作者提出 **注意力归一化优化**：

$$
M_{norm} = \frac{M - \mu}{\sigma}
$$

通过调整相似度分布，使得推理时风格变化不影响 token 匹配，显著提升了跨域鲁棒性。
