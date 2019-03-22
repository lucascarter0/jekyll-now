---
layout: post
title: Simple Application of an Unscented Kalman Filter for Navigation
---

The following is a very simple example showcasing an effective Unscented Kalman Filter implementation to improve a navigation solution.

***

In this hypothetical situation, a customer is designing a high-speed train. In this initial design phase, the goal is to have the train maintain a defined velocity for a set time. The train is running autonomously, adjusting its acceleration based on an estimated velocity that is computed onboard. 

## Objective
Determine an accurate navigation estimate to maintain the train's velocity within a maximum error at the end of its cruise manuever.

## Defining the System (Given Information)
The train travels along a frictionless rail in one dimension, perpendicular to gravity, with aerodynamic drag being the only external force acting on the vehicle. The train adjusts its acceleration with perfect actuation, meaning the train executes the exact acceleration command returned from the onboard computer. ![Fig 1. Train Free Body Diagram]({{ site.baseurl }}/images/train_fbd.png "Fig 1. Train Free-Body Diagram")

The train's speed is adjusted with a proportional controller. It computes an acceleration command based on velocity error:

$$ a_{commanded} = K(v_{desired} - v_{t}) $$

where $K$ is a gain parameter and $t$ is the given time. There are no other constraints or design requirements on the controller. The acceleration of the vehicle due to drag is proportional to the train's current velocity by a drag parameter $C_D$. Combining the controller and system dynamics, the governing equation of the train's velocity is:

$$ \dot{v_t} = K(v_{desired} - v_t) - C_D*v_t $$

The customer has requested the following design constraints:

Parameter              | Value 
---                    | ---
$v_{desired}$          | 50 m/s
$t_{end}$              | 100 seconds
Max Error at $t_{end}$ | 2 m/s
$K$                    | 0.35 
$C_D$                  | 0.00001

The ideal execution of this manuever would resemble Figure 2. The vehicle accelerates quickly to its desired velocity, and maintains that velocity until it reaches the end of the manuever ($t_{end}$) with no error. ![Fig 2. Ideal Execution]({{ site.baseurl }}/images/train_ideal.png "Fig 2. Ideal Execution")

## Tracking Velocity
To track the train's velocity, we will start with an accelerometer aligned along the axis of train motion. The accelerometer will measure changes in velocity at a rate of 100 Hz, and its integrated measurements will be used by the controller.

The problem with this is that all accelerometers are subject to random errors, and these errors will accumulate over time when integrated. If we continue using our governing equation with no way to account for these errors, the train's onboard computer will assume it is maintaining the desired velocity, while in reality the true velocity is deviating due to error, as seen in the figure below. 

![Fig 3. Error Propagation]({{ site.baseurl }}/images/train_imuOnly.png "Fig 3. Error Propagation in System")

## Implementing a New Navigation Strategy
We will minimize the final velocity error in two ways:
1. Attempt to account for the accelerometer error in the navigation estimate
2. Fuse the accelerometer information with an additional sensor

Error for this particular accelerometer can be completely modeled by a scale factor $s$ on the measured measured value, a walking bias $b$, and a Gaussian white noise $\eta$. Defining equations for each of these terms:

$$ \dot{b}_t = \frac{-b}{\tau} + w_b $$