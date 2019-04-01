author Matthew Millard
date   : 27 March 2019
version: 0.1

Note: ContactToolkit.h is not yet a part of RBDL, but I am planning on putting 
it in its own addon later this year.

================================================================================
Quick Start
================================================================================

This is an illustrative example to show how to compute and apply contact and 
friction forces to a bouncing ball. To get started:

1. Make a 'build' folder.

2. Run cmake from the 'build' folder. You will be asked to set CUSTOM_RBDL_PATH,
   which is the path to the install folder of the version of RBDL that you 
   would like to use. 

    If you have a system-wide installation of RBDL this folder is '/usr/local'

3. From a terminal in the build directory call 
    make

4. From a terminal in the build directory run

    ./bouncingBallBenchmark

5. If everything is working you should see simulation data (see 'Example Output') 
   printed to the screen and 4 files will be written to the output folder.
   One file is printed prior to the simulation, while 3 are printed after the
   simulation.

6. To view an animation, from a terminal in the 'bouncingBall' folder run

    meshup model/ballPlaneContact.lua output/animation.csv output/animationForces.ff



================================================================================
High Level Overview
================================================================================

Before getting started be aware:
  
  Time commitment: 
    30-45 minutes


1. The state of the ball is used to evaluate the position and velocity of the 
point of contact between a sphere and a plane. This is a compliant model so the 
sphere is allowed to penetrate the plane. The point of contact is the point on 
the sphere that is deepest into the plane.

2. The kinematics of the contact point in the normal direction are used to 
compute contact forces using a Hunt-Crossley contact model. 

3. The tangential velocity of the contact point is used to compute a 
coefficient of friction using a regularized friction model. This coefficient of 
friction is used to compute friction forces. 

4. Normal and friction forces are put into the f_ext vector into the proper 
location. 

5. RBDL's ForwardDynamics function is used to calculate the state derivative

6. Numerical integration is used to compute a state trajectory. The sum of 
system energy less work is tracked: if everything has been done correctly this 
will remain mostly constant, only changing as a function of numerical error 
from the integrator.

================================================================================
Detailed Code Tour
================================================================================

There are numbered comments in the files which correspond to those below. Go
to those comments in the files for further details

In 'bouncingBallBenchmark.cc'

0a. ballPlaneContact.lua model is read in
0b. Contact and friction parameters are set
0c. An object that Boost can integrate is instantiated:

1a. The regularized friction model is created and printed to file
    ('../output/frictionCoefficientCurve.csv'.)
    To understand more about this (relatively simple) friction model please
    read the documentation that appears in ContactToolkit.h above
    the definition for createRegularizedFrictionCoefficientCurve.

1b. The state derivative for this model is called by Boost whenever the 
    code for the 'operator()' function is called

1c. The state x is split into generalized positions (q), generalized
    velocities (qdd). Here tau is set to 0 because we are not 
    applying any generalized forces: the ball contact/friction forces
    are applied as external forces

2a. To evaluate the contact forces we must see if the ball is penetrating
    the ground. Outside of FEA modeling it is common to allow (non deforming)
    geometry to penetrate and to use the penetration to compute
    contact forces. Here we go with the simplest option and map the 
    depth of penetration to a contact force.

    If the ascii notation used to describe vectors (e.g. r0K0), unit vectors
    (e.g. eNO), and matrices (e.g EBO) is new to you, please refer to the 
    notation guide that appears in this class just above where the member
    variables are declared.

2b. If the sphere is in contact with the plane, we must get the position and
    velocity of the contact point.

2c. Now we can evaluate the contact forces using a Hunt-Crossley contact model
    This is a popular contact model because it is numerically well behaved.
    See the comments in ContactToolkit.h above the function definition
    for calcHuntCrossleyContactForce for further details.

2d. To apply this contact force as an external force in RBDL, we must first
    transform it into a spatial wrench that is resolved in the Root frame.
    Here we turn the scalar contact force into a vector, and then evaluate
    the torque that this force vector produces about the Root frame  

2e. Now we can use the tangential velocity of the contact point of the ball
    to evaluate the coefficient of friction, and then the friction forces

2f. As with the contact model we evaluate the force the friction model applies
    to the ball and also the torque it generates about the ROOT frame

2g. The total wrench generated by the contact and friction models is applied to
    the entry in the vector fext that corresponds to the ball.    

3a. Now the generalized accelerations of the ball can be computed

3b. The state derivative dxdt is now formed using qd, and qdd. The derivatives 
    of the work done by the contact and friction models is also stored so that 
    we can track the system energy of the system through the simulation

4a. The model state is initialized

4b. This model is integrated forward in time from t0 to t1 and is evaluated
    at npts between these time points

4c. At each time point the q's, qd's, and the work done on the ball by the 
    contact & friction model is saved


4d. The q's, qd's, and work terms are used to numerically evaluate the system
    energy of the ball less the work done on it: ke+pe-w, where 
    ke is kinetic energy, pe is potential energy, and w is work.

4e. This is evaluated relative to the system energy of the ball at the 
    beginning of the simulation. If the integrator were perfect the sum of 
    ke+pe-w-kepe0 would be numerically zero. Because numerical integration is
    not perfect this quantity (printed to screen) will drift over time.
    You can see this if you adjust the absolute and relative tolerances
    (set between 4b and 4c) on the integrator and re-run the simulations.  

4f. Here we grab quantities that we would like to save

5a. Finally we write the simulation data to file

================================================================================
Example Output
================================================================================
Wrote: ../output/frictionCoefficentCurve.csv
Columns below:
          t,        theta,   d/dt theta,           ke,           pe,            w, ke+pe-w-kepe0
0.000000e+00, 0.000000e+00, 0.000000e+00, 5.000000e-01, 9.810000e+00, 0.000000e+00, 0.000000e+00
1.010101e-02, 0.000000e+00, 0.000000e+00, 5.049095e-01, 9.805090e+00, 0.000000e+00, -1.776357e-15
2.020202e-02, 0.000000e+00, 0.000000e+00, 5.196380e-01, 9.790362e+00, 0.000000e+00, 0.000000e+00
3.030303e-02, 0.000000e+00, 0.000000e+00, 5.441855e-01, 9.765814e+00, 0.000000e+00, 1.776357e-15
4.040404e-02, 0.000000e+00, 0.000000e+00, 5.785521e-01, 9.731448e+00, 0.000000e+00, 1.776357e-15
5.050505e-02, 0.000000e+00, 0.000000e+00, 6.227376e-01, 9.687262e+00, 0.000000e+00, 0.000000e+00
6.060606e-02, 0.000000e+00, 0.000000e+00, 6.767421e-01, 9.633258e+00, 0.000000e+00, 1.776357e-15
7.070707e-02, 0.000000e+00, 0.000000e+00, 7.405657e-01, 9.569434e+00, 0.000000e+00, -1.776357e-15
8.080808e-02, 0.000000e+00, 0.000000e+00, 8.142083e-01, 9.495792e+00, 0.000000e+00, -1.776357e-15
9.090909e-02, 0.000000e+00, 0.000000e+00, 8.976698e-01, 9.412330e+00, 0.000000e+00, -3.552714e-15
1.010101e-01, 0.000000e+00, 0.000000e+00, 9.909504e-01, 9.319050e+00, 0.000000e+00, -3.552714e-15
1.111111e-01, 0.000000e+00, 0.000000e+00, 1.094050e+00, 9.215950e+00, 0.000000e+00, -3.552714e-15
1.212121e-01, 0.000000e+00, 0.000000e+00, 1.206969e+00, 9.103031e+00, 0.000000e+00, -3.552714e-15
1.313131e-01, 0.000000e+00, 0.000000e+00, 1.329706e+00, 8.980294e+00, 0.000000e+00, -1.776357e-15
1.414141e-01, 0.000000e+00, 0.000000e+00, 1.462263e+00, 8.847737e+00, 0.000000e+00, -1.776357e-15
1.515152e-01, 0.000000e+00, 0.000000e+00, 1.604638e+00, 8.705362e+00, 0.000000e+00, 1.776357e-15
1.616162e-01, 0.000000e+00, 0.000000e+00, 1.756833e+00, 8.553167e+00, 0.000000e+00, -1.776357e-15
1.717172e-01, 0.000000e+00, 0.000000e+00, 1.918847e+00, 8.391153e+00, 0.000000e+00, -1.776357e-15
1.818182e-01, 0.000000e+00, 0.000000e+00, 2.090679e+00, 8.219321e+00, 0.000000e+00, -5.329071e-15
1.919192e-01, 0.000000e+00, 0.000000e+00, 2.272331e+00, 8.037669e+00, 0.000000e+00, -5.329071e-15
2.020202e-01, 0.000000e+00, 0.000000e+00, 2.463802e+00, 7.846198e+00, 0.000000e+00, -5.329071e-15
2.121212e-01, 0.000000e+00, 0.000000e+00, 2.665091e+00, 7.644909e+00, 0.000000e+00, -3.552714e-15
2.222222e-01, 0.000000e+00, 0.000000e+00, 2.876200e+00, 7.433800e+00, 0.000000e+00, -3.552714e-15
2.323232e-01, 0.000000e+00, 0.000000e+00, 3.097128e+00, 7.212872e+00, 0.000000e+00, -1.776357e-15
2.424242e-01, 0.000000e+00, 0.000000e+00, 3.327874e+00, 6.982126e+00, 0.000000e+00, 1.776357e-15
2.525253e-01, 0.000000e+00, 0.000000e+00, 3.568440e+00, 6.741560e+00, 0.000000e+00, 1.776357e-15
2.626263e-01, 0.000000e+00, 0.000000e+00, 3.818825e+00, 6.491175e+00, 0.000000e+00, 3.552714e-15
2.727273e-01, 0.000000e+00, 0.000000e+00, 4.079029e+00, 6.230971e+00, 0.000000e+00, 1.776357e-15
2.828283e-01, 0.000000e+00, 0.000000e+00, 4.349051e+00, 5.960949e+00, 0.000000e+00, 3.552714e-15
2.929293e-01, 0.000000e+00, 0.000000e+00, 4.628893e+00, 5.681107e+00, 0.000000e+00, 1.776357e-15
3.030303e-01, 0.000000e+00, 0.000000e+00, 4.918554e+00, 5.391446e+00, 0.000000e+00, 5.329071e-15
3.131313e-01, 0.000000e+00, 0.000000e+00, 5.218033e+00, 5.091967e+00, 0.000000e+00, 5.329071e-15
3.232323e-01, 9.060691e-05, 9.150213e-02, 5.426510e+00, 4.782923e+00, -1.005667e-01, 3.977149e-09
3.333333e-01, 9.299352e-03, 1.428571e+00, 2.783872e+00, 4.502157e+00, -3.023971e+00, 2.357456e-10
3.434343e-01, 2.372937e-02, 1.428571e+00, 3.861658e-01, 4.400655e+00, -5.523180e+00, 2.554227e-09
3.535354e-01, 3.815938e-02, 1.428571e+00, 2.455625e+00, 4.525857e+00, -3.328517e+00, -7.294245e-09
3.636364e-01, 5.258939e-02, 1.428571e+00, 3.590998e+00, 4.761831e+00, -1.957172e+00, -5.166241e-09
3.737374e-01, 6.701941e-02, 1.428571e+00, 3.417157e+00, 5.011459e+00, -1.881384e+00, 1.069337e-09
3.838384e-01, 8.144942e-02, 1.428571e+00, 3.176929e+00, 5.251687e+00, -1.881384e+00, 1.069333e-09
3.939394e-01, 9.587943e-02, 1.428571e+00, 2.946519e+00, 5.482097e+00, -1.881384e+00, 1.069333e-09
4.040404e-01, 1.103094e-01, 1.428571e+00, 2.725929e+00, 5.702687e+00, -1.881384e+00, 1.069330e-09
4.141414e-01, 1.247395e-01, 1.428571e+00, 2.515157e+00, 5.913458e+00, -1.881384e+00, 1.069330e-09
4.242424e-01, 1.391695e-01, 1.428571e+00, 2.314205e+00, 6.114411e+00, -1.881384e+00, 1.069330e-09
4.343434e-01, 1.535995e-01, 1.428571e+00, 2.123072e+00, 6.305544e+00, -1.881384e+00, 1.069326e-09
4.444444e-01, 1.680295e-01, 1.428571e+00, 1.941757e+00, 6.486859e+00, -1.881384e+00, 1.069326e-09
4.545455e-01, 1.824595e-01, 1.428571e+00, 1.770262e+00, 6.658354e+00, -1.881384e+00, 1.069326e-09
4.646465e-01, 1.968895e-01, 1.428571e+00, 1.608586e+00, 6.820030e+00, -1.881384e+00, 1.069326e-09
4.747475e-01, 2.113195e-01, 1.428571e+00, 1.456728e+00, 6.971888e+00, -1.881384e+00, 1.069326e-09
4.848485e-01, 2.257496e-01, 1.428571e+00, 1.314690e+00, 7.113926e+00, -1.881384e+00, 1.069330e-09
4.949495e-01, 2.401796e-01, 1.428571e+00, 1.182470e+00, 7.246145e+00, -1.881384e+00, 1.069330e-09
5.050505e-01, 2.546096e-01, 1.428571e+00, 1.060070e+00, 7.368546e+00, -1.881384e+00, 1.069330e-09
5.151515e-01, 2.690396e-01, 1.428571e+00, 9.474888e-01, 7.481127e+00, -1.881384e+00, 1.069330e-09
5.252525e-01, 2.834696e-01, 1.428571e+00, 8.447264e-01, 7.583889e+00, -1.881384e+00, 1.069330e-09
5.353535e-01, 2.978996e-01, 1.428571e+00, 7.517831e-01, 7.676833e+00, -1.881384e+00, 1.069330e-09
5.454545e-01, 3.123296e-01, 1.428571e+00, 6.686588e-01, 7.759957e+00, -1.881384e+00, 1.069330e-09
5.555556e-01, 3.267597e-01, 1.428571e+00, 5.953535e-01, 7.833262e+00, -1.881384e+00, 1.069330e-09
5.656566e-01, 3.411897e-01, 1.428571e+00, 5.318672e-01, 7.896749e+00, -1.881384e+00, 1.069330e-09
5.757576e-01, 3.556197e-01, 1.428571e+00, 4.781999e-01, 7.950416e+00, -1.881384e+00, 1.069333e-09
5.858586e-01, 3.700497e-01, 1.428571e+00, 4.343517e-01, 7.994264e+00, -1.881384e+00, 1.069337e-09
5.959596e-01, 3.844797e-01, 1.428571e+00, 4.003224e-01, 8.028293e+00, -1.881384e+00, 1.069337e-09
6.060606e-01, 3.989097e-01, 1.428571e+00, 3.761121e-01, 8.052504e+00, -1.881384e+00, 1.069337e-09
6.161616e-01, 4.133397e-01, 1.428571e+00, 3.617209e-01, 8.066895e+00, -1.881384e+00, 1.069337e-09
6.262626e-01, 4.277698e-01, 1.428571e+00, 3.571486e-01, 8.071467e+00, -1.881384e+00, 1.069333e-09
6.363636e-01, 4.421998e-01, 1.428571e+00, 3.623954e-01, 8.066220e+00, -1.881384e+00, 1.069330e-09
6.464646e-01, 4.566298e-01, 1.428571e+00, 3.774612e-01, 8.051155e+00, -1.881384e+00, 1.069330e-09
6.565657e-01, 4.710598e-01, 1.428571e+00, 4.023460e-01, 8.026270e+00, -1.881384e+00, 1.069330e-09
6.666667e-01, 4.854898e-01, 1.428571e+00, 4.370498e-01, 7.991566e+00, -1.881384e+00, 1.069333e-09
6.767677e-01, 4.999198e-01, 1.428571e+00, 4.815726e-01, 7.947043e+00, -1.881384e+00, 1.069333e-09
6.868687e-01, 5.143498e-01, 1.428571e+00, 5.359144e-01, 7.892701e+00, -1.881384e+00, 1.069333e-09
6.969697e-01, 5.287799e-01, 1.428571e+00, 6.000752e-01, 7.828541e+00, -1.881384e+00, 1.069333e-09
7.070707e-01, 5.432099e-01, 1.428571e+00, 6.740550e-01, 7.754561e+00, -1.881384e+00, 1.069333e-09
7.171717e-01, 5.576399e-01, 1.428571e+00, 7.578538e-01, 7.670762e+00, -1.881384e+00, 1.069333e-09
7.272727e-01, 5.720699e-01, 1.428571e+00, 8.514717e-01, 7.577144e+00, -1.881384e+00, 1.069330e-09
7.373737e-01, 5.864999e-01, 1.428571e+00, 9.549085e-01, 7.473707e+00, -1.881384e+00, 1.069330e-09
7.474747e-01, 6.009299e-01, 1.428571e+00, 1.068164e+00, 7.360451e+00, -1.881384e+00, 1.069326e-09
7.575758e-01, 6.153599e-01, 1.428571e+00, 1.191239e+00, 7.237377e+00, -1.881384e+00, 1.069326e-09
7.676768e-01, 6.297900e-01, 1.428571e+00, 1.324133e+00, 7.104483e+00, -1.881384e+00, 1.069326e-09
7.777778e-01, 6.442200e-01, 1.428571e+00, 1.466846e+00, 6.961770e+00, -1.881384e+00, 1.069326e-09
7.878788e-01, 6.586500e-01, 1.428571e+00, 1.619378e+00, 6.809238e+00, -1.881384e+00, 1.069326e-09
7.979798e-01, 6.730800e-01, 1.428571e+00, 1.781729e+00, 6.646887e+00, -1.881384e+00, 1.069326e-09
8.080808e-01, 6.875100e-01, 1.428571e+00, 1.953899e+00, 6.474717e+00, -1.881384e+00, 1.069326e-09
8.181818e-01, 7.019400e-01, 1.428571e+00, 2.135888e+00, 6.292728e+00, -1.881384e+00, 1.069326e-09
8.282828e-01, 7.163700e-01, 1.428571e+00, 2.327696e+00, 6.100920e+00, -1.881384e+00, 1.069326e-09
8.383838e-01, 7.308000e-01, 1.428571e+00, 2.529322e+00, 5.899293e+00, -1.881384e+00, 1.069322e-09
8.484848e-01, 7.452301e-01, 1.428571e+00, 2.740768e+00, 5.687847e+00, -1.881384e+00, 1.069322e-09
8.585859e-01, 7.596601e-01, 1.428571e+00, 2.962033e+00, 5.466582e+00, -1.881384e+00, 1.069322e-09
8.686869e-01, 7.740901e-01, 1.428571e+00, 3.193117e+00, 5.235498e+00, -1.881384e+00, 1.069326e-09
8.787879e-01, 7.885201e-01, 1.428571e+00, 3.434020e+00, 4.994596e+00, -1.881384e+00, 1.069326e-09
8.888889e-01, 8.029501e-01, 1.428571e+00, 3.505829e+00, 4.744985e+00, -2.059186e+00, 3.681732e-08
8.989899e-01, 8.173801e-01, 1.428571e+00, 1.663603e+00, 4.528897e+00, -4.117500e+00, 2.075402e-08
9.090909e-01, 8.318101e-01, 1.428571e+00, 3.965162e-01, 4.460354e+00, -5.453129e+00, 2.197916e-08
9.191919e-01, 8.462402e-01, 1.428571e+00, 1.793972e+00, 4.566752e+00, -3.949276e+00, 1.516591e-08
9.292929e-01, 8.606702e-01, 1.428571e+00, 2.599091e+00, 4.762423e+00, -2.948486e+00, 1.570672e-08
9.393939e-01, 8.751002e-01, 1.428571e+00, 2.470217e+00, 4.970411e+00, -2.869372e+00, 2.215064e-08
9.494949e-01, 8.895302e-01, 1.428571e+00, 2.271419e+00, 5.169209e+00, -2.869372e+00, 2.215064e-08
9.595960e-01, 9.039602e-01, 1.428571e+00, 2.082440e+00, 5.358188e+00, -2.869372e+00, 2.215065e-08
9.696970e-01, 9.183902e-01, 1.428571e+00, 1.903281e+00, 5.537347e+00, -2.869372e+00, 2.215064e-08
9.797980e-01, 9.328202e-01, 1.428571e+00, 1.733940e+00, 5.706688e+00, -2.869372e+00, 2.215065e-08
9.898990e-01, 9.472503e-01, 1.428571e+00, 1.574419e+00, 5.866209e+00, -2.869372e+00, 2.215065e-08
1.000000e+00, 9.616803e-01, 1.428571e+00, 1.424716e+00, 6.015912e+00, -2.869372e+00, 2.215064e-08
1.010101e+00, 9.761103e-01, 1.428571e+00, 1.284833e+00, 6.155795e+00, -2.869372e+00, 2.215064e-08
Columns above:
          t,        theta,   d/dt theta,           ke,           pe,            w, ke+pe-w-kepe0

Wrote: ../output/animation.csv (meshup animation file)
Wrote: ../output/animationForces.ff (meshup force file)
Wrote: ../output/kepe.csv (simulation data)
