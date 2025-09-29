bisenet v2

ViT 代码

---

真的去做的方向？d

可以做的：

1. RGB-P（材料分割，缺陷检测）Polarization，

​	基础模型迁移->微调（Polarization 没人做微调），用很好的基础模型干！

​	观察信息和什么有关？

2. 裂缝检测（二分类任务，加一个额外模态）

这个分割出来的就是细长裂缝？传统微调不一定会work，用新的微调方式 ICCV25,

3. RGB-D/RGB-T 压力大（）
   沿着学长的 RGB-X 一直做，nips2025 nku 的学校。

4. RGB-任意 deliver 的微调（很多人做，hkust在做）
5. 事件相机 RGB-event

最多：两卡 3090