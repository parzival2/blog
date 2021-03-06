---
author: "Kalyan"
title: "QtCreator with ROS"
date: "2021-03-05"
description: "Setting up QtCreator to work with ROS"
tags: ["ros", "c++", "qtcreator", "robotics"]
ShowToc: true
---

I have tried many IDEs and found that QtCreator is the one that I like. So I wanted to setup QtCreator for ROS development. 
## Prerequisites
### Installing ROS
I assume that you have already installed ROS on your operating system. If not you can follow these [instructions](http://wiki.ros.org/noetic/Installation/Ubuntu)
### Installing QtCreator
An open-sourced version of QtCreator can be downloaded from [here](https://www.qt.io/offline-installers). From here you can only download QtCreator IDE without downloading the Qt framework.
## Create Workspace and Compiling Our Package
Create a catkin-workspace if you haven't done so by running 

```bash
mkdir catkin_ws
cd catkin_ws
mkdir src
catkin_make
```

Then create a package using

```bash
catkin_create_pkg package_name std_msgs roscpp
```

Finally compile the package using 

```bash
catkin_make
```

## Setting up QtCreator

Actually, there is little to setup. The QtCreator works with ROS out of the box but I had some trouble. It also messes up the ROS build environment too which will have been created using catkin-make. For this step to work, we need to build the project at least once so that QtCreator can find the correct build folders. 
For QtCreator to load the environment variables provided in `~/.bashrc`, we need to launch it by creating a custom `.desktop` file. 

Paste the following contents in to a file on the desktop named appropriately, `QtCreator.desktop`

```
[Desktop Entry]
Type=Application
Exec=bash -i -c "/home/kalyan/Qt/Tools/QtCreator/bin/qtcreator" %F
Name=Qt Creator
GenericName=The IDE of choice for Qt development.
Icon=QtProject-qtcreator
StartupWMClass=qtcreator
Terminal=false
Categories=Development;IDE;Qt;
MimeType=text/x-c++src;text/x-c++hdr;text/x-xsrc;application/x-designer;application/vnd.qt.qmakeprofile;application/vnd.qt.xml.resource;text/x-qml;text/x-qt.qml;text/x-qt.qbs;
```

Right-click on the created file and choose `Allow Launching`. 

I guess this is a new addition in Ubuntu20.04. As soon as you do that, you can launch QtCreator by double-clicking on it.

Launch the QtCreator and open the `CMakeLists.txt` in `~catkin_ws\src` folder. 

Now, de-select all the Temporary Kit and select the `QtDesktop Kit` that you have already setup. 

Setup the build type of your choice and most importantly select the `build` folder found in your workspace as the path to the build folder as shown in the following screenshot.

![QtCreator build folder selection](qtcreator-buildfolder.png)

---

## References

[Qt](https://www.qt.io/product/development-tools)

[ROS](https://www.ros.org/)

[ROS IDEs](http://wiki.ros.org/IDEs)