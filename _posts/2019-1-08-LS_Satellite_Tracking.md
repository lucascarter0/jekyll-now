---
layout: post
title: Satellite Orbit Determination Using Least Squares
permalink: /homework/satellite_least_squares
---

This problem is based on a report I wrote recently for a graduate linear algebra course. It describes a differential-correction technique for estimating a satellite's position and velocity from measurements taken from a ground-based radar station. Since the context of the paper was to address the linear algebra portion of the problem, emphasis is given to the least squares method used to process the ground measurements, and less emphasis to modeling the orbital mecahnics of satellite motion. The report used data and orbital mechanics models from Fundamentals of Astrodynamics and Applications by David Vallado.

***

## Introduction ##

  While orbiting Earth, various forces perturb the motion of satellites, causing them to deviate from their intended path. Periodic impulses must be executed by the satellite’s control system to offset this behavior. To determine the magnitude and direction of the impulses needed to correct orbit deviations, an accurate estimate of the satellite state $x$ is required. In the context of this analysis, satellite state consists of position $r$ and velocity $v$ referenced to an inertial reference frame with axes $I$, $J$, and $K$.

$$ \boldsymbol{x} = \begin{bmatrix}r_I \  r_J  \ r_K  \ v_I  \ v_J  \ v_K\end{bmatrix}^T $$

  Following initial insertion into a desired orbit, propagation models are used to estimate the vehicle state over time, but modeling errors cause the estimate to deviate from the satellites' true state [[^1]]. Radar measurements from a ground-based tracking station can be used to improve this estimate. The least squares technique described in this analysis presents a method for improving upon the propagated state estimate by attempting to minimize the residual error between a sample of ground-based measurements and the predicted satellite state estimate.

## Construction of the Least Squares Problem ## 

For a linear system, the least squares method seeks to minimize the residual error between estimated parameters and observed data.

$$ min f(x) = \|\| \boldsymbol{A}\boldsymbol{x} - \boldsymbol{y} \|\|_2 $$

where $y$ represents a size p x 1 column matrix of the measurement data, $x$ represents a q x 1 column matrix of estimated parameters (the satellite state), and $A$ represents a p x q matrix of the system model relating parameters of interest to the observed data. Noble and Daniel [[^2]] derive that the estimated parameters $x$ minimize the residual error if and only if $x$ solves

$$ \boldsymbol{A}^{T}\boldsymbol{A}\boldsymbol{x} = \boldsymbol{A}^{T}\boldsymbol{y} $$

meaning that once observation and system model matrices are defined, least squares can be used to find an approximation of the system parameters. The relationship between the satellite’s inertial state and ground-based measurements is nonlinear, so to account for this, an approximation is made regarding the system dynamics so that linear least squares may be used. The estimated observation $Ax$ can be linearized about a nominal state using a first order Taylor series. 

At some time $t_i$, the computed observation can be approximated by

$$ \boldsymbol{y}_{c_{i}} = \boldsymbol{A}\boldsymbol{x}_{i} = \boldsymbol{y}_{0} + \frac{\delta\boldsymbol{y}_0}{\delta\boldsymbol{x}}\Delta\boldsymbol{x}_{i} $$

where $y_0$ indicates the nominal observation estimate at time $t_0$, the partial differential term describes how the nominal state is changing with respect to the estimated parameters, and $\delta{x_i}$ represents a change in the estimated parameters from the nominal estimate to the measured time $t_I$. Substituting this approximation back into the least squares model, 

$$
\ \boldsymbol{y}_i = \boldsymbol{y}_{c_{i}} \\
\ \boldsymbol{y}_i = \boldsymbol{y}_{0} + \frac{\delta\boldsymbol{y}_0}{\delta\boldsymbol{x}}\Delta\boldsymbol{x}_{i} \\
\ \boldsymbol{y}_i - \boldsymbol{y}_{0} = \frac{\delta\boldsymbol{y}_0}{\delta\boldsymbol{x}}\Delta\boldsymbol{x}_{i} \\
$$

Arranged in this form, the first-order Taylor series approximation still resembles the least squares problem, where the measurement matrix of residual error and the estimated paramaters are related by a Jacobian matrix. The Gauss-Newton algorithm solves the non-linear least squares problem:

$$ 
\ \Delta\boldsymbol{x}_i \approx (\boldsymbol{J}(\boldsymbol{x}_{0})^{T}\boldsymbol{J}(\boldsymbol{x}_{0}))^{-1}\boldsymbol{J}(\boldsymbol{x}_{0})^{T}\Delta{\boldsymbol{y}_i}  \\
$$

It's important to note that the estimated parameters will be correcting the nominal estimate, whereas the linear least squares example solves for the parameters directly. Using a sample of measurements, the 

##### A. Observation Vector #####

There are various approaches to measuring a satellite’s position from a ground station, but in this application, it can be assumed the ground station of interest measures relative satellite azimuth $\beta$, elevation $el$, and range $\rho$ from a central point on the sensor as described in Figure 1. ![Fig 1. Ground Station Measurements of Orbiting Satellite]({{ site.baseurl }}/images/satellite_fig.PNG "Fig 1. Ground Station Measurements of Orbiting Satellite")

Previously it was shown that for a linear approximation of a nonlinear system, the observation vector is composed of residual values, meaning that the difference of each measurement from the nominal estimate is used to construct the observation vector. Since the satellite is actively moving over time, incremental measurements from the starting epoch time $t_0$ are summed to time $t_i$ to create the residual observation matrix.

$$ \boldsymbol{y}_{i} - \boldsymbol{y}_{0} = \sum_{n=1}^{i} \boldsymbol{y}_{t} - \boldsymbol{y}_{0} $$

##### B. Model Matrix #####

The Jacobian $J$ describes the relationship between observation vector and estimated parameters. In the case of estimating the satellite state, it describes how relative ground-based measurements are changing with respect to the nominal estimated state. Considering each component of the observation and state matrices described previously, the $A$ matrix is expanded to 

$$
\boldsymbol{J}(\boldsymbol{x}) = \begin{bmatrix}
\frac{\delta{\rho}_{i}}{\delta{r}_{I_{0}}} \frac{\delta{\rho}_{i}}{\delta{r}_{J_{0}}} \frac{\delta{\rho}_{i}}{\delta{r}_{K_{0}}} \frac{\delta{\rho}_{i}}{\delta{v}_{I_{0}}} \frac{\delta{\rho}_{i}}{\delta{v}_{J_{0}}} \frac{\delta{\rho}_{i}}{\delta{v}_{K_{0}}} \\
\frac{\delta{\beta}_{i}}{\delta{r}_{I_{0}}} \frac{\delta{\beta}_{i}}{\delta{r}_{J_{0}}} \frac{\delta{\beta}_{i}}{\delta{r}_{K_{0}}} \frac{\delta{\beta}_{i}}{\delta{v}_{I_{0}}} \frac{\delta{\beta}_{i}}{\delta{v}_{J_{0}}} \frac{\delta{\beta}_{i}}{\delta{v}_{K_{0}}} \\
\frac{\delta{el}_{i}}{\delta{r}_{I_{0}}} \frac{\delta{el}_{i}}{\delta{r}_{J_{0}}} \frac{\delta{el}_{i}}{\delta{r}_{K_{0}}} \frac{\delta{el}_{i}}{\delta{v}_{I_{0}}} \frac{\delta{el}_{i}}{\delta{v}_{J_{0}}} \frac{\delta{el}_{i}}{\delta{v}_{K_{0}}}
\end{bmatrix} 
$$

### Jacobian calculated analytically ###

These partial derivatives can be approximated analytically by perturbing each element of the nominal state estimate and propagating the perturbed state to time $t_i$. These methods are discussed at length by Vallado [[^3]].

##### Detail Analytic Technique for Calculating Jacobian from Orbit Propagation #####

## Implementation ##

With the observation, model, and parameter terms defined, linear least squares can be used to improve the initial satellite state estimate. An algorithm to implement this approach consists of the following:
* For each measurement taken from $t_0$ to $t_i$:
  * Compute the residual observation matrix by differencing each measurement with the nominal estimate.
  * Compute the model Jacobian A using analytical propagation techniques.
  * Sum each observation matrix and model matrix for every measurement to $t_i$.
* Compute the estimated state correction term $\delta{x}$ by solving the least squares problem $\boldsymbol{A}^{T}\boldsymbol{A}x=\boldsymbol{A}^{T}\boldsymbol{b}$ using the cumulative observation and model matrices.
* Apply this computed correction to the nominal estimate.
* Repeat process until nominal estimate converges to an acceptable tolerance as specified by the user.

## Application ##

The utility of this approach is demonstrated using a sample set of eighteen measurements taken of a GEOS-III weather satellite during a period of high visibility, lasting approximately five minutes. ![Fig 2. GEOS-III Tracking Data Provided by Ground Station]({{ site.baseurl }}/images/geos_iii_data.PNG "Fig 2. GEOS-III Tracking Data Provided by Ground Station")

An initial estimate of the satellite’s state was provided based on high fidelity propagation methods, but error from the propagation methods was corrected using the least squares method and the data provided in Table 1. The least squares method described previously was used to calculate a correction to the nominal state estimate. This process was repeated for ten iterations to analyze the least squares method’s convergence to an estimated state. In this application, the estimated satellite position was compared against a known true satellite position to calculate a position error magnitude. The results of these iterations are shown in Figure 2. ![Fig 2. Convergence of least squares estimate over ten iterations. ]({{ site.baseurl }}/images/nls_satellite_iters.PNG "Fig 2. Convergence of least squares estimate over ten iterations.")

It is shown by the figure that the position error magnitude quickly converges after only four iterations of applying the least squares correction. With an initial position error magnitude of 250 kilometers from the propagated estimate, the least squares estimated state converges to a position error magnitude of 1.2 kilometers, greatly improving the monitoring ground station’s ability to determine any necessary corrections for the satellite.

## Conclusion ##

  While the least squares approach was shown to significantly improve the estimated parameter accuracy for a nonlinear system, several assumptions are necessary to create this result:
  
* The nominal estimate $x_{0}$ cannot deviate too greatly from the computed state. This means that the initial estimate of the satellite's state must be within a reasonable tolerance of the measured values, or the least squares estimate will quickly diverge.
* All measurements are treated as true measurements of the satellite state. This can cause poor convergence when using sensor measurements with a high degree of uncertainty. 
* The analytical methods used to obtain the model matrix and the first-order approximation of the system are assumed to contain a negligible amount of error. These uncertainties can be accounted for by weighting each coefficient being solved in the least squares problem [[^4]], or by quantifying system uncertainties by Gaussian random variables and weighting the nominal estimate against the least squares estimate through a Kalman filter [[^5]]. 
  
  As shown in this report, linear least squares estimation provides a viable approach to estimating a satellite’s position and velocity at a specified time based on ground-based relative measurements. The nonlinear system relationship between inertial state and relative measurements can be represented by a first-order Taylor series approximation to apply iterative corrections to a nominal state estimate. It was also shown that while some state position error remained after ten iterations of the differential correction technique, linear least squares provided an effective method for greatly improving the propagation-only state estimate.

[^1]: Chen, Bai, Liang, Li, Orbital Data Applications for Space Objects, Springer, Singapore, 2017, Ch 2.
[^2]: Vallado, Fundamentals of Astrodynamics and Applications, 4th Edition, Microcosm Press, Hawthorne, CA, 2013, pp. 731-776.
[^3]: Noble, Daniel, Applied Linear Algebra, 3rd Edition, Prentice Hall, New Jersey, 1998, Ch 2.6.6
[^4]: Long, Cappellari Jr., Velez, Fuchs, “Goddard Trajectory Determination System Mathematical Theory”, NASA document X582-76-77, 1989.
[^5]: M. K. El-Mahy, "Efficient satellite orbit determination algorithm," Proceedings of the Eighteenth National Radio Science Conference, Vol. 1, 2001, pp. 225-232.
