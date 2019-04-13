---
title: Keras FRCNN Object Detector for Fox in SSBM
author: ~
date: '2017-12-28'
slug: super-smash-cv
categories: ['cv','python','keras']
tags: ['cv','python','keras','ssbm','fox','20xx','deep learning','computer vision']
---

In this post I'll be sharing a computer vision model that was trained to detect the character Fox in the game *Super Smash Bros Melee* for the Nintendo Gamecube.

<hr>

Here's a sneak peak at the output if you aren't too intereseted in reading more about the process.

![](/assets/2017/12/kjh_plup_vgbc_tbh7_fox_detect2.gif){: .center-image width="80%" }

# The Process

Thanks to there already being a [keras-frcnn framework](https://github.com/yhenon/keras-frcnn) coded up, the steps to making this fox model were reduced to (1) gathering/tagging training data, (2) training the model, & (3) testing the model.

This will be a relatively high level overview.  If you have any specific questions about the process please leave a comment, and I'll do my best to help answer it.

## 1. Gathering & tagging training data

The data used was from the prominent Super Smash Bros streamer [VGBootCamp](https://www.twitch.tv/vgbootcamp).  Training data was gathered from the tournament [Smash Conference](http://wiki.teamliquid.net/smash/Smash_Conference_LXIX/Melee).  Testing data was gathered from [The Big House 7](http://wiki.teamliquid.net/smash/The_Big_House/7).

Bounding boxes were annotated in 348 frames of gameplay for training.  The bounding boxes were drawn using [imglab](https://github.com/davisking/dlib/tree/master/tools/imglab).

## 2. Training the model

Thanks to the effort put in by the keras-frcnn authors and [Adrian Rosebrock](https://twitter.com/PyImageSearch) of [PyImageSearch](https://www.pyimagesearch.com/) the training step was relatively simple.  I used the PyImageSearch [pre-configured Amazon AMI](https://www.pyimagesearch.com/2017/09/20/pre-configured-amazon-aws-deep-learning-ami-with-python/) as the location for model training.  Once the server was spun up I followed the instructions listed in the readme of the [keras-frcnn repo](https://github.com/yhenon/keras-frcnn).  Since I only had 348 training images I supplemented the training by using horizontal flips, vertical flips, and 90 degree rotations.  After a couple days, I stopped the training process to try out the model.

## 3. Testing the model

Again thanks to standing on the shoulders of giants, testing the model was a relatively simple task.  To test I followed the instructions in the readme of the [keras-frcnn repo](https://github.com/yhenon/keras-frcnn), and I was pleasantly surprised by the results.  The model was correctly finding and annotating Fox in a majority of the test images!  However, I did not tag the testing images, so there is no quantitative metric on the model's accuracy.

The .gif at the top of this post shows a combination of man.

# Sources

* [keras-frcnn github repo](https://github.com/yhenon/keras-frcnn)
* [Fast R-CNN white paper](https://www.cv-foundation.org/openaccess/content_iccv_2015/papers/Girshick_Fast_R-CNN_ICCV_2015_paper.pdf)
* [PyImageSearch Deep Learning for CV AMI](https://www.pyimagesearch.com/2017/09/20/pre-configured-amazon-aws-deep-learning-ami-with-python/)
* [imglab](https://github.com/davisking/dlib/tree/master/tools/imglab)
* VGBootCamp ([twitch](https://www.twitch.tv/vgbootcamp) & [youtube](https://www.youtube.com/channel/UCj1J3QuIftjOq9iv_rr7Egw))

<hr>

# Update

I forgot to link to [the github repo for this project](https://github.com/AdamSpannbauer/ssbm_fox_detector).  The repo has all the code, links to training data, & usage information.
