---
layout: post
title:  "Stacked Hourglass Networks for Human Pose Estimation"
subtitle: "ECCV 2016"
date:   2019-07-04 
categories: [paper]
---

目前单人姿态估计，主流算法是基于Hourglass各种更改结构的算法。

[源代码](http://www-personal.umich.edu/~alnewell/pose )

# Introduction

姿态估计的挑战：

1. 遮挡和剧烈形变
2. 稀有姿态
3. 由于衣着和光照造成的外观变化

> 直观的挑战

4. 不同的关节点可能在不同的特征图上有最好的识别精度
5.  关节之间的关系有助于更好地定位

The network captures and consolidates information across all scales of the image.

> 多尺度特征？
>
> 对于姿态估计这种关联型任务，全身不同的关节点，并不是在相同的feature map上具有最好的识别精度。

Like many convolutional approaches that produce pixel-wise outputs, the hourglass network pools down to a very low resolution, then upsamples and combines features across multiple resolutions.

> 为什么pixel-level的输出要使用这种结构？
>
> 因为要对每一个像素进行分类，所以最后的输出与输入图像相同大小，而CNN会逐渐降低特征的大小，所以要在最后使用反卷积或上采样恢复原来的大小。

On the other hand, the hourglass differs from prior designs primarily in its more symmetric topology. 

> 对称的拓扑结构会带来什么好处吗？
>
> 直接进行很大的上采样得到的结果不够精细，对图像的细节不敏感；使用对称结构，将上采样结果与特征结合，一方面能够融合多尺度特征，另一方面能够细化结果？

We expand on a single hourglass by consecutively placing multiple hourglass modules together end-to-end. This allows for repeated bottom-up, top-down inference across scales. 

> 这种 repeated bottom-up, top-down inference会带来什么好处？
>
> 堆叠的hourglass可以探索关节点之间的关系，功能相当于图模型

In conjunction with the use of intermediate supervision, repeated bidirectional inference is critical to the network’s final performance.

>  intermediate supervision？repeated bidirectional inference？
>
>  可以解决由于网络过深而造成的梯度消失问题

**本文的贡献：**

1. 多尺度特征融合
2. 对称结构
3. 多个对称结构堆叠
4.  中继监督

# Related Work

+ DeepPose： 第一个在姿势估计中使用深度网络；直接回归关节坐标
+  [15]：使用heatmaps（可能是第一个），输入多尺度图像来获取多尺度信息。

---

本文的网络设计很大程度上基于[15]，探索如何获取多尺度信息，并对该网络做了改进以组合多分辨率的特征

[15] 联合使用了卷积网络和图模型，使用图模型学习关节之间的空间关系。但本文在不使用图模型或者任何人体显式模型的情况下达到更好的性能。

> 意味着本文使用其他方法学习关节之间的空间关系？使用什么方法？

---

making successive predictions for pose estimation：

+ [19]：使用Iterative Error Feedback. 将预测结果作为输入输入网络来进行细化。多阶段训练；
+ [18]：multi-stage pose machines 
+ [27]：与本文架构相似，but their model ties weights in the bottom-up and top-down portions of computation as well as across iterations. 

>  intermediate supervision有什么优点？

[15]使用级联使定位更加精确。但是对于由于遮挡或者misattributed limbs 而造成的错误，仅仅细化局部位置并不能提升性能。需要整体的信息。

> 本文在这方面做了什么努力？
>
> 多尺度特征融合+堆叠hourglass？

hourglass modules: 全卷积网络+多尺度空间信息+encoder-decoder architectures 

fully convolutional networks and holistically-nested architectures are both heavy in bottom-up processing but light in their top-down processing 

# Network Architecture

## Hourglass

目的：获取多尺度信息。The person’s orientation, the arrangement of their limbs, and the relationships of adjacent joints are among the many cues that are best recognized at different scales in the image.  

如何融合多尺度特征：use a single pipeline with skip layers to preserve spatial information at each resolution 

![hourglass](https://github.com/suoluowan/learngit/blob/master/images/1562231671583.png?raw=true)

## Layer Implementation

+ reduction steps with 1x1 convolutions 
+ using consecutive smaller filters to capture a larger spatial context 

## Stacked Hourglass with Intermediate Supervision 

**limits of applying intermediate supervision with only the use of a single hourglass module：**

Most higher order features are present only at lower resolutions except at the very end when upsampling occurs.  If supervision is provided after the network does upsampling then there is no way for these features to be reevaluated relative to each other in a larger global context.  

If we want the network to best refine predictions, these predictions cannot be exclusively evaluated at a local scale. The relationship to other joint predictions as well as the general context and understanding of the full image is crucial.  

Applying supervision earlier in the pipeline before pooling is a possibility, but at this point the features at a given pixel are the result of processing a relatively local receptive field and are thus ignorant of critical global cues. 

一个hourglass集成了局部和全局信息。其输出的heatmap代表了输入对象的所有关节点，那么该heatmap就包含了所有关节点的相互关系，可以看作是图模型。所以将第一个hourglass给出的heatmap作为下一个hourglass的输入，就意味着第二个hourglass可以使用关节点件的相互关系，从而提升了关节点的预测精度。

In the final network design, eight hourglasses are used. It is important to note that weights are not shared across hourglass modules, and a loss is applied to the predictions of all hourglasses using the same ground truth.  

# Results

Metric: PCK. reports the percentage of detections that fall within a normalized distance of the ground truth.  

For FLIC, distance is normalized by torso size, and for MPII, by a fraction of the head size (referred to as PCKh). 

## Ablation Experiments

+ stacked hourglass design 

证明性能的提升不是由于网络更深 $\rightarrow$减少hourglass数目的同时，每个hourglass增加残差模块的数目

+ intermediate supervision 

# Further Analysis

+ Multiple People：overlap
+ Occlusion
  + 关节不可见但它的位置能根据上下文看出来（数据集中会给出该关节的标注）
  + 没有该关节的信息（数据集中不标注）





































