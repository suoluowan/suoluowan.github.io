---
layout: post
title:  "CrowdPose: Efficient Crowded Scenes Pose Estimation and A New Benchmark"
subtitle: "CVPR 2019"
date:   2019-07-10 
categories: [paper]
---

针对拥挤场景下的多人姿态估计问题

# Introduction

解决问题：

1. 拥挤场景数据稀少
2. 多人重叠
   1. 检测器无法生成高质量的bbox
   2. 拥挤场景经常出现漏检或组合错误

Contribution：

1. 构建CrowdPose数据集，用来衡量算法在拥挤场景中的性能.

2. 提出一个高效的算法来解决拥挤人群中的姿态估计问题
   1. 设计适用于拥挤场景的loss函数
   2. 使用图模型做后处理替换传统的NMS

公开数据集的图片缺乏拥挤人群场景的数据，而先前很少有人关注拥挤场景下的姿态估计问题。

Crowd index:

$$ Crowd Index = \frac{1}{n}\sum_{i=1}^n\frac{N_i^b}{N_i^a}$$

![crowdindex1](https://github.com/suoluowan/learngit/blob/master/images/crowdindex1.png?raw=true)



# CrowdPose Dataset

## 数据集构造：

1. 将MPII，COCO和AI Challenger按照crowd index分成20组，从这些组中总共采集30000张图片

2. 由于这三个数据集标注的关节点个数不同,同时拥挤场景很容易出现标注错误，因此对这30000张图片重新进行了标注
   1. 标注14个关键点以及bbox
   2. 分析重新标注的30000张图片的crowd index，选出20000张高质量的图片
   3. 为每一个bbox标注干扰关键点

##  数据集统计：

+ 20000张图片，包含80000个人

+ 一致的crowd index分布

+ 平均bbox IoU：0.27
  + MSCOCO: 0.06
  + MPII: 0.11
  + AI Challenger: 0.12

在拥挤人群的场景下，由于人群过于密集，重合程度太高，一个人的关节点预测很可能被另一个人干扰，传统的SPPE只识别目标关节点，完全消除非目标关节点：

1. 拥挤场景下，很难准确识别目标关节点；当识别错误时，无法使用后处理步骤修正

2. 识别出来的非目标关节点可能有助于对另一个目标关节点的识别

![crowdindex2](https://github.com/suoluowan/learngit/blob/master/images/crowdindex2.png?raw=true)

因此本文提出了JC-SPPE，不仅为目标关节点，也为干扰的关节点生成heatmap（生成的heatmap数是固定的吗？相当于对bbox区域做了一次自底向上？）并为此设计了loss函数

$$ Loss_i = \frac{1}{K}\sum_{k=1}^KMSE[P_i^k, T_i^k+\mu C_i^k]$$

> i表示第i个人，K表示关节点数量。MSE是均方误差损失，P是预测的heatmap，T是目标关节点的heatmap，C是干扰关节点的heatmap。U是衰减因子，一方面要抑制干扰关节点，另一方面由于干扰关节点可能有助于其他目标关节点的检测，不能彻底地抑制。
>
> 使用这样的损失在能够输出目标关节点的情况下，还能输出少量的干扰关节点，保证了召回率
>
> 相当于对bbox区域做了一次自底向上？
>
> 通过JC-SPPE生成的候选关节点大于真实的关节点数量，而且一个bbox生成的关节点并不一定属于一个人，所以要探索这些关节点与人之间的联系，本文通过构建图模型来解决。

## Person-Joint Graph

1. 构造Joint Node集合：当两个关节点很近时，视作一个Node

   $$||p_1^{(k)}-p_2^{(k)}||_2\leq min\{u_1^k, u_2^k\}\delta^{(k)}$$

2. 构造Person Node集合

3. Person-Joint Edge

   如果一个joint Node包含一个person node的候选关节点，则在两者之间添加一条边，这条边的权重是候选关节点的响应评分

在构造了图之后，预测human pose就可以转换为最大化图中所有边的权重来解决

## Globally Optimizing Association

![ci3](https://github.com/suoluowan/learngit/blob/master/images/crowdindex3.png?raw=true)

这个是解决上述问题的目标函数，w是每条边的权重，d表示是否要在最后的图中保留这条边

使用下面的约束来使每一个proposal最多只能匹配一个k节点

在图G解决全局分配问题等效于分别在子图Gk上解决

Gk包含所有的human node和k关节的joint node。

使用KM算法更新优化每一个子图来求得最优解。

![ci4](https://github.com/suoluowan/learngit/blob/master/images/crowdindex4.png?raw=true)

使用图模型做后处理的优点：

1. Reject redundant and poor human proposal：没有显性行人实例的proposal由于关节响应很低无法竞争到关节点而被消除掉

2. 传统的NMS是基于实例的，无法解决漏掉的关节点和错误组合问题，而本文的方法构建了一个全局的图模型，在匹配过程中考虑到了整体的连接方式，因此很好地改善了two-step方法中缺乏全局视野的不足。

# Conclusion

1. 目前大部分算法很难适应拥挤场景
   + 对于top-down，由于检测器很难准确地检测到单个人，十分依赖bbox质量的SPPE很难取得较好的效果
   + 对于bottom-up，身体部件之间的互干扰特别大，因此在合并路线上会有很大的阻碍

2. 类似于top-down和bottom-up的结合
   + 在一个bbox里面检测所有可能的关节点，所有bbox的关节点最后以一个全局的视角与行人实例关联起来。
   + Top-down方法的一个缺点就是因为是对每一个bbox做姿态估计，很难使用全局信息，而拥挤场景下会出现很多干扰节点，全局信息对于准确的关键点预测很重要

3. 拥挤场景下的姿态估计还有很大的提升空间





















