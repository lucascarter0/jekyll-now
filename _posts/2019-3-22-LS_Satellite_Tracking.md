---
layout: post
title: Satellite Orbit Determination Using Least Squares
permalink: /homework/
---

This problem is based on a report I wrote recently for a graduate linear algebra course. It describes a differential-correction technique for estimating a satellite's position and velocity from measurements taken from a ground-based radar station. Since the context of the paper was to address the linear algebra portion of the problem, emphasis is given to the least squares method used to process the ground measurements, and less emphasis to modeling the orbital mecahnics of satellite motion. The report used data and orbital mechanics models from Fundamentals of Astrodynamics and Applications by David Vallado.

***

## Introduction ##

  While orbiting Earth, various forces perturb the motion of satellites, causing them to deviate from their intended path. Periodic impulses must be executed by the satellite’s control system to offset this behavior. To determine the magnitude and direction of the impulses needed to correct orbit deviations, an accurate estimate of the satellite state $x$, consisting of an inertially-referenced position and velocity, is required.

$$ \boldsymbol{x} = \begin{bmatrix}
r_I\\
r_J\\
r_K\\
r_I\\
r_J\\
r_K
\end{bmatrix} $$


Following initial insertion into a desired orbit, propagation models are used to estimate the vehicle state over time, but modeling errors within these methods cause the estimate to deviate from the true state [^1]. Radar measurements from a ground-based tracking station can be used to improve this estimate. The least squares technique described in this paper presents a method for improving upon the propagated state estimate by attempting to minimize the residual error between a sample of ground-based measurements and the predicted satellite state estimate.

## Construction of the Least Squares Problem ## 

This section describes the construction of each element of the least squares approach for estimating the satellite state at a specified time $t_0$, including definition of ground measurements composing the observation vector and analytic techniques for construction of the model matrix. Derivation of the underlying transformations from inertial to relative coordinates is not of interest in the context of this report, but it should be noted that the relationship between the satellite’s inertial state and ground-based measurements is nonlinear. To account for this, an approximation is made regarding the system dynamics so that linear least squares may be used.

Consider a linear system of the form

$$ $$

where $y$ represents a size $p$ x 1 column matrix of observation data, $x$ represents a $q$ x 1 column matrix of estimated parameters (the satellite state in this instance), and $A$ represents a $p$ x $q$ matrix of the system model relating parameters of interest to the observed data. The least squares method seeks to minimize the residual error 

$$ $$

between estimated parameters and observed data to determine an optimal estimate so that

$$ $$

Noble and Daniel [^2] derive that the estimated parameters $x$ minimize the residual error if and only if $x$ solves

$$ $$

meaning that once observation and system model matrices are defined, least squares can be used to find an approximation of the system parameters. In the case of the system model $A$ being composed of nonlinear relationships to the measured observations, an approximation of the system must be used to estimate parameters using linear techniques. The estimated observation $Ax$ can be linearized about a nominal state using a first order Taylor series. At some time $t_i$, the computed observation can be approximated by

$$ $$

where $y_0$ indicates the nominal observation estiamte at time $t_0$, the partial differential term describes how the nominal state is changing with respect to the estimated parameters, and $\deltax_i$ represents a change in the estimated parameters from time $t_0$ to $t_I$. Substituting this approximation back into the least squares system described previously, at some time $t_i$, 

$$ $$
$$ $$
$$ $$

Arranged in this form, it is quickly seen that the first-order Taylor series approximation still resembles the least squares problem, where the observation matrix consists of residual error, and the paramaters being estimated are based on a partial differential relationship to the nominal state.

$$ $$

This means that in the application of least squares, the estimated parameters will be providing a correction to a nominal estimate, whereas the conventional least squares example solves for the parameters directly.

##### A. Observation Vector #####

There are various approaches to measuring a satellite’s position from a ground station, but in this application, it can be assumed a ground stations measures relative satellite azimuth $\beta$, elevation $el$, and range $\rho$ from a central point of the sensor as described in Figure 1.

## Picture of Satellite with caption ##

Previously it was shown that for a linear approximation of a nonlinear system, the observation vector is composed of residual values, meaning that the difference of each measurement from the nominal estimate is used to construct the observation vector. Since the satellite is actively moving over time, incremental measurements from the starting epoch time $t_0$ are summed to time $t_i$ to create the residual observation matrix.

$$ $$

##### B. Model Matrix #####

The model matrix $A$ describes the relationship between observation vector and estimated parameters. In the case of estimating the satellite state, it describes how relative ground-based measurements are changing with respect to the nominal estimated state. Considering each component of the observation and state matrices described previously, the $A$ matrix is expanded to 

$$ Jacobian $$
### Jacobian calculated analytically ###

These partial derivatives can be approximated analytically by perturbing each element of the nominal state estimate and propagating the perturbed state to time $t_i$. These methods are discussed at length by Vallado [^3].

## Implementation ##

With the observation, model, and parameter terms defined, linear least squares can be used to improve the initial satellite state estimate. An algorithm to implement this approach consists of the following:
* For each measurement taken from $t_0$ to $t_i$:
  * Compute the residual observation matrix by differencing the measurement and nominal estimate.
  * Compute the model Jacobian A using analytical propagation techniques.
  * Sum each observation matrix and model matrix for every measurement to $t_i$.
* Compute the estimated state correction term $\deltax$ by solving the least squares problem $\boldsymbol{A}^{T}\boldsymbol{A}x=\boldsymbol{A}^{T}\boldsymbol{b}$ using the cumulative observation and model matrices.
* Apply this computed correction to the nominal estimate.
* Repeat process until nominal estimate converges to an acceptable tolerance as specified by the user.

## Application ##

The utility of this approach was demonstrated using a sample set of eighteen measurements taken of a GEOS-III weather satellite during a period of high visibility, lasting approximately five minutes.

### Table of GEOS-III Data ###

An initial estimate of the satellite’s state was provided based on high fidelity propagations methods, but error from the propagation methods was corrected using the least squares method and the data provided in Table 1. The least squares method described previously was used to calculate a correction to the nominal state estimate. This process was repeated for ten iterations to analyze the least squares method’s convergence to an estimated state. In this application, the estimated satellite position was compared against a known true satellite position to calculate a position error magnitude. The results of these iterations are shown in Figure 2.

It is shown by the figure that the position error magnitude quickly converges after only four iterations of applying the least squares correction. With an initial position error magnitude of 250 kilometers from the propagated estimate, the least squares estimated state converges to a position error magnitude of 1.2 kilometers, greatly improving the monitoring ground station’s ability to determine any necessary corrections for the satellite.

## Conclusion ##

  While the least squares approach was shown to significantly improve the estimated parameter accuracy for a nonlinear system, several assumptions are necessary to create this result. One of the most significant assumptions of approximating a nonlinear system is that the nominal estimate cannot deviate too greatly from the computed state. This means that the satellite state initial estimate must be within a reasonable tolerance of the measured values, or the least squares estimate will quickly diverge from the true state. 
  Another important assumption concerns measurement accuracy and state model accuracy. In this example, all measurements were assumed to provide a perfectly accurate relative position of the satellite. However, physical sensor measurements contain a certain level of uncertainty, the analytical methods used to obtain the model matrix contain uncertainty, and the first-order approximation of the system contains uncertainty. These uncertainties must be accounted for by including the unknown behavior of each term in the system in the form of random variables and measurement weighting [^4]. When system uncertainties are quantified by Gaussian random variables, the uncertainty of each measurement can be used to determine an optimal correction to the nominal state estimate in a technique known as Kalman filtering [^5]. 
  As shown in this report, linear least squares estimation provides a viable approach to estimating a satellite’s position and velocity at a specified time based on ground-based relative measurements. The nonlinear system relationship between inertial state and relative measurements can be represented by a first-order Taylor series approximation to apply iterative corrections to a nominal state estimate. It was also shown that while some state position error remained after ten iterations of the differential correction technique, linear least squares provided an effective method for greatly improving the propagation-only state estimate.

[^1]: Chen, Bai, Liang, Li, Orbital Data Applications for Space Objects, Springer, Singapore, 2017, Ch 2.
[^2]: Vallado, Fundamentals of Astrodynamics and Applications, 4th Edition, Microcosm Press, Hawthorne, CA, 2013, pp. 731-776.
[^3]: Noble, Daniel, Applied Linear Algebra, 3rd Edition, Prentice Hall, New Jersey, 1998, Ch 2.6.6
[^4]: Long, Cappellari Jr., Velez, Fuchs, “Goddard Trajectory Determination System Mathematical Theory”, NASA document X582-76-77, 1989.
[^5]: M. K. El-Mahy, "Efficient satellite orbit determination algorithm," Proceedings of the Eighteenth National Radio Science Conference, Vol. 1, 2001, pp. 225-232.
