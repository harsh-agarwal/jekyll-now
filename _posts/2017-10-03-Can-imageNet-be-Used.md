---
layout: post
title : Can ImageNet features be used for getting pose? 
---

I did my internship under _[Prof. Ian D Reid](http://www.robots.ox.ac.uk/~ian/)_ at _University of Adelaide_, I was supposed to work on a ConvNet architecture that can give us the depth and the pose at the same time! So essentially a full-fledged vSLAM system I would say, using a Deep Learning framework. Should be possible, isn’t it? Just imagine when you were a kid you were not taught geometry and concepts of optics and vision to move around, you just started moving around, bumping, falling and eventually learning how to walk! Can we train a DeepNet with a similar idea?

So after reading some papers, specifically [_SFM-Net: Learning Structure and Motion from Video_](https://arxiv.org/pdf/1704.07804.pdf) and [_Unsupervised Learning of depth and Ego-Motion from Video_](https://people.eecs.berkeley.edu/~tinghuiz/projects/SfMLearner/cvpr17_sfm_final.pdf). Well, both the papers have been published by Google and have similar ideas to estimate the depth and pose. 

The basic idea is as follows: 

- Two networks one for calculating the depth and other for determining the pose. 
- They use the pose, and the depth found to get the optic flow, from src image to the target image. 
- The flow is used to warp one image into another. 
- The better the warp, the better was our estimate for depth and pose. 

There are significant differences, in architecture, loss and various other ideas and possibly I would be writing another post on that as well. However, primarily the story remains the same.

After reading several other papers and even trying some methods that have been proposed, yielding no result, Ravi and I had a significant question. **_Can we use features from ImageNet, directly or after fine-tuning, to get the pose?_** Just imagine, you are walking in a market, looking at the surrounding and forming a complete picture, you are trying to make a cogent reason about your position. So can we say that analogous to our understanding of what we are referring to “looking at the surrounding and forming a complete picture,” could be the global features that we get from ImageNet? Worth a shot! 

Implicitly I feel the idea has been proven to work by Learning to See by Moving. In their work they train the network in a supervised manner for pose and say that the features thus obtained can be used for classification. we are kind of going the other way round.  

So we started working on this, there were several steps involved to get this working. First, we need something in CAFFE to convert Depth and Pose into an optic flow that could help us in warping one image into another. This warp would assist us in creating our unsupervised loss! 

So we started making the layers in CAFFE. After struggling a lot with CUDA, we decided to write the layer in python, based on the implementation that has been given by Handa et al. in [GVNN](https://github.com/ankurhanda/gvnn). The GVNN library has been written in Torch, so we created an analogous Python Layer in CAFFE. 

We created three layers (look into pygeometry.py): 

- **SE3 Generator Layer:** This layer would be used for converting the 6 DoF’s that we predict for getting the 4x4 Transformation Matrix 
- **Transform 3D Grid:** You need to transform the Target Depth according to the estimated pose.  
- **Pin Hole Camera Projection:** You need to project the transformed depth onto the image, or equivalently get the flow and add this flow to your source image to obtain your target image. 

The mathematics behind it might seem very complicated. No worries, [Huangying Zhan](https://www.roboticvision.org/rv_person/huangying-zhan/) had prepared a nice documentation regarding the derivation of the formula. We will take up the proof of the SE3 Generator Layer in detail. The complete documentation can be found here. [Spatial Transformer Network Formulation by Huangying Zhan](https://drive.google.com/file/d/0B3BMdiXdDUoKTzExVVctWHB2NTYzMTNROW85a0Jpa1ZybDNJ/view?usp=sharing) 

In most of our articles we would be referring to them as "HuangYing's Layer" :)  



  

         
