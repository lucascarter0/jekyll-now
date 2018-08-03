---
layout: post
title: Simple Application of an Unscented Kalman Filter for Navigation
---

The following is a very simple example showcasing the ease and effectiveness of implementing a UKF to improve a navigation solution.

***

In this hypothetical situation, a customer is designing a high-speed train. The train travels along a frictionless rail in one dimension, perpendicular to gravity, and the only external force acting upon the train is aerodynamic drag. In this initial design phase, the goal is to have the train accelerate to a defined velocity, and maintain that velocity for a set time. The train is running autonomously, and will adjust its acceleration based on a navigation estimate that is computed onboard. 

__The problem:__ Using available sensors, we want to determine a navigation estimate that accurately determines train velocity within a set error requirement.

The train is equipped with the following hardware: one accelerometer, and one Doppler radar sensor. 

   $$ f(x|\theta) = \frac{n!}{x_1!x_2!x_3!x_4!}(\frac{1}{2}+\frac{\theta}{4})^{x_1}(\frac{1}{4}(1-\theta))^{x_2}(\frac{1}{4}(1-\theta))^{x_3}(\frac{\theta}{4})^{x_4} $$


