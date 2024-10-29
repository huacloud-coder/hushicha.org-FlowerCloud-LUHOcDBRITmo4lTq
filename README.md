

> 来源：晓飞的算法工程笔记 公众号，转载请注明出处


**论文: ClearCLIP: Decomposing CLIP Representations for Dense Vision\-Language Inference**


![](https://developer.qcloudimg.com/http-save/6496381/12565d26a827b9901251f8600a1c6385.png)


* **论文地址：[https://arxiv.org/abs/2407\.12442](https://github.com)**
* **论文代码：[https://github.com/mc\-lan/ClearCLIP](https://github.com):[西部世界官网](https://tianchuang88.com)**


# 创新点




---


* 发现两个关键因素在将`CLIP`适配密集视觉\-语言推理中起着至关重要的作用：残差连接影响的减少以及通过自注意力机制的空间信息重组。
* 提出`ClearCLIP`，在`CLIP`的最后一层中进行了三项简单的修改：去除残差连接、最后一个注意力层中采用自注意力机制以及舍弃前馈网络（`FFN`）。这些修改旨在增强注意力输出，从而为开放词汇语义分割任务生成更清晰的表示。


# 内容概述




---


尽管大规模预训练的视觉\-语言模型（`VLMs`），特别是`CLIP`在各种开放词汇任务中取得了成功，但它们在语义分割中的应用仍然面临挑战，常常产生噪声分割图，存在误分割区域。


论文仔细重新审视了`CLIP`的架构，并确定残差连接是降低分割质量的主要噪声源。通过对不同预训练模型中残差连接与注意力输出的统计特性进行比较分析，发现`CLIP`的图像\-文本对比训练范式强调全局特征，而牺牲了局部可区分性，从而导致噪声分割结果。


为此，论文提出了`ClearCLIP`，这是一种新颖的方法，旨在分解`CLIP`的表示，以增强开放词汇语义分割。对最终层进行了三项简单的修改：去除残差连接、最后一个自注意力层中采用自注意力机制以及丢弃前馈网络。`ClearCLIP`可以一致地产生更清晰、更准确的分割图，并在多个基准测试中超过现有方法。


# ClearCLIP




---


基于`ViT`的`CLIP`模型由一系列残差注意力块组成。


## 舍弃残差连接


![](https://developer.qcloudimg.com/http-save/6496381/8d745cad026f044f6989f07153da4501.png)


通过比较`COCOStuff`数据集中`CLIP-B`/`16`和`CLIP-L`/`14`模型最后一个模块的残差连接 XresXres 与不同注意力输出 XattnXattn 的范数来开始分析，可以很容易地观察到这两个子图的共性和差异：


1. 共性在于`mIoU`曲线和 XattnXattn 的范数曲线表现出一定程度的正相关。
2. 差异包括：`1`）`CLIP-B`/`16`中 XresXres 的范数远小于`CLIP-L`/`14`的范数；`2`）`CLIP-B`/`16`中的注意力修改在`q-k`基线之上表现出一致的改善，而`CLIP-L`/`14`中的情况则没有。


因此，当 XresXres 的影响（或范数）最小化时，注意力修改才是有效的。换句话说， XresXres 显著削弱了`CLIP`在密集推断任务上的表现。


![](https://developer.qcloudimg.com/http-save/6496381/803ead2be22bf01c58bab19bf864658c.png)


为了验证这一假设，基于`CLIP-B`/`16`使用 XsumXsum 、 XresXres 和 XattnXattn 进行开放词汇语义分割实验。`COCOStuff`数据集上的实验结果如图`3`所示，发现 XresXres 的`mIoU`接近于零，这表明残差连接可能对图像分割没有帮助。相反，仅使用 XattnXattn 的`mIoU`显著高于 XsumXsum 。图`3`中的可视化结果表明，`CLIP`的噪声分割图可以分解为一个模糊的 XresXres 图和一个更清晰的 XattnXattn 图。根据这些实验结果，可以初步得出结论：分割图中的噪声主要来源于残差连接。


为了进一步证明 XresXres 如何影响`CLIP`的性能，引入了一个缩放因子 αα ，使得 Xsum\=Xres\+αXattnXsum\=Xres\+αXattn ，该因子控制 XattnXattn 相对于 XresXres 的相对影响。实验表明表明更大的 αα 显著提升了性能，这清楚地说明了 XresXres 对性能的不利影响。


最后，论文建议直接舍弃残差连接以在密集的视觉\-语言推理任务中实现最佳性能。


## 舍弃前馈网络（`FFN`）


`Transformer`架构中的前馈网络（`FFN`）在建模数据中的关系和模式方面起着至关重要的作用，但最近的研究显示，`FFN`在推理过程中对图像表示的影响微乎其微。最后一个注意力模块中的`FFN`特征与最终分类特征的余弦角度明显更大，因此建议在密集预测任务中舍弃`FFN`。


在应用于基础`CLIP`模型时，论文发现移除`FFN`对开放词汇语义分割任务的影响较小。但当与去除残差连接相结合时，舍弃`FFN`会导致结果的改善，特别是在模型规模较大的情况下。这种改进的原理在于，去除残差连接显著改变了`FFN`的输入，从而影响其输出。因此，去除`FFN`的输出可能会减轻其对性能的负面影响。


## 自注意力机制


基于上述分析，使用最后一个自注意力层的注意力输出用于视觉\-语言推理。


Xvisual\=Xattn\=Proj(Attn(⋅)(⋅)⋅v),(1\)(1\)Xvisual\=Xattn\=Proj(Attn(⋅)(⋅)⋅v),受到之前工作的启发，可以在注意力机制 Attn(⋅)(⋅)Attn(⋅)(⋅) 中使用不同的查询\-键组合。实际上， AttnqqAttnqq 在大多数情况下始终能够实现更好的性能，因此选择默认使用它。


# 主要实验




---


![](https://developer.qcloudimg.com/http-save/6496381/79dea488fb2f4807c9b05a860cd844be.png)


![](https://developer.qcloudimg.com/http-save/6496381/342d9a51bfe86e7275da25ce9d792620.png)


![](https://developer.qcloudimg.com/http-save/6496381/346eae6d84ea70317882223b32cefc35.png)


 
 
 



> 如果本文对你有帮助，麻烦点个赞或在看呗～
> 更多内容请关注 微信公众号【晓飞的算法工程笔记】


![work-life balance.](https://upload-images.jianshu.io/upload_images/20428708-7156c0e4a2f49bd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
