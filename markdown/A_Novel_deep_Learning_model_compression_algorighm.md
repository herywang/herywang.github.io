# A Novel Deep Learning Model Compression Algorithm

提出了一种weight allocation(权重分配)的模型压缩算法, 在cifar-10数据集上, resnet-32模型从3726KB压缩到了1842KB, 精度保持在**93.28%**, 比原始模型精度还要高. 提升了计算速度.

剪枝算法有三个优势：①更少的训练时间;②更快的推理速度;③更可行的嵌入式部署;

本文提出的算法分为两个stage: 1.Prue; 2.Quantization

在剪枝阶段使用知识蒸馏来引导训练模型, 提升模型精度. 接着在量化阶段进一步的对模型进行缩小以及降低计算资源的消耗. 剪枝部分使用了FPGM算法(filter pruning via geometric median)




## 部分参考文献

**LeCun, Y.; Denker, J.S.; Solla, S.A. Optimal brain damage. Adv. Neural Inf. Process. Syst. 1990, 2, 598–605.**
> **思想**: OBD的基本思想是将一个训练好的网络，删除一半甚至一半以上的权重，最终会和原来的网络性能一样好，甚至更好.
> 其重新定义的损失: $Loss = 原始的训练误差+网络复杂度的度量$
> 计算过程:
> 1 选择一个合理的网络架构
> 2 训练网络直至收敛
> 3 计算每个参数的二阶偏导数值
> 4 计算每个参数的贡献度
> 5 将参数按照贡献度排序，并删除一些贡献度低的参数
> 6 迭代至第2步

**AHe, Y.; Liu, P.; Wang, Z.; Hu, Z.; Yang, Y. Filter Pruning via Geometric Median for Deep Convolutional Neural Networks Acceleration. In Proceedings of the 2019 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR), Long Beach, CA, USA, 15–20 June 2019**(FPGM)算法
> 提出了通过几何中值的过滤器剪枝方法来压缩模型

**Lee, E.; Hwang, Y. Layer-Wise Network Compression Using Gaussian Mixture Model. Electronics 2021, 10, 72**
> 不同的网络层中, 权重的分布有所区别. 文章提出了一种分层剪枝的方法. 步骤: ①使用高斯混合分布拟合出每一层的权重分布;②根据算法选择哪一层需要被剪枝;③对选中的层进行剪枝操作

**Vanhoucke, V.; Senior, A.; Mao, M.Z. Improving the Speed of Neural Networks on CPUs. Deep Learning and Unsupervised Feature Learning Workshop, NIPS 2011.**
>