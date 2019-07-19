---
layout: post
title:  "Point in, Box out: Beyond Counting Persons in Crowds"
subtitle: "CVPR 2019"
date:   2019-07-16 
categories: [paper]
---

> 如何从单个点扩展到bbox？设计思路及方法

It can simultaneously detect the size and location of human heads and count them in crowds.

>  提供信息：
>
> 1. 头部位置（点的形式）
>
> 2. 头部大小？不提供，从头部位置信息挖掘头部大小进行初始化
>
>    基于两个观察：
>
>    1. 近距离行人头部距离影响头部大小
>    2. 同一个水平线的行人头部大小相似，越远越小
>
> 先为每一个点生成初始化的bbox然后调节，调节方式？

提出检测网络在训练时只使用点标注但测试时能够输出bbox标注

# Method

设计网络：

输入：初始化的bbox

输出：精细化的bbox

For each anchor, we predict 4 offsets relative to its coordinates and 1 score for classification.  

> 预先定义了25种anchor，anchor难道不是输入的bbox吗？
>
> 整个网络是在整张图片上做的吗？直接在整张图片上做目标检测？那输入的bbox是怎么使用的？

1. online ground truth (GT) updating scheme  

   从点标注生成初始bbox然后在训练时更新

   > 如何初始化？为什么要这样初始化？

2. locally-constrained regression loss 

   使用点标注回归bbox

   > 如何设计的？

3. curriculum learning strategy  

   先从相对准确的伪GT上训练模型

   > 这种训练策略有什么优点？

## Online ground truth updating scheme 

### 初始化

密集场景种头部大小与两个邻近头部中心点的距离有关$\rightarrow$ 以与最近的头部中心的距离初始化bbox大小$\rightarrow$ 生成正方形bbox

> 在密集人群会接近真实值，但是对于稀疏人群会过大

### 更新

we propose to iteratively update them to train a reliable object detector  

每一次迭代使用bbox大小小于初始bbox的正样本种评分最高的

根据与伪gt的IoU，将anchor分为正负样本进行训练

> 这个正负样本相当于目标检测中的GT？
>
> no，回归的是伪GT的坐标，正负样本输入分类器控制分类器的训练倾向？

## Locally-constrained regression loss 

$$l_{xy} = (gx-\hat{gx})^2+(gy-\hat{gy})^2$$

同一水平线上的行人大小相似$\rightarrow$ 乘法违反这个发现的$\hat{g}$

•让$g_{ij}=(gx_{ij},gy_{ij},gw_{ij},gh_{ij})$记做特征图上ij处的伪gt。先计算一条窄带(
row :i−1:i+1; column: 1:W)中的所有Box的宽/高的均值和标准差，W为特征图宽度，统计如下:

$$\mu w_i = \frac{1}{|G_i|}\sum_{mn\in G_i}gw_{mn}$$

$$\sigma w_i = \sqrt{ \frac{1}{|G_i|}\sum_{mn\in G_i}(gw_{mn}-\mu w_i)^2}$$

如果预测的限位框宽度$\hat{gw}_{ij}$，大于$μw_i+3σw_i$或小于$μw_i−3σw_i$，则将被惩罚。

![PIBO1](https://github.com/suoluowan/learngit/blob/master/images/PIBO1.png?raw=true)

$$L_{reg} = \sum_{ij\in G}\widetilde{lxy_{ij}} + \widetilde{lw_{ij}}+\widetilde{lh_{ij}}$$

## Curriculum learning 

稀疏人群的初始化伪gt通常过大，而极密集场景尺寸又过小，很难检测。这两种情况对于训练都很不利。故采用先简单后困难的训练流程。

对训练集的所有$d(g,NN_g)$计算均值和标准差，并利用高斯密度函数$Φ(d_g|μ,σ)$计算伪gt框的得分（概率），尺寸适中的框会得到更高的分。

一张图片的训练难度定义为

$$TL=1-\frac{1}{|G|}\sum_{g\in G}\Phi(d_g|\mu,\sigma)$$

按照图片的难度分，将训练集分割为多个部分，先训练最简单的，跑一定的epoch后，开始训练最简单和次简单的，依次递推直到训练整个数据集。











































