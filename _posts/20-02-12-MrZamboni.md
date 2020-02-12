---
layout: post
title: MrZamboni
author: Mike Wessling
no-excerpt: Defaults to first page
---

The story. 
==========

On Monday 2 weeks before AWS Re:Invent a team meeting I suggested that it would be funny to turn the Roomba into Deepracer track cleaning robot.  How hard could it be? The basic principle of Deepracer is pretty straightforward: feed a 160x120 grayscale image into a Tensorflow CNN and it outputs an action probability vector. Then if I could match the camera image position and translate the actions into Roomba motor commands it should work in principle.

The team liked the idea a lot and thought it would be great to have it before AWS Re:invent.  It turned out to be quite a job to get it done in time and many times I thought that it would never work.. 

We had the basic components: 
* A Roomba 980
* an Nvidia Jetson Nano board 
* a pretty robust Deepracer Model, called MrDeepracer_2b_3hrs_Toronto.  

Nvidia has Jetson optimized Tensorflow APIs.  People suggested to use a Deeplens or to use the board of a Deepracer but I wanted to use completely different hardware.  Nvidia Jetson is base on an Arm64 processor instead of an Intel Atom processor. 

I tackled the problem in the following steps. 

1. Loading a trained model into Tensorflow, feed images from an actual Deepracer and compare the results.  I have added an ROS-MQTT bridge to one of our Deepracers and connected it to AWS IOT so I could subscribe and feed the messages through the TF CNN.  Figuring out which operation in the large graph created by RL_coach was used took a lot of time and a story by itself. 
2. Find a camera with a similar field of view as the Deepracer and mount it on the Roomba at the right height and angle.  In the end a Raspberry PI camera V2 was used.  Figuring out the actual angle of the Deepracer was difficult.  I found the angle used in the simulator but it not was wide as the physical camera.  I used large pieces of paper and markers to mark out what the cameras saw.  
3. Install ROS on the Jetson Board and create a backpack to hold the camera, Jetson and powerbank. 
4. Control the Roomba. This looked like an easy step; ROS has a package for iCreate robots and the Roomba 980 seemed to support the iRobot Open Interface. It turned out that there are subtle differences between the last published specification (V3) and the protocol used by the 980.. After fun day of wireshark USB traces and manual packet decoding I found 2 differences and patched the libraries. 
5. Translate the actions used by Deepracers (Speed and steering angle in an Ackermann model) into speed and radian/s used by the Roomba Twist interface.  I didn’t know how to do this until inspiration struck last Thursday evening. 

In the end I wrote 3 ROS nodes: 
1. Jetson_cam to read the camera an publish to /video_mjpeg.  (I used to same topics and messages because I wanted to be able to feed the images to a Deepracer and compare the results)
2. Mrinfer to do the inference. It loads the model. It subscribes to /video_mjpeg, pre-processes the images, feeds it through TF and publish the result vector to /rl_results 
3. Mrmoveit. It loads the action json file, subscribes to /rl_results, selects to action with the highest probability, calculates the speed and Radians/s needed and publishes it to /cmd_vel. This node is also responsible for starting and stopping of the autonomous mode. 

I added a joystick remote to control the Roomba and to switch between manual, autonomous and docking. It will auto dock using Roomba’s own mechanism. The Roomba is unmodified. 

The performance of the Jetson board is nice. It easily processes 15 frames per second and its power usage is low. It can run a whole day on a power bank. 

The maximum speed of a Roomba is 0.5m/s so it is a bit slower.  It does the Re:Invent 2018 track in 42sec. 

Amazingly enough it ran a complete clean lap the first time.  I had expected to spend quite some time to tweak the model and translation.  

I think I spend about 70-80 hrs on the whole project

Next steps: 
1. migrate ROS to python 3. It is now a mix of python2 and 3 because of the tensorflow libraries. This causes a lot of pain.  
2. Refactor the code and put it on github. 
3. Do a writeup on how to build your own deepracer.  
4. Add features: 
  * Object detection/tracking?
  * Use the odom data to track position. 
  * Add a simple UI.
  * Rewrite the mrinfer in cpp to be able to use the protobuf models?

PS. The “mr”-theme is a reference to spaceballs and mrcoffee. 
