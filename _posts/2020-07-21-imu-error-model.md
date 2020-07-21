---
layout: post
title: IMU Noise Model
categories: [SLAM, SensorFusion]
---

Any localization application that uses IMU as an information source can't escape from asking these fundamental questions:

1. What kinds of noises exist in IMU?

2. How do you calibrate these noises?

3. Once you have continuous time noise model parameters, how do you translate them to the discrete ones, which are actually used in most algorithms?

This post is about seeking answers to these three questions.

## 1. IMU noise model

This [blog](https://www.numerickly.com/2019/10/03/allan-variance-and-its-use-in-characterizing-inertial-measurement-unit-errors/) has a good explanation of the major sources of noise in Gyro, corrsponding to different segments in the Allan Deviation Plot. If you are confused about the various definitions of Allan variance, you only have to know up to the extent of equation (3) in this Freescale [application note](https://www.nxp.com/docs/en/application-note/AN5087.pdf), which is 

$$\theta^{2}(\tau)=\frac{1}{2 \tau^{2}(N-2 m)}$$

