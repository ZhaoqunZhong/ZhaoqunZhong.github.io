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

This [blog](https://www.numerickly.com/2019/10/03/allan-variance-and-its-use-in-characterizing-inertial-measurement-unit-errors/) has a good explanation of the major sources of noise in Gyro, corrsponding to different segments in the Allan Deviation Plot. If you are confused about the various definitions of Allan variance, you only have to know up to the extent of equation (3) in this Freescale [application note](https://www.nxp.com/docs/en/application-note/AN5087.pdf), which is:

$$\theta^{2}(\tau)=\frac{1}{2 \tau^{2}(N-2 m)} \sum_{k=1}^{N-2 m}\left(\theta_{\mathrm{K}+2 m}-2 \theta_{\mathrm{K}+m}+\theta_{\mathrm{K}}\right)^{2}$$

where \\(m\\) is the number of samples in each cluster, notice that \\(\theta\\) is angle instead of angular speed. So \\(\theta^{2}(\tau)\\) is just average of square of \\(\omega\\) sample mean difference of neighbouring clusters. 
The two main noise sources that most IMU algorithms use are Angle Random Walk with the -1/2 slope, and the Rate Random Walk with the +1/2 slope on the Allan Deviation Plot, which are the 'noise' and 'bias' we usually refer to.
According to [Kalibr wiki](https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model#from-the-allan-standard-deviation-ad), these two parameters can be identified like this:

>  \\(\sigma_{g}\\) and \\(\sigma_{a}\\) correspond to the values at \\(\tau=1s\\)(point (1) in the figure below). This is only true since the noise power in most inertial sensors is dominated by "white noise" at a frequency of approximately 1Hz. 
\\(\sigma_{bg}\\) and \\(\sigma_{ba}\\) are identified as the value of the (fitted) "random walk" diagonal at an integration time of \\(\tau=3s\\)(point (2) in the figure below)

I don't know yet the reason why choose \\(\tau=1s\\) and \\(\tau=3s\\), but let's just follow it for now. 

## 2. IMU calibration

[This paper](http://ieeexplore.ieee.org/xpls/abs_all.jsp?arnumber=6907297) describe a steady movement calibration method. The author also shared their [code](https://bitbucket.org/alberto_pretto/imu_tk/src/master/) on bitbucket. Their code has the functionality of computing Allan variance and the calibration of scale and coordinate-misalignment error also. 

## 3. Continuous noise parameters discretization

Currently, I've only found two literatures that explain this process in detail. They are 

* Joan Sola's book <Quaternion kinematics for the error-state Kalman filter>
* A technical report <Indirect Kalman Filter for 3D Attitude Estimation> from MARS LAB

I was really confused by a detail in Joan Sola's book, which is in 

![](/images/imu-noise.png)

Why the noise terms are companied with \\(\Delta t^{2}\\) while the bias terms with \\(\Delta t\\)? The explanation was like this:

![](/images/bias-deltat-explain.png)

Which essentially made the noise term constant during the integration interval. But mathematically, the Angle Random Walk(noise) and Rate Random Walk(directive of bias) terms are of the same type -- white noise. We can of caurse assume any model type of these terms, but declaing one to be white noise and then fix it during integration seems a bit contradictory. If we stick to the assumption that they are both white noise, then the result noise covariance matrix should have both noise terms acompanied with \\(\Delta t\\), which is consistent with the derivation from MARS LAB's report. 