# Notes & Thoughts on The Sim-To-Real Gap

## [Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World](Domain Randomization for Transferring Deep Neural Networks from Simulation to the Real World.pdf)

Published in 2017 by OpenAI, this work was the *first* deep neural network to *successfully* transfer trained on solely simulated RGB images.

The goal of **domain randomisation** is to create a simulated data set which is diverse enough such that the real world appears as though it were another variation. As a keystone technique in closing the sim-to-real gap, it enables the application of deep learning to robotics through enormous simulated data sets. Additionally, simulated data can much more easily be acquired for failure modes such as physical crashes or overexertion which can prove either costly or dangerous in real life.

Traditional techniques involve the *system identification* step, where parameters of the simulation are tuned to match the behaviour of reality as closely as possible. This process is both time consuming and does not include *unmodeled physical effects* such as non-rigidity, gear backlash, and wear-and-tear. This is referred to as the *reality gap*.

To test the concept of domain randomisation for sim-to-real transfer, they applied the technique to object localisation from RGB imagery. Ultimately, the detector was accurate to within 1.5 cm in the real world with no need for pre-training on real-world data and robotic grippers successfully grasped objects located at the coordinates they indicated.

By varying the number and type of objects in the scene, their positions, textures, and orientation, number and properties of lights in the scene,  and the amount of random noise added to the images, solid sim-to-real transfer was achieved without pre-training on real-world data using a deep convolutional neural network.

An ablation study was performed to determine the sensitivity of the resulting model to various factors. Most interestingly, adding noise appeared to have a negligible effect.

## [Learning Agile and Dynamic Motor Skills for Legged Robots](Learning Agile and Dynamic Motor Skills for Legged Robots.pdf)

Published in 2019, this work developed a novel method for training quadrupedal robots in simulation using RL which successfully transfers to real world hardware. It has been cited nearly 2000 times as of February, 2026.

### Method

A previously-existing rigid-body simulator with intermittent contact support was used for this work.

Domain randomisation was applied as follows, using 30 different ANYmal models with stochastically sampled inertial properties:

- COM positions, link masses, and joint positions were randomised by adding a noise sampled from U(-2,2) cm, U(-15,15) %, and U(-2,2) cm respectively.

#### Modelling The Actuation

Supervised learning was used to develop a model which provides an action-to-torque mapping. It produces an output of joint torques given a history  of position errors (the actual position subtracted from the commanded position) and velocities.

A history consisting of the current state and two past states (t - 0.01 and t - 0.02 seconds) is used. The length of the history is an important attribute to choose. It should be sufficiently longer than the sum of all communication delays and the mechanical response time. In practice, the exact input configuration is tuned based on the validation error.

## [Noise Is All You Need to Bridge The Sim-To-Real Locomotion Gap](https://news.asimov.inc/p/noise-is-all-you-need)

​	Menlo research are in the early stages of developing their open source bipedal humanoid, *Asimov*. Along the way they discovered that the leading factor in sim-to-real transfer success was not the accuracy of the MuJoCo physics platform used in their simulations, but the accurate modelling of the robot's firmware and communications stack *in simulation*.

This technique is called "Processor-in-the-Loop", which routes the control loop through the actual firmware stack including threading, timing, and numeric constraints even including an i2c emulator.

Some key takeaways:

- The *real* robot hardware stack is running, without knowing that it is in a simulation. It knows no difference whether it is talking to an actual or simulated i2c bus.
- Python math and C math often disagree, with Python apparently having infinite precision vs. the constrained floats of embedded C math which yield different answers.
- Modelling CANbus jitter in the control stack helps to catch bugs in CI (Continuous Integration) rather than on the robot, where it's incredibly difficult and time consuming to debug.

Discussing with them in their Discord server, the slowdown to run processor-in-the-loop during training was about 2 to 3x, which they deemed acceptable.