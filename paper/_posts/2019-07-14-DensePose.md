---
layout: post
title:  "DensePose: Dense Human Pose Estimation In The Wild "
subtitle: "CVPR 2018"
date:   2019-07-14 
categories: [paper]
---

Dense pose estimation aims at mapping all human pixels of an RGB image to the 3D surface of the human body. 

> 是否需要深度信息？
>
> no。in our case we consider a single RGB image as input, based on which we establish a correspondence between surface points and image pixels. 

DensePose-COCO

> 标注形式？

SMPL

> 参数化人体模型，是马普所提出的一种人体建模方法，该方法可以进行任意的人体建模和动画驱动

# Annotation

1. we ask annotators to delineate regions corresponding to visible, semantically defined body parts.  

   + These include Head, Torso, Lower/Upper Arms, Lower/Upper Legs, Hands and Feet.  
   + In order to use simplify the UV parametrization we design the parts to be isomorphic to a plane, partitioning the limbs and torso into lower-upper and frontal-back parts. 
   + For head, hands and feet, we use the manually obtained UV fields provided in the SMPL model [27]. For the rest of the parts we obtain the unwrapping via multidimensional scaling applied to pairwise geodesic distances. 

   > UV域是什么？标注的过程中还要用到三维模型？最后标注出来的数据是UV坐标吗？

2. we sample every part region with a set of roughly equidistant points obtained via k-means and request the annotators to bring these points in correspondence with the surface.  

   + The number of sampled points varies based on the size of the part and the maximum number of sampled points per part is 14.  总共336个标注点
   + we ‘unfold’ the part surface by providing six pre-rendered views of the same body part and allow the user to place landmarks on any of them 

   > ？？？在六个视角标注？？
   >
   > 在原图像标注之后，会从6个视角展示标注的点
   >
   > 为了更准确的标注

# The annotations file structure

```json
{
    "images" : [image],
    "annotations" : [annotation],
    "categories" : [category]
}

image {
    "coco_url" : str,
    "date_captured" : datetime,
    "file_name" : str,
    "flickr_url" : str,
    "id" : int,
    "width" : int,
    "height" : int,
    "license" : int
}
// DensePose annotations are stored in dp_* fields:
// 'dp_masks' : RLE encoded dense masks. 针对14个身体部件的mask，每一个身体部件会采样14个点
// All part masks are of size 256x256. They correspond to 14 semantically meaningful parts of the body: Torso, Right Hand, Left Hand, Left Foot, Right Foot, Upper Leg Right, Upper Leg Left, Lower Leg Right, Lower Leg Left, Upper Arm Left, Upper Arm Right, Lower Arm Left, Lower Arm Right, Head
// dp_x, dp_y: spatial coordinates of collected points on the image. The coordinates are scaled such that the bounding box size is 256x256;
// dp_I: The patch index that indicates which of the 24 surface patches the point is on.1, 2 = Torso, 3 = Right Hand, 4 = Left Hand, 5 = Left Foot, 6 = Right Foot, 7, 9 = Upper Leg Right, 8, 10 = Upper Leg Left, 11, 13 = Lower Leg Right, 12, 14 = Lower Leg Left, 15, 17 = Upper Arm Left, 16, 18 = Upper Arm Right, 19, 21 = Lower Arm Left, 20, 22 = Lower Arm Right, 23, 24 = Head;
// dp_U, dp_V: Coordinates in the UV space. Each surface patch has a separate 2D parameterization.
annotation {
    "area": float,
    "bbox": [x, y, width, height],
    "category_id": int,
    "dp_I": [float],
    "dp_U": [float],
    "dp_V": [float],
    "dp_masks": [dp_mask],
    "dp_x": [float],
    "dp_y": [float],
    "id": int,
    "image_id": int,
    "iscrowd": 0 or 1,
    "keypoints": [float],
    "segmentation": RLE or [polygon]
}

category {
    "id" : int,
    "name" : str,
    "supercategory" : str,
    "keypoints": [str],//a length k array of keypoint names
    "skeleton": [edge] //defines connectivity via a list of keypoint edge pairs and is used for visualization.
}

dp_mask {
    "counts": str,
    "size": [int, int]
}
```

# Result file

```json
 {
        "category_id": 1, 
        "uv_shape": [
            3, 
            342, 
            215
        ], // the shape of uv_data array, it should be of size (3, height, width), where height and width should match the bounding box size. 
        "image_id": 785, 
        "score": 0.999954342842102, 
        "bbox": [
            278.7, 
            42.6, 
            215, 
            342
        ], 
        "uv_data": "iVBORw0KGgoAAAANSUhEUgAAANcAAAFWCAIAAAB1hJ8rAABOVUlEQVR42u19d5ydR3X2c2beuyvb\nkrDANk6jORAIhGKa+SChBVIIOBDj0DFgWkIgkAABQhI6BEIJYDoGE2ya6eYzvQRseucDB5yAScDY\nMlIsWdLufWfO98e0M/POu2pX2rvSzG9+47l3765Xd5/7POecOXMO0EYbbbTRRhtttNFGG2200UYb..."// contain PNG-compressed patch indices and U and V coordinates scaled to the range 0-255.
    }, 
```

> The dense estimates of patch indices and coordinates in the UV space for the specified bounding box are stored in `uv_shape` and `uv_data` fields. 

[13] Dense Regression (DenseReg) system 

# DensePose-RCNN

We develop cascaded extensions of DensePose-RCNN that further improve accuracy and describe a training-based interpolation method that allows us to turn a sparse supervision signal into a denser and more effective variant. 

1. 分类像素是属于背景还是parts，并粗糙估计坐标
2. 回归准确坐标

坐标回归：

$$c^* = argmax_cP(c|i), [U,V]=R^{c^*}(i)$$

> P是分类器，R是回归器

## Region-based Dense Pose Regression 

At the top of this branch we have the same classification and regression losses as in the FCN baseline, but we now use a supervision signal that is cropped within the proposed region 

> ???

## Multi-task cascaded architectures 

利用了关键点信息和实例分割

> 直接把keypoint输入1x1卷积提特征加到densepose分支上就可以了？
>
> keypoint对这个网络的贡献大吗？
>
> 只提升了不到一个百分点

![DP1](https://github.com/suoluowan/learngit/blob/master/images/DP1.png?raw=true)

## Distillation-based ground-truth interpolation 

Even though we aim at dense pose estimation at test time, in every training sample we annotate only a sparse subset of the pixels, approximately 100-150 per human. 

For this we adopt a learning-based approach where we firstly train a “teacher” network (depicted in Fig. 9) to reconstruct the ground-truth values wherever these are observed, and then deploy it on the full image domain, yielding a dense supervision signal. 

> 从稀疏到稠密，应该包含采样点之间的关系













































