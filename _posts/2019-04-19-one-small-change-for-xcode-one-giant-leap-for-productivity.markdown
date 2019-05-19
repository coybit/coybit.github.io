---
layout: post
title:  "one small change for xcode, one giant leap for productivity"
categories: 
---
Last week I learned something that in the future can potentially save hours of my life that otherwise, I would have to spend for preparing an app for testing or taking screenshots. It was really simple and it has been always there, but I hadn't noticed till last week.
XCode allows you to take a snapshot of the state of the app you are developing and restore it later when you run the app or run the tests. Pretty cool! What it actually does is nothing more than giving you a copy of AppDate and then later on replacing the app's AppDate with the one you want to restore to. So if everything you are saving in AppDate is included in the snapshot. 

### How to take a snapshot?
It's easy. In xCode, go to **Windows > Devices and Simulators** and select the device which you want to take a snapshot of AppData of an app installed on it. Press the gear icon (App container actions) and choose **Download Container...**. Then you will be asked to choose a path and the snapshot ( a `.xcappdata` file ) will be saved there.

--IMG--

`.xcappdata` is a bundle file. To see what is inside it just **Right Click > Show Package Contents**:


### How to restore a snapshot
First of all, you need to import the .xcappdata file that you saved in the previous step into the project. Then go in xCode, go to **Product > Schema > Edit Schema**.

--IMG--

- If you want the snapshot to be restored every time the app runs:
From the left menu, click on **Run**, under Options tab in the right panel, from Application Data list, choose the snapshot you just added to the project
--IMG--

- If you want the snapshot to be restored every time the tests run:
From the left menu, click on **Test**, under Info tab in the right panel, in Tests list, click on **Option...** button of the test target and in the opened popover, from Application Date list, choose the snapshot you just added to the project.
--IMG--
