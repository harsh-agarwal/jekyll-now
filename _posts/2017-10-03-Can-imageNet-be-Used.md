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

So once we have the HuangYing's Layer we repeated the experiments dont in Learning to See by Moving, in order to sanity check our experiments. Now once we are confident our layers work we can go on to our set of experiments. 

## Network 

A schematic view of the network that we used is as shown: 

<img width="861" alt="screen shot 2017-10-04 at 11 38 10 pm" src="https://user-images.githubusercontent.com/11302053/31191899-449c0138-a95d-11e7-9c48-92e8882a353e.png">

## What's the plan! 

So, you see the idea was simple enough:

- _Get the global features for the Src and the Tgt images_
- _Concatenate the two features and pass them through a couple of FC's_ 
- _Get the 6 DoF's that represent the camera pose to move from Src image to Tgt image_
- _Use the ground truth depth of Tgt image (just for now) and the estimated pose to get the optic flow from Src to Tgt_ 
- _Use this optic flow to warp our Src image into Tgt image to get a warped Tgt image_
- _We would be considering the photometric loss between the warped target and the ground truth target image to train._ 

Key points: 

- _We would be creating a mask as the ground truth depth map has too many holes. we don't want to consider those points in our loss calculation._ 

- _For creating a training set we would be using a particular frame distance or a combination of several frame distances to get the src and the target image. Be careful we need to choose a frame distance that leads to a significan motion!Otherwise the network might learn just to predict zeros :P_

- _We are finetuning the AlexNet which is initialised with the weights of ImageNet clssification features._ 

## Tentative Results  

So we trained the network and following were the results: 

<img width="855" alt="screen shot 2017-10-04 at 11 38 34 pm" src="https://user-images.githubusercontent.com/11302053/31191992-8af9861e-a95d-11e7-8c97-a535dfb98148.png">

The warp error has gone down significantly. And the average loss if we could created a perfect warp is of the order of almost 8000 (because even if we create a perfect warp there would be points that would go out of the frame and we haven't considered a mask for them!) and we have reached to a limit of almost 12000 during our training. The training curve seems to have sudden spikes possibly because of Adam used as an optimiser. As when we repeated the experiment with normal SGD (without momentum) there were no spikes, but the training was very slow. So we went on with this model!  

The following figure shows a qualitative idea about the pose estimated by looking at the warp that we get from the estimated pose and the groud truth pose. Personally, I feel we can obtain better results if we could clean our data a bit and kind of enforce the loss to to give more weights to the edges! 

<img width="580" alt="screen shot 2017-10-04 at 11 44 42 pm" src="https://user-images.githubusercontent.com/11302053/31192119-f1336274-a95d-11e7-8de4-d3f3f0cb605d.png"> 

## Early Fusion Vs Late Fusion

However, we felt we have tried late fusion as an experiment. There is a possibility that we lose too much of information. So, we wanted to try another experiment that would be based on early fusion of images and then passing that through the AlexNet, and see if we are compromising a lot by doing late fusion. 

Quantitatively on calculating the RPE for the two methods: 

|     Fusion          |   Translational           |   Rotational                |
|     Style           |    RMSE(m)                |   RMSE(degrees)             |
|:-------------------:|:-------------------------:|:---------------------------:|              
|**Late**             | 0.180157                  |5.55                         |
|**Early**            | 0.160374                  |4.99                         |

This was expected. definitely early fusion was expected to work better. The rotational RMSE shows a significant improvement as was expected because a small rotational error leads to a large warp error, however that's not the case with translational error. So generally if you would have noticed, when we warp images mannually, we first use the rotation just for our ease. So you see possibly decoupling the fc's and training first for rotation might help. We would consider such experiments some other time :) 

Based on the above experiments we were having two choices: 

1. We go for depth estimation and pose estimation using two images. 
2. We decide to go for pose estimation based on two images and depth estimation based on a single images. 

We would be proceeding with choice two primarily cause that is what Ian was interested in, Single View Depth Estimation and a MonoSlam setup. A potential architecture can be designed using the first idea. And I would be taking that up some other day :) 

So surging ahead we have an idea that we can estimate the pose using an **unsupervised photometric loss** given we have a reasonable depth map of the target image. So now in order to establish the idea that a single network that can be used for monocular depth estimation we need to prove that **_depth features can be used for pose estimation_**. 

For proving this idea I did a very simple experiment, training a completely supervised network for depth and pose. I took a resNet 50 1by2, added a bilinear upsampling layer which increased the size of the network by 32 timees and trained this network on NYUDv2 datasets to get the deapth map. I knew i can't get goo depth maps ou of this. However definately the features that we have in the layers are depth features.Now i froze the depth network. I tapped the network from a suitable point, added a couple of FC's trained it for pose. So in essence I trained a couple of FC's for getting the pose given depth features as input. I was able to train this network reasonbly to get a pose. This experiment kind of gave me a hint that depth features can be used for pose estimation. However we need to improve significantly on the depth features, consequently it's architecture to get better architecture.

TODO :  create a gist and add the link here so that one can visualise the network that was used! 

So we decided to work on depth network that we would be training in a supervised manner for now to get a reasonable depth map. Further we can try working on this network to implement the ideas of Unsupervised Photometric Loss to train it for depth and pose. 

Cheers, 
Harsh Agarwal 







  

         
