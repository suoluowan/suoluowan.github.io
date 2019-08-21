---
layout: post
title:  "Weakly and Semi Supervised Human Body Part Parsing via Pose-Guided Knowledge Transfer"
subtitle: "CVPR 2018"
date:   2019-07-21
categories: [paper]
---

we present a novel method to generate synthetic human part segmentation data using easily-obtained human keypoint annotations.  

Our key idea is to exploit the anatomical similarity among human to transfer the parsing results of a person to another person with similar pose.  

Due to physical anatomical constraints, humans that share the same pose will have a similar morphology.  

> 搜索有同样姿势的样本，平均它们的解析结果可以作为part-level先验

With the strong part-level prior, we combine it with the input image and forward them through a refinement network to generate an accurate part segmentation result. 

> 和PIBO这篇论文有点像，先初始化，然后逐渐优化

# Method

## Problem

目的：使用姿势信息弱监督人体解析网络的训练

> 姿态作为输入？有没有人体部件分割结果作为监督？

## Overview

$$D_p = \{I_i,K_i\}^M_{i=1}$$

$$D_s = \{I_i,S_i,K_i\}^N_{i=1}$$

1. 给定$D_p$种的一张图像和其关节点标注，搜索$D_s$中关节点标注相似的图像

   > 做的是单人的？

2. 对收集到的图像在他们的部件分割结果上使用pose-guided morphing 使其与目标姿态校准，校准后的标注进行平均生成先验

   > 很显然是针对两个数据集的，但是最后怎么评价自己的分割性能？
   >
   > 如果你在相同的数据集上做，搜索到的姿势很可能是自己的姿势，而且一个数据集的姿态相似性应该会比不同的数据集高，这样子得到的分割性能可能无法扩展到多个数据集。
   >
   > 或者说把一个数据集分成两部分？

3. a refinement network is applied to estimate the body part segmentation 

   > 这个网络也是迭代优化的吗？

## Discovering Pose-Similar Clusters 

> 之前也有一篇论文把不同的姿态分类的

1. We normalize the pose size by fixing their torsos to the same length. 
2. the hip keypoints are used as the reference coordinate to align with the origin.  
3. 度量欧氏距离，选择前k个

## Generating Part-Level Prior 

**pose-guided morphing method：**

使用使用两个姿态之间的变换参数进行仿射变换

## Training of Refinement Network  

refinement network: U-Net

使用$D_s$的数据训练

The only difference is that when discovering pose-similar cluster, we find n nearest neighbours each time and randomly pick k of them to generate part-level prior. 































