---
title: 一句话泛读papers
date: 2025-08-12 12:29:29
tags: CV,机器学习
---
# 一句话泛读 papers

简单读一下实验室过两年论文,interest based.

<!--more-->

## 2025

- ✔ *Playing to the Strengths of High- and Low-Resolution Cues for Ultra-high Resolution Image Segmentation*
  Qi Li, Jiexin Luo, Chunxiao Chen, Jiaxin Cai, Wenjie Yang, Yuanlong Yu, **Wenxi Liu***
  **IEEE Robotics and Automation Letters (RA-L)**

​	原有的对于高分辨率图像分割的方法：

	1. 划分重叠的块进行顺序处理，然后用上下文信息和局部精细结构进行刻画。缺点在于整体推理速度慢
	2. 高分辨率图跑浅层网络，低分辨率图跑深层网络学习特征，然后互补。缺点在于包含冗余冲突的语义信息。

​	这个论文就提出一种方法把方法 2 的任务给解耦，也就是说高分辨率特征专门提取低层次的结构细节；对于低分辨率输入我们专注于把识别出的标记划分给区域。也就是说我们的高分辨率图不再需要和低分辨率图做 fusion，而是专注于把图像特征给弄到每个区域上。论文提出了一个框架，框架的三个分支：RPB-划分区域边界，SAB-分配语义标记，LPB-增强小物体分割，并且使用 MoE 方案来做区域像素关联。

​	Future work:提取语义token的工作交给大模型？

- ✔*Keep the Balance: A Parameter-Efficient Symmetrical Framework for RGB+X Semantic Segmentation*
  Jiaxin Cai, Jingze Su, Qi Li, Wenjie Yang, Shu Wang, Tiesong Zhao, Shengfeng He, **Wenxi Liu***
  **CVPR (Oral)**

​	原有对于多模态语义分割的局限性：本质都是对于 RGB 参数的pre-train model进行fine-tune，但是本质上没用上其他模态的全部潜力。并且如果使用 peft 技术那么就会产生噪声。并且之前的方法很多都是对于不对称的模型进行工作

​	本文就做了更有泛用性的工作：提出对称结构的框架，并且对每个模态都可以适用，提出了独立的模块。

- ✔*URWKV: Unified RWKV Model with Multi-state Perspective for Low-light Image Restoration*
  Rui Xu, Yuzhen Niu, Yuezhou Li, Huangbiao Xu, **Wenxi Liu**, Yuzhong Chen
  **CVPR**

​	领域是关于图像复原，图像复原碰到的几种问题：低光照、噪音、模糊……此前的方法很多都是 transformer-based,缺乏adaptive机制，同时因为 Transformer 的参数利用效率不高，垂直部署会遇到问题。这篇文章把基座换成了 RWKV，并且注意到了复杂光照场景下的问题，然后从多状态视角观察：引入了阶段内和阶段间的状态，实现更精细的图像恢复。并且提出了LAN,EMA,SSF来处理多状态特征。

## 2024

- ✔*Monocular BEV Perception of Road Scenes via Front-to-Top View Projection*
  **Wenxi Liu**, Qi Li, Weixiang Yang, Jiaxin Cai, Yuanlong Yu, Yuexin Ma, Shengfeng He, Jia Pan
  **IEEE Transactions on Pattern Analysis and Machine Intelligence (TPAMI)**

​	一个自动驾驶-related 的 work，高清地图重建的纯视觉方案。这篇文章解决了一个很有挑战性的问题：根据只有一张front-view图片转换成鸟瞰图。传统方法通过估计相机参数来研究视角变换，近期方法利用深度学习实现变换，但是效果并不特别好，因为卷积层的感受野是有限的。

​	这篇文章用两个模块：CVP 和 CVT 来完成了这两个工作，还采用了多尺度FTVP模块，将不同尺度的前视图特征投影和传播到顶视图特征，为更精确的物体位置估计提供多尺度细节。

- ✔*Ultra-high Resolution Image Segmentation via Locality-aware Context Fusion and Alternating Local Enhancement*
  **Wenxi Liu**, Qi Li, Xindai Lin, Weixiang Yang, Shengfeng He, Yuanlong Yu
  **International Journal of Computer Vision (IJCV)** [[arxiv](https://arxiv.org/abs/2109.02580)]

  还是做的超高分辨率语义分割的work,follow 此前的流程：裁切-局部分割-拼接。在局部分割的过程中，局部快的语义可以通过不同尺度的上下文区域关联推断。所以本文提出了 LCF 模块，挖掘局部区域和上下文区域之间的相关性。LCF分成两步，先是 LCC 捕捉位置相关性，然后用attention机制增强局部块特征，然后再引入上下文感知融合的 MCF 方案。对于噪声来说，引入 ALE 模块来减少噪声。

- 🛫*Towards next-generation diagnostic pathology: AI-empowered label-free multiphoton microscopy*
  Shu Wang, Junlin Pan, Xiao Zhang, Yueying Li, **Wenxi Liu**, et al.
  **Lights: Science & Applications**

  skip

- ✔*Attentive and Contrastive Image Manipulation Localization with Boundary Guidance*
  **Wenxi Liu**, Hao Zhang, Xinyang Lin, Qing Zhang, Qi Li, Xiaoxiang Liu, Ying Cao
  **IEEE Transactions on Information Forensics & Security (TIFS)** [[code](https://github.com/ZZZH85/ACBG)]

  做的是identification的工作，核心思想在于关注篡改区域的边界，并且使用 Attentive learning和constrative learning。

- ✔*Category-Contrastive Fine-grained Crowd Counting and Beyond*
  Meijing Zhang, Mengxue Chen, Qi Li, Yanchen Chen, Rui Lin, Xiaolian Li, Shengfeng He, **Wenxi Liu***
  **IEEE Transactions on Multimedia (TMM)**、

  人群计数，但是更加关注细粒度

- *STD-Net: Spatio-Temporal Decomposition Network for Video Demoiréing with Sparse Transformers*
  Yuzhen Niu, Rui Xu, Zhihua Lin, **Wenxi Liu***
  **IEEE Trans. Circuits Syst. Video Techn (TCSVT)** [[code](https://github.com/FZU-N/LVDM)]

- *Learning Nighttime Semantic Segmentation the Hard Way*
  **Wenxi Liu**, Jiaxin Cai, Qi Li, Chenyang Liao, Jingjing Cao, Shengfeng He, Yuanlong Yu
  **ACM Transactions on Multimedia Computing Communications and Applications (TOMM)**

  核心在于，关注的是客观环境差的条件下的图像分割问题。看不到全文 jump

- *Another Perspective of Over-Smoothing: Alleviating Semantic Over-Smoothing in Deep GNNs*
  Jin Li, Qirong Zhang, **Wenxi Liu**, Antoni B. Chan, Yang-Geng Fu
  **IEEE Transactions on Neural Networks and Learning Systems (TNNLS)**

- *Efficient Semantic Segmentation for Compressed Video*
  Jiaxin Cai, Qi Li, Yulin Shen, Jia Pan, **Wenxi Liu***
  **ICRA**

- *Memory-Constrained Semantic Segmentation for Ultra-High Resolution UAV Imagery*
  Qi Li, Jiaxin Cai, Jiexin Luo, Yuanlong Yu, Jason Gu, Jia Pan, **Wenxi Liu***
  **IEEE Robotics and Automation Letters (RA-L)** [[paper](https://ieeexplore.ieee.org/document/10380673)][[arxiv](https://arxiv.org/abs/2310.04721)][[code](https://github.com/liqiokkk/SGHRQ)]

- *Prototype learning based genericmultiple object tracking via pointto-box supervision*
  **Wenxi Liu**, Yuhao Lin, Qi Li, Yinhua She, Yuanlong Yu, Jia Pan, Jason Gu
  **Pattern Recognition**

- *Semi-Supervised Domain Generalization with Evolving Intermediate Domain*
  Luojun Lin, Han Xie, Zhishu Sun, Weijie Chen, **Wenxi Liu***, Yuanlong Yu, Lei Zhang
  **Pattern Recognition**

- *Contrastive and Uncertainty-Aware Nuclei Segmentation and Classification*
  **Wenxi Liu**, Qing Zhang, Qi Li, Shu Wang
  **Computers in Biology and Medicine