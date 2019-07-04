---
layout: post
title:  "Stacked Hourglass Networks for Human Pose Estimation"
subtitle: "ECCV 2016"
date:   2019-07-04 
categories: [paper]
---

目前单人姿态估计，主流算法是基于Hourglass各种更改结构的算法。

姿态估计的挑战：

1. 遮挡和剧烈形变
2. 稀有姿态
3. 由于衣着和光照造成的外观变化

> 直观的挑战

The network captures and consolidates information across all scales of the image.

> 多尺度特征？
>
> 对于姿态估计这种关联型任务，全身不同的关节点，并不是在相同的feature map上具有最好的识别精度。

Like many convolutional approaches that produce pixel-wise outputs, the hourglass network pools down to a very low resolution, then upsamples and combines features across multiple resolutions.

> 为什么pixel-level的输出要使用这种结构？
>
> 因为要对每一个像素进行分类，所以最后的输出与输入图像相同大小，而CNN会逐渐降低特征的大小，所以要在最后使用反卷积或上采样恢复原来的大小。