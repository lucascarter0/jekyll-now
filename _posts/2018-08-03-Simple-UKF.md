---
layout: post
title: Simple Application of an Unscented Kalman Filter for Navigation
---

The following is a very simple example showcasing an effective Unscented Kalman Filter implementation to improve a tracking estimate.

***

## Objective
Determine an accurate state estimate to track an object undergoing high-acceleration maneuvers using an unscented kalman filter, python, and filterpy.

## Defining the System (Given Information)
This filter is designed to track an object undergoing high-accleration maneuvers. A sensor supplies noisy position measurements in an inertial frame. The true motion and supplied measurements are plotted below:

![True Target Track and Supplied Measurements.]({{ site.baseurl }}/images/ukf_true_motion.png "Target motion. Target undergoes constant velocity and constant turn rate segments in its trajectory.")

As part of the design, the sensor is designed to capture the position of the target with a standard deviation of 2 meters.
