---
layout: post
title: Simple Application of an Unscented Kalman Filter for Navigation
---

The following is a very simple example showcasing an effective Unscented Kalman Filter implementation to improve a navigation solution.

***

In this hypothetical situation, a customer is designing a high-speed train. In this initial design phase, the goal is to have the train maintain a defined velocity for a set time. The train is running autonomously, adjusting its acceleration based on an estimated velocity that is computed onboard. 

__Objective:__ Determine an accurate navigation estimate maintain the train's velocity within a maximum error at the end of its cruise manuever.

__Defining the System (Given Information):__ The train travels along a frictionless rail in one dimension, perpendicular to gravity, with aerodynamic drag being the only external force acting on the vehicle. The train adjusts its acceleration with perfect actuation, meaning the train executes the exact acceleration command returned from the onboard computer.

![Fig 1. Train Free Body Diagram]({{ "/images/tain_fbd.png" | site.baseurl }})

The train's speed is adjusted with a proportional controller. It computes an acceleration command based on velocity error:

$$ a_{commanded} = K(v_{desired} - v_{t}) $$

where $K$ is a gain parameter and $t$ is the given time. There are no other constraints or design requirements on the controller. The acceleration of the vehicle due to drag is proportional to the train's current velocity by a drag parameter $C_D$. Combining the controller and system dynamics, the governing equation of the train's velocity is:

$$ \dot{v_t} = K(v_{desired} - v_t) - C_D*v_t $$

The customer has requested the following design constraints:

| Parameter              | Value         |
| -----------------------|:-------------:|
| $v_{desired}$          | 50 m/s        |
| $t_{end}$              | 100 seconds   |
| Max Error at $t_{end}$ | 2 m/s         |
| $K$                    | 0.35          |
| $C_D$                  | 0.00001       |

The ideal execution of this controller should resemble Figure 2. The vehicle accelerates quickly to its desired velocity and maintains that it until it reaches the end of the manuever ($t_{end}$).

![Fig 2. Ideal Execution]({{ site.baseurl }}/images/tain_fbd.png)


