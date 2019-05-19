---
layout: post
title:  "one small change for xcode, one giant leap for productivity"
categories: 
---
Last week I learned something that in the future can potentially save hours of my life that otherwise, I would have to spend for preparing an app for testing or taking screenshots. It was really simple and it has been always there, but I hadn't noticed till last week.
XCode allows you to take a snapshot of the state of the app you are developing and restore it later when you run the app or run the tests. Pretty cool! What it actually does is nothing more than giving you a copy of AppDate and then later on replacing the app's AppDate with the one you want to restore to. So if everything you are saving in AppDate is included in the snapshot. 

### How to take a snapshot?
It's easy. In xCode, go to **Windows > Devices and Simulators** and select the device which you want to take a snapshot of AppData of an app installed on it. Press the gear icon (App container actions) and choose **Download Container...**.

![](https://github.com/coybit/coybit.github.io/raw/master/assets/appdata-1.png)

Then you will be asked to choose a path and the snapshot ( a `.xcappdata` file ) will be saved there.

![](https://github.com/coybit/coybit.github.io/raw/master/assets/appdata-2.png)

`.xcappdata` is a bundle file. To see what is inside it just **Right Click > Show Package Contents**:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/appdata-3.png)

### How to restore a snapshot
First of all, you need to import the .xcappdata file that you saved in the previous step into the project. Then go in xCode, go to **Product > Schema > Edit Schema**.

- If you want the snapshot to be restored every time the app runs:
From the left menu, click on **Run**, under Options tab in the right panel, from Application Data list, choose the snapshot you just added to the project

![](https://github.com/coybit/coybit.github.io/raw/master/assets/appdata-4.png)


- If you want the snapshot to be restored every time the tests run:
From the left menu, click on **Test**, under Info tab in the right panel, in Tests list, click on **Option...** button of the test target and in the opened popover, from Application Date list, choose the snapshot you just added to the project.

![](https://github.com/coybit/coybit.github.io/raw/master/assets/appdata-5.png)


The next time the app runs, its AppData directory gets replaced with the AppData which the snapshot contains. Doesn't matter what how AppData directory contents change. Because as long as the snapshot is the same, every time the app runs as if it starts from the same state.

