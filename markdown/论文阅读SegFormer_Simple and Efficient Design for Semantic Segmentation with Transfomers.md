# SegFormer: Simple and Efficient Design for Semantic Segmentation with Transformer

文章提出了`SegFormer`算法--使用Transformer架构实现的一种简单, 高效的图像语义分割算法网络模型. 网络有两个亮点: 1)SegFormer使用了一中新型的分层结构Transformer encoder编码器, 输出多尺度的特征. 相对于传统的ViT算法而言, 它不需要使用位置编码, 因此也就避免了如果测试图片的分辨率和训练图片的分辨率不同时导致的性能下降问题; 2)SegFormer避免使用复杂的decoder解码器. 文章中使用MLP解码器聚合了来自不同层的信息, 结合了局部注意和全局注意, 因此具备强有力的表征能力. 在文章中证明了这种简单轻量级的设计是Transformer网络进行有效语义分割的关键手段. 文章对提出的方法进行拓展, 获得了SegFromer-B0 ~ SegFromer-B5六个模型, 与先前的语义分割算法研究相比达到了显著的性能提升和效率提升. SegFormer-B4网络模型参数大小为64MB, 在ADE20K数据集上达到了mIOU=50.3%, 模型大小是先前提出的最好的语义分割算法模型的1/5, 并且mIOU超越之前最好的语义分割算法模型2.2个百分点. 最好的SegFormer-B5模型在Cityscapes校验数据集上则是实现了84.0% mIOU值, 在Cityscapes-C上表现出出色的zero-shot鲁棒性. [代码](github.com/NVlabs/SegFormer)

## Introduction

语义分割(Semantic segmentation)是计算机视觉领域的基础研究, 并且基于语义分割衍生了很多下游的一些应用. 它与图像分类相关, 产生的是像素级别的预测, 这种关系在文章`[1]`中进行了系统性的研究阐述. 作者使用了一种全卷机神经网络实现semantic segmentation, 从那时起FCN激发了很多后序研究工作, 并且成为了密集预测(基于像素的预测)的一种主流方式.

由于senmantic segmentation和Image classification有着非常强的关联性, 许多最先进的语义分割框架是ImageNet上常用的图像分类体系结构的变体, 因此设计一个backbone架构在语义分割领域依然是一个非常活跃的领域. 实际上, 从使用最初的VGG backbone到使用更深层、更强大的backbone`[2]`, backbone的进化升级使得语义分割的性能提升到了一个新的高度.

除了设计各种强大的backbone网络结构之外，另一项工作将语义分割视为结构化预测的的问题，结构化预测的重点是捕获图像的上下文关系，有个典型的代表作就是`空洞卷积(dilated convolution)[3, 4]`，通过在卷积核上使用`空洞`来增加上下文的感受视野。

受Transformer网络在NLP任务上的巨大成功，ViT将Transformer结构引入到计算机视觉分类任务中，是transformer在图像分类领域的开山之作。在语义分割领域Zheng et al.提出了`SETR[5]`算法，证明transformer网络在语义分割领域使用的可行性。SETR使用ViT作为backbone，并结合了一系列的CNN decoder来增加特征感受视野。尽管ViT有着非常好的performance，但仍然有一些局限性：1)ViT输出单尺度、低分辨率特征，而不是多尺度特征；2)在大图像上有着很高的计算成本。为了解决这两个局限性，PVT（pyramid vision transformer)算法模型被提出，相对于ResNet系列在目标检测和语义分割上，PVT在各方面都有着非常大的提升。然而相对于其他的方法，例如Swin Transformer、Twins，这些方法主要考虑的都是在提升encoder层的设计，而忽略了在decoder部分进一步改进的贡献。

**本文所提出的SegFormer算法--在图像语义分割领域一个尖端的transformer框架综合考虑效率、准确率和稳健型。与之前的方法相比，SegFormer重新设计了encoder和decoder，主要创新点有以下：**

> 1. 不再依赖像ViT中的位置编码，并且使用了分层的 Transformer Encoder。
> 2. 使用了轻量级的纯粹多层感知机解码模块(All-MLP decoder)，不在需要复杂且计算消耗高的模块即可产生强大的表征能力。
> 3. SegFormer在公开的三种图像语义分割数据集上获得了最先进的效率，包括：识别效率、准确率和鲁棒性。

## Partial Reference

[[1] Fully Convolutional Networks for Semantic Segmentation](https://openaccess.thecvf.com/content_cvpr_2015/html/Long_Fully_Convolutional_Networks_2015_CVPR_paper.html)
[[2] ResNest: Split-attention networks](https://arxiv.org/abs/2004.08955)
[[3] DeepLab: Semantic Image Segmentation with Deep Convolutional Nets, Atrous Convolution,and Fully CRFs](https://ieeexplore.ieee.org/abstract/document/7913730)
[[4] Multi-scale context aggregation by dilated convolutions](https://arxiv.org/abs/1511.07122)
[[5] Rethinking semantic segmentation from a sequence-to-sequence perspective with transformers](https://openaccess.thecvf.com/content/CVPR2021/html/Zheng_Rethinking_Semantic_Segmentation_From_a_Sequence-to-Sequence_Perspective_With_Transformers_CVPR_2021_paper.html)
