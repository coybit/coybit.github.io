---
layout: post
comments: true
title:  "SwiftUI: Double-Edged Sword"
categories: 
tags: swiftui uikit architecture
excerpt_separator: <!--more-->
---
One reason that I favor SwiftUI over UIKit is that SwiftUI is architecture-less.
<!--more-->
That is, SwiftUI doesn’t assume what architecture you are going to use. In contrast to UIKit that assumes you build your app using the MVC pattern. Whatever architecture you come up with, eventually, MVC pattern creeps in somewhere somehow. Regardless of how consistent and flawless your architecture is, you are doomed to interact with UIKit, and these surfaces of interaction are where MVC pattern leaks into your code.
On the other hand, in SwiftUI you are completely free to choose the architecture you are comfortable with and don’t need to be worried about the Apple preference. This freedom allows you to build a more coherent and uniform architecture. Although, in lack of a scaffold, as we used to have in UIKit, this time instead of ending up with Massive View Controller, we may slide into the [Big ball of mud](https://en.wikipedia.org/wiki/Big_ball_of_mud). More freedom comes at the cost of more responsibility.

![](https://github.com/coybit/coybit.github.io/raw/master/assets/uikit/uikit.png)