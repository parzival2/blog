---
author: "Kalyan"
title: "ROS connection between multiple machines on LAN"
date: "2021-03-06"
description: "Making ROS work over multiple machines that are in LAN"
tags: ["ros", "lan"]
ShowToc: true
---

I have an [orangepi zero](http://www.orangepi.org/orangepizero/) that I want to connect to a ROS instance running on my desktop to play around with manual control on PX4.
I have followed some instructions [here](http://wiki.ros.org/ROS/Tutorials/MultipleMachines) but they appear not to work for me. The orangepi always says that there is no `roscore` running. 
I searched and searched and I came across this [page](https://answers.ros.org/question/272065/specification-of-ros_master_uri-and-ros_hostname/?answer=298712#post-id-298712) that seems to provide correct instructions.
I did it like in the below-mentioned procedure and the orange pi seems to recognize the ROS instance running on my desktop.

### Master
```bash
export ROS_MASTER_URI=http://192.168.0.157:11311
export ROS_IP=192.168.0.157
roscore
```
### Orange pi
```bash
export ROS_MASTER_URI=http://192.168.0.157:11311
export ROS_IP=192.168.0.207
rostopic list
```
and now I get
```bash
/rosout
/rosout_agg
```

You can also put the first two lines in `~/.bashrc` but then you have to have a static IP address for both the machines.