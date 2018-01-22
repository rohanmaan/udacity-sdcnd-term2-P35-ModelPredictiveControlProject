# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1(mac, linux), 3.81(Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.

* **Ipopt and CppAD:** Please refer to [this document](https://github.com/udacity/CarND-MPC-Project/blob/master/install_Ipopt_CppAD.md) for installation instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.)
4.  Tips for setting up your environment are available [here](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/23d376c7-0195-4276-bdf0-e02f1f3c665d)
5. **VM Latency:** Some students have reported differences in behavior using VM's ostensibly a result of latency.  Please let us know if issues arise as a result of a VM environment.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./

# Project Description
The purpose of this project is to develop a nonlinear model predictive controller (NMPC) to steer a car around a track in a simulator. The simulator provides a feed of values containing the position of the car, its speed and heading direction. Additionally it provides the coordinates of waypoints along a reference trajectory that the car is to follow. All coordinates are provided in a global coordinate system. 

### The Vehicle Model
The vehicle model used in this project is a kinematic bicycle model. It neglects all dynamical effects such as inertia, friction and torque. The model takes changes of heading direction into account and is thus non-linear. The model used consists of the following equations
```
      // x_[t+1] = x[t] + v[t] * cos(psi[t]) * dt
      // y_[t+1] = y[t] + v[t] * sin(psi[t]) * dt
      // psi_[t+1] = psi[t] + v[t] / Lf * delta[t] * dt
      // v_[t+1] = v[t] + a[t] * dt
      // cte[t+1] = f(x[t]) - y[t] + v[t] * sin(epsi[t]) * dt
      // epsi[t+1] = psi[t] - psides[t] + v[t] * delta[t] / Lf * dt
```
Here, `x,y` denote the position of the car, `psi` the heading direction, `v` its velocity `cte` the cross-track error and `epsi` the orientation error. `Lf` is the distance between the center of mass of the vehicle and the front wheels and affects the maneuverability. The vehicle model can be found in the class `FG_eval`. 

### Polynomial Fitting and MPC Preprocessing
The waypoints are preprocessed by transforming them to the vehicle's perspective. This simplifies the process to fit a polynomial to the waypoints because the vehicle's x and y coordinates are now at the origin (0, 0) and the orientation angle is also zero.
All computations are performed in the vehicle coordinate system. The coordinates of waypoints in vehicle coordinates are obtained by first shifting the origin to the current poistion of the vehicle and a subsequet 2D rotation to align the x-axis with the heading direction. Therby the waypoints are obtained in the frame of the vehicle. A third order polynomial is then fitted to the waypoints. The transformation between coordinate systems is implemented in `transformGlobalToLocal`. The transformation used is 
```
 X' =   cos(psi) * (ptsx[i] - x) + sin(psi) * (ptsy[i] - y);
 Y' =  -sin(psi) * (ptsx[i] - x) + cos(psi) * (ptsy[i] - y);  
```
where `X',Y'` denote coordinates in the vehicle coordinate system. Note that the initial position of the car and heading direction are always zero in this frame. Thus the state of the car in the vehicle cordinate system is 
```
          state << 0, 0, 0, v, cte, epsi;
```
initially. 




### Optimal Control Problem
For every state value provided by the simulator an optimal trajectory for the next N time steps is computed that minimizes a cost function. The cost function is quadratic in the cross-track error, the error in the heading direction, the difference to the reference velocity, the actuator values and the difference of actuator values in adjacent time steps. Specifically I chose parameter values here that lead to smooth driving both for slow (25mph) and fast velocities (70mph). The control problem is restricted by the vehicle model as well as actuator max and min values. The cost function can be found in `FG_eval`. 

In the receeding horizon problem the cost function is minimized at each time step, but only the actuations corresponding to the first time step are sent to the simulator. At the next time step the entire optimal control problem is solved again.

### Model Predictive Control with Latency
An additional complication of this project consists in taking delayed actuations into account. After the solution of the NMPC problem a delay of 100ms is introduced before the actuations are sent back to the simulator. 

When delays are not properly accounted for oscillations and/or bad trajectories can occur. These delays make the control problem a so-called sampled NMPC problem.

The approach to dealing with latency was twofold (not counting simply limiting the speed): the original kinematic equations depend upon the actuations from the previous timestep, but with a delay of 100ms (which happens to be the timestep interval) the actuations are applied another timestep later, so the equations have been altered to account for this . Also, in addition to the cost functions suggested in the lessons (punishing CTE, epsi, difference between velocity and a reference velocity, delta, acceleration, change in delta, and change in acceleration) an additional cost penalizing the combination of velocity and delta  was included and results in much more controlled cornering.

Two common aproaches to take delays into account in detail :
1. In one approach the prospective position of the car is estimated based on its current speed and heading direction by propagating the position of the car forward until the expected time when actuations are expected to have an effect. The NMPC trajectory is then determined by solving the control problem starting from that position. 
2. In the other approach the control problem is solved from the current position and time onwards. Latency is taken into account by constraining the controls to the values of the previous iteration for the duration of the latency. Thus the optimal trajectory is computed starting from the time after the latency period. This has the advantage that the dynamics during the latency period is still calculated according to the vehicle model. 
 
Here, I chose the second approach . The actuations are forced to remain at their previous values for the time of the latency. This is implemented in 
`MPC::Solve` like so. 

```  
  // constrain delta to be the previous control for the latency time
  for (int i = delta_start; i < delta_start + latency_ind; i++) {
    vars_lowerbound[i] = delta_prev;
    vars_upperbound[i] = delta_prev;
  }
 ... 
  
  // constrain a to be the previous control for the latency time 
  for (int i = a_start; i < a_start+latency_ind; i++) {
    vars_lowerbound[i] = a_prev;
    vars_upperbound[i] = a_prev;
  }
```

Accordingly, the value that is fed to the simulator is then taken to be the first freely varying control parameter of the optimal trajectory:
```
          // compute the optimal trajectory          
          Solution solve = mpc.Solve(state, coeffs);

          double steer_value = sol.Delta.at(latency_ind);
          double throttle_value= sol.A.at(latency_ind);
```



### Timestep Length and Elapsed Duration (N & dt)
The time `T=N dt` defines the prediction horizon. Short prediction horizons lead to more responsive controlers, but are less accurate and can suffer from instabilities when chosen too short. Long prediction horizons generally lead to smoother controls. For a given prediction horizon shorter time steps `dt` imply more accurate controls but also require a larger NMPC problem to be solved, thus increasing latency. 

Here I chose values of `N` and `dt` such that drives the car smoothly around the track for slow velocities of about 25mph all the way up to about 70mph. 
The values are `N=12` and `dt=0.05`.  Note that the `100ms = 2*dt` latency imply that the controls of the first two time steps are not used in the optimization. They are frozen to the values of the previous actuations, like so 

### Cost Function Parameters
The cost of a trajectory of length N is computed as follows
```
   Cost  = Sum_i cte(i)^2 
              + epsi(i)^2 
              + (v(i)-v_ref)^2 + delta(i)^2 
              + 10 a(i)^2 
              + 600 [delta(i+1)-delta(i)] 
              + [a(i+1)-a(i)]
```
where the increased weight on steering changes between adjacent time intervals is the most important in order to arrive at smooth trajectories. 





