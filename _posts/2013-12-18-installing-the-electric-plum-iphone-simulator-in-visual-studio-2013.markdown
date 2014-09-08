---
layout: post
title: "Installing the Electric Plum iPhone Simulator in Visual Studio 2013"
date: 2013-12-18 10:51:36 -0400
comments: true
categories: 
---

I'm going to file this under "Late to the Party Again", but I just learned that Microsoft offers a free version of the [Electric Plum](http://www.electricplum.com/) iPhone/iPad Simulator through Web Matrix. The version is a lite version, but well worth the free download.

1. First, if you don't already have it, install and run [Web Matrix](http://www.microsoft.com/web/webmatrix/).
2. Once Web Matrix is open, create a new site (I chose the empty site template).
3. On the tool bar, click on the `Extensions` button.
4. Search for the `iPhone Simulator` if it's not already on the screen.
5. Click the `Install` button and click through to install the simulator.
6. You may close Web Matrix once the installation is complete.
7. Start up Visual Studio 2013 and open your solution
8. Click the down arrow next to the `Play` button
9. Click the `Browse With` menu item
10. In the `Browse With` dialog, click the `Add` button
11. In the `Add Program` dialog, navigate to `C:\Users\<your account name>\AppData\Local\Microsoft\WebMatrix\Extensions\30\iPhoneSimulator\ElectricMobileSim\ElectricMobileSim.exe` in the program field.
12. In the `Arguments` field, enter `"1"` to simulate an iPhone, or `"2"` to simulate an iPad.
13. Click the `OK` button
14. Now you may select your new emulator from the list of browsers.

