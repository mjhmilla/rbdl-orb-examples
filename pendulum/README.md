pendulumForwardDynamics
Copyright (c) 2019 Matthew Millard <matthew.millard@iwr.uni-heidelberg.de>
Licensed under the zlib license. See LICENSE for more details.

================================================================================
Quick Start
================================================================================

This simple example reads in a pendulum model, described in 
model/pendulum_luamodel.lua, uses rbdl to compute its state derivative, and
uses Boost to numerically integrate its movements. To get started

1. Make a 'build' folder here

2. Run cmake from the 'build' folder. You will be asked to set CUSTOM_RBDL_PATH,
   which is the path to the install folder of the version of RBDL that you 
   would like to use. 

    If you have a system-wide installation of RBDL this folder is '/usr/local'

3. From a terminal in the build directory call 
    make

4. From a terminal in the build directory run

    ./pendulumForwardDynamics

5. If everything works you should see a bunch of data printed to the screen and
   2 files written to the output folder. See the 'Example Output' section for
  an example.

6. To view an animation, from a terminal in the 'pendulum' folder run

  meshup model/pendulum_luamodel.lua output/meshup.csv

================================================================================
Detailed Code Tour
================================================================================

1. RBDL is a C++ implementation of Roy Featherstone's recursive order-n 
   algorithms which makes use of spatial vectors. If you are going to be using
   RBDL you should consider reading the following papers

  Featherstone R. A beginner's guide to 6-d vectors (part 1). IEEE robotics 
  & automation magazine. 2010 Sep;17(3):83-94.

  Featherstone R. A beginner's guide to 6-D vectors (part 2)[tutorial]. IEEE 
  robotics & automation magazine. 2010 Dec;17(4):88-99.

  Featherstone R, Orin D. Robot dynamics: equations and algorithms. 
  In Proceedings 2000 ICRA. Millennium Conference. IEEE International Conference 
  on Robotics and Automation. Symposia Proceedings (Cat. No. 00CH37065) 2000 
  (Vol. 1, pp. 826-834). IEEE.

2. The geometric, inertia, and toplogy properties of the rigid-body model of
   a pendulum are stored in model/pendulum.lua. Have a look through it and
   see if you can answer these questions:

   a. Using the identityMatrix, is the inertia of the pendulum about the Z axis?

   b. The 'joint_frame' locates the location of the joint frame in the 
      coordinates of the parent frame. Using the 'parent' and 'joint_frame'
      entries figure out where the pendulum's revolute joint is located 
      in the ROOT frame

   c. Many quantities in RBDL make use of Featherstone's spatial mathematics.
      A spatial vector has 6 elements: the first 3 correspond to
      rotations about the x,y, and z axis; the second 3 correspond to 
      translations about the x,y, and z axis. Look closely at the 'joint' tag:
      what axis is the pendulum rotating about?

3. The code that reads in the lua model and integrates its state forward in
  time is src/pendulumForwardDynamics.cc. Open it. All of the comments
  below appear in that file - just search for '3a', for example to find
  the first comment in this guided tour:

  3a. The Lua model is read in here, and turned into a series of 
      vectors and matricies in model which RBDL uses to evaluate 
      dynamics quantities

  3b. Here we instantiate a wrapper class which is needed so that 
      Boost can evaluate the state derivative of the model.

  3c. Boost uses this 'operator()' function to evaluate the state
      derivative of the pendulum.

  3d. Here we split out q (generalized positions) and qd 
      (generalized velocities) from the x (state vector)

  3e. Here we set the applied generalized forces to zero

  3f. RBDL's ForwardDynamics function is used to evaluate
      qdd (generalized accelerations)

  3g. Here qd, and qdd are used to populate dxdt 
      (the state derivative)    

  3h. Here we integrate forward in time between a series of nPts from
      t0 to t1

  3i. At each point the state, kinetic (ke), and potential energy (pe) 
      is evaluated. In this conservative system the sum of kinetic and
      potential energy should be constant. Any error that accumulates
      is due to the cumulation of integration error.      
  
  3j. Now the data we have accumulated is written to file


================================================================================
Example Output
================================================================================

ua: Constructing 1 dof pendulum
Lua: Body mass and geometry properties
Lua: Putting bodies in a table
Lua: Making a table of joints
Lua: Making the meshes
Lua: Making the model
DoF: 1
Forward Dynamics 
Columns
      t,         q,       qd,       ke,        pe,   ke+pe-(kepe0)
0.000000, -1.570796, 0.000000, 0.000000, -0.000000, -0.000000
0.010000, -1.570551, 0.049050, 0.002406, -0.002406, 0.000000
0.020000, -1.569815, 0.098100, 0.009624, -0.009624, 0.000000
0.030000, -1.568589, 0.147150, 0.021653, -0.021653, 0.000000
0.040000, -1.566872, 0.196200, 0.038494, -0.038494, 0.000000
0.050000, -1.564665, 0.245249, 0.060147, -0.060147, 0.000000
0.060000, -1.561967, 0.294298, 0.086611, -0.086611, 0.000000
0.070000, -1.558779, 0.343345, 0.117886, -0.117886, 0.000000
0.080000, -1.555100, 0.392390, 0.153970, -0.153970, 0.000000
0.090000, -1.550931, 0.441433, 0.194863, -0.194863, 0.000000
0.100000, -1.546272, 0.490470, 0.240561, -0.240561, 0.000000
0.110000, -1.541122, 0.539502, 0.291063, -0.291063, 0.000000
0.120000, -1.535482, 0.588527, 0.346364, -0.346364, 0.000000
0.130000, -1.529351, 0.637540, 0.406458, -0.406458, 0.000000
0.140000, -1.522731, 0.686541, 0.471339, -0.471339, 0.000000
0.150000, -1.515621, 0.735526, 0.540999, -0.540999, 0.000000
0.160000, -1.508021, 0.784491, 0.615426, -0.615426, 0.000000
0.170000, -1.499931, 0.833431, 0.694608, -0.694608, 0.000000
0.180000, -1.491352, 0.882343, 0.778529, -0.778529, 0.000000
0.190000, -1.482284, 0.931220, 0.867171, -0.867171, 0.000000
0.200000, -1.472728, 0.980057, 0.960511, -0.960511, 0.000000
0.210000, -1.462683, 1.028846, 1.058525, -1.058525, 0.000000
0.220000, -1.452151, 1.077581, 1.161182, -1.161182, 0.000000
0.230000, -1.441132, 1.126254, 1.268448, -1.268448, 0.000000
0.240000, -1.429626, 1.174855, 1.380284, -1.380284, 0.000000
0.250000, -1.417635, 1.223375, 1.496645, -1.496645, 0.000000
0.260000, -1.405159, 1.271803, 1.617482, -1.617482, 0.000000
0.270000, -1.392199, 1.320128, 1.742738, -1.742738, 0.000000
0.280000, -1.378757, 1.368338, 1.872349, -1.872349, 0.000000
0.290000, -1.364833, 1.416420, 2.006246, -2.006246, 0.000000
0.300000, -1.350429, 1.464360, 2.144350, -2.144350, 0.000000
0.310000, -1.335546, 1.512143, 2.286575, -2.286575, 0.000000
0.320000, -1.320187, 1.559752, 2.432828, -2.432828, 0.000000
0.330000, -1.304352, 1.607172, 2.583003, -2.583003, 0.000000
0.340000, -1.288044, 1.654385, 2.736989, -2.736989, 0.000000
0.350000, -1.271265, 1.701370, 2.894661, -2.894661, 0.000000
0.360000, -1.254017, 1.748110, 3.055888, -3.055888, 0.000000
0.370000, -1.236304, 1.794582, 3.220525, -3.220525, 0.000000
0.380000, -1.218127, 1.840765, 3.388418, -3.388418, 0.000000
0.390000, -1.199489, 1.886637, 3.559398, -3.559398, 0.000000
0.400000, -1.180395, 1.932172, 3.733289, -3.733289, 0.000000
0.410000, -1.160847, 1.977347, 3.909901, -3.909901, 0.000000
0.420000, -1.140849, 2.022135, 4.089030, -4.089030, 0.000000
0.430000, -1.120406, 2.066510, 4.270462, -4.270462, 0.000000
0.440000, -1.099521, 2.110443, 4.453968, -4.453968, 0.000000
0.450000, -1.078198, 2.153906, 4.639311, -4.639311, 0.000000
0.460000, -1.056444, 2.196869, 4.826235, -4.826235, 0.000000
0.470000, -1.034263, 2.239303, 5.014476, -5.014476, 0.000000
0.480000, -1.011660, 2.281174, 5.203756, -5.203756, 0.000000
0.490000, -0.988641, 2.322452, 5.393785, -5.393785, 0.000000
0.500000, -0.965213, 2.363104, 5.584261, -5.584261, 0.000000
0.510000, -0.941381, 2.403096, 5.774871, -5.774871, 0.000000
0.520000, -0.917153, 2.442394, 5.965289, -5.965289, 0.000000
0.530000, -0.892536, 2.480964, 6.155180, -6.155180, 0.000000
0.540000, -0.867537, 2.518769, 6.344199, -6.344199, 0.000000
0.550000, -0.842163, 2.555776, 6.531992, -6.531992, 0.000000
0.560000, -0.816424, 2.591948, 6.718197, -6.718197, 0.000000
0.570000, -0.790327, 2.627250, 6.902442, -6.902442, 0.000000
0.580000, -0.763882, 2.661645, 7.084352, -7.084352, 0.000000
0.590000, -0.737097, 2.695097, 7.263546, -7.263546, 0.000000
0.600000, -0.709983, 2.727570, 7.439637, -7.439637, 0.000000
0.610000, -0.682549, 2.759028, 7.612237, -7.612237, 0.000000
0.620000, -0.654806, 2.789437, 7.780958, -7.780958, 0.000000
0.630000, -0.626764, 2.818760, 7.945409, -7.945409, 0.000000
0.640000, -0.598435, 2.846964, 8.105202, -8.105202, 0.000000
0.650000, -0.569829, 2.874014, 8.259954, -8.259954, 0.000000
0.660000, -0.540958, 2.899877, 8.409284, -8.409284, 0.000000
0.670000, -0.511835, 2.924520, 8.552819, -8.552819, 0.000000
0.680000, -0.482472, 2.947914, 8.690194, -8.690194, 0.000000
0.690000, -0.452881, 2.970026, 8.821054, -8.821054, 0.000000
0.700000, -0.423076, 2.990829, 8.945055, -8.945055, 0.000000
0.710000, -0.393069, 3.010294, 9.061868, -9.061868, 0.000000
0.720000, -0.362875, 3.028395, 9.171175, -9.171175, 0.000000
0.730000, -0.332506, 3.045108, 9.272680, -9.272680, 0.000000
0.740000, -0.301977, 3.060409, 9.366100, -9.366100, 0.000000
0.750000, -0.271303, 3.074276, 9.451176, -9.451176, 0.000000
0.760000, -0.240497, 3.086692, 9.527666, -9.527666, 0.000000
0.770000, -0.209574, 3.097637, 9.595354, -9.595354, 0.000000
0.780000, -0.178549, 3.107096, 9.654045, -9.654045, 0.000000
0.790000, -0.147437, 3.115055, 9.703570, -9.703570, 0.000000
0.800000, -0.116253, 3.121504, 9.743785, -9.743785, 0.000000
0.810000, -0.085012, 3.126431, 9.774573, -9.774573, 0.000000
0.820000, -0.053729, 3.129831, 9.795844, -9.795844, 0.000000
0.830000, -0.022420, 3.131698, 9.807535, -9.807535, 0.000000
0.840000, 0.008900, 3.132030, 9.809612, -9.809612, 0.000000
0.850000, 0.040215, 3.130826, 9.802068, -9.802068, 0.000000
0.860000, 0.071511, 3.128087, 9.784927, -9.784927, 0.000000
0.870000, 0.102772, 3.123818, 9.758239, -9.758239, 0.000000
0.880000, 0.133982, 3.118025, 9.722081, -9.722081, 0.000000
0.890000, 0.165127, 3.110717, 9.676559, -9.676559, 0.000000
0.900000, 0.196192, 3.101904, 9.621806, -9.621806, 0.000000
0.910000, 0.227160, 3.091598, 9.557980, -9.557980, 0.000000
0.920000, 0.258019, 3.079816, 9.485264, -9.485264, 0.000000
0.930000, 0.288752, 3.066572, 9.403867, -9.403867, 0.000000
0.940000, 0.319345, 3.051888, 9.314018, -9.314018, 0.000000
0.950000, 0.349785, 3.035782, 9.215970, -9.215970, 0.000000
0.960000, 0.380056, 3.018277, 9.109995, -9.109995, 0.000000
0.970000, 0.410146, 2.999397, 8.996385, -8.996385, 0.000000
0.980000, 0.440040, 2.979169, 8.875448, -8.875448, 0.000000
0.990000, 0.469725, 2.957619, 8.747507, -8.747507, 0.000000
1.000000, 0.499188, 2.934775, 8.612902, -8.612902, 0.000000
Columns
      t,         q,       qd,       ke,        pe,      ke+pe-(kepe0)
Wrote: ../output/meshup.csv (meshup animation file)
Wrote: ../output/kepe.csv (simulation data)
