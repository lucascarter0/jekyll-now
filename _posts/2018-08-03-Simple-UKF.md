---
layout: post
title: Simple Application of an Unscented Kalman Filter for Navigation
---

The following is a very simple example showcasing an effective Unscented Kalman Filter implementation to improve a navigation solution.

***

In this hypothetical situation, a customer is designing a high-speed train. In this initial design phase, the goal is to have the train maintain a defined velocity for a set time. The train is running autonomously, adjusting its acceleration based on an estimated velocity that is computed onboard. 

__Objective:__ Determine an accurate navigation estimate to maintain the train's desired velocity.

__Available information:__ The train travels along a frictionless rail in one dimension, perpendicular to gravity, with aerodynamic drag being the only external force acting on the vehicle. The train adjusts its acceleration with perfect actuation, meaning the train executes the exact acceleration command returned from the onboard computer.

![Train Free Body Diagram]({{ site.baseurl }}/images/tain_fbd.png)

The train's speed is adjusted with a proportional controller. It computes an acceleration command based on velocity error:

$$ a_{commanded} = K*(v_{desired} - v_{current}) $$

where $K$ is a gain parameter.

The train is equipped with the following hardware: one accelerometer, and one Doppler velocimeter. 

   $$ f(x|\theta) = \frac{n!}{x_1!x_2!x_3!x_4!}(\frac{1}{2}+\frac{\theta}{4})^{x_1}(\frac{1}{4}(1-\theta))^{x_2}(\frac{1}{4}(1-\theta))^{x_3}(\frac{\theta}{4})^{x_4} $$
