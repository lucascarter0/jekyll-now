---
layout: post
title: Air Vehicle Simulation in Python
---

# Air Vehicle Simulation in Python

This post will serve as a running catalog for models I am developing for a personal 6-DOF simulation. The motivation for this project is to reduce the amount of startup time required for modeling scenarios for course projects as part of the ACM program at Johns Hopkins.


### Kinematics and Dynamics
Model incorporates forces and moments into vehicle state.

```python
def kinematics_dynamics(state,forces_moments,const):
    '''
    Computes state derivative terms. Assumes state in following format:
        x(1:3):   Position in inertial frame (NED, see assumptions)
        x(4:6):   Velocity in body coordinates
        x(7:10):  NED to Body frame quaternion
        x(11:13): Body frame rates about NED frame, expressed in body frame
        
        Assumptions:
            Flat Earth
            Vehicle is rigid body
            Body frame origin is CG of vehicle
    '''
    x_ned     = state[0:3]
    v_g_b     = state[3:6]
    q_ned_b   = state[6:10]
    w_b_ned_b = state[10:13]

    # Rotation matrix from NED to body
    # TODO: rotate directly from quaternion instead of calculating rotation matrix
    R_ned_b = quaternion_to_rotationmatrix(q_ned_b)

    # Forces and Moments calculated in the body frame
    f_b = forces_moments[0:3]
    m_b = forces_moments[3:6]
    
    
    # Position delta in NED
    x_dot_ned = R_ned_b.transpose().dot(v_g_b)
    
    # Velocity delta in body frame 
    # (since forces and moments are calculated in body frame)
    v_dot_g_b = np.cross(-w_b_ned_b, v_g_b) + (1/const.mass)*f_b

    q = q_ned_b #For simpler notation in calculation
    
    # Quaternion delta from NED to body frame
    q_dot_ned_b = 0.5*np.array([[-q[1], -q[2], -q[3]],
                       [ q[0], -q[3],  q[2]],
                       [ q[3],  q[0], -q[1]],
                       [-q[2],  q[1],  q[0]]]).dot(w_b_ned_b)
    
    # Body rate delta from body to NED, expressed in body frame
    w_dot_b_ned_b = np.linalg.inv(const.J).dot(np.cross(-w_b_ned_b, const.J.dot(w_b_ned_b)) + m_b)
    
    out = np.concatenate([x_dot_ned, 
                          v_dot_g_b, 
                          q_dot_ned_b, 
                          w_dot_b_ned_b])
    
    return out
```


### Support Libraries

#### Quaternion to Rotation Matrix
```python
def quaternion_to_rotationmatrix(q):
    sq = np.square
    
    R = np.nan*np.ones([3,3])

    R[0][0] = sq(q[1]) + sq(q[0]) - sq(q[2]) - sq(q[3])
    R[0][1] = 2*(q[1]*q[2] + q[3]*q[0])
    R[0][2] = 2*(q[1]*q[3] - q[2]*q[0])
    
    R[1][0] = 2*(q[1]*q[2] - q[3]*q[0])
    R[1][1] = sq(q[2]) + sq(q[0]) - sq(q[1]) - sq(q[3])
    R[1][2] = 2*(q[2]*q[3] + q[1]*q[0])
    
    R[2][0] = 2*(q[1]*q[3] + q[2]*q[0])
    R[2][1] = 2*(q[2]*q[3] - q[1]*q[0])
    R[2][2] = sq(q[3]) + sq(q[0]) - sq(q[1]) - sq(q[2])
    
    return R
```

#### Integrator
This function uses Euler method. Assumes small enough time step to be appropriate.

```python
def integrate(state,xdot,const):
    
    state = state + xdot*const.dt
    
    # Normalize quaternion after integration
    state[6:10] = np.divide(state[6:10],
         np.linalg.norm(state[6:10]))
    
    return state
```
