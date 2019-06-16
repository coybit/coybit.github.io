---
layout: post
title:  "firebase sdk: one repository, multiple pods"
categories: firebase, pod, podspec
---
Honestly, when I learned how [Firebase SDK](https://github.com/firebase/firebase-ios-sdk) has been organized as a single repository that contains multiple pods, I found it really neat and efficient.

Firebase iOS SDK includes different components that can be used independently in your app, like FirebaseCore, FirebaseMessaging, FirebaseFirestore, ...

But how have they managed to do that?

The short answer is by utilizing [git tagging mechanism](https://git-scm.com/book/en/v2/Git-Basics-Tagging). Let look at one of the podsepc files, for example, [FirebaseCore.podspec](https://github.com/firebase/firebase-ios-sdk/blob/master/FirebaseCore.podspec). The interesting part is this part:

``` yaml
s.version = '6.0.0'

...

s.source = {
    :git => 'https://github.com/firebase/firebase-ios-sdk.git',
    :tag => 'Core-' + s.version.to_s
    }
```
This podspec points to a specific tag in this repository (Core-6.0.0). How is it different from other podspec files, say [FirebaseFirestore.podspec](https://github.com/firebase/firebase-ios-sdk/blob/master/FirebaseFirestore.podspec):
``` yaml
s.version = '1.2.1'

...

s.source =
    :git => 'https://github.com/firebase/firebase-ios-sdk.git',
    :tag => 'Firestore-' + s.version.to_s
    }
```

As you can see, this podspec points to another tag (Firestore-1.2.1). Now, let's take a look into [the list of tags of the repository](https://github.com/firebase/firebase-ios-sdk/tags):

![](https://github.com/coybit/coybit.github.io/assets/screen-shot-2019-04-14-at-10.59.22-am.png)

You can see [Firestore-1.2.1](https://github.com/firebase/firebase-ios-sdk/releases/tag/Firestore-1.2.1) was created 15 days ago, along with 3 more tags that all of them point to the same commit (7f869fb).

So how does the development process look like?

1. You create a new branch.
1. You make a bunch of changes in the repository.
1. If these changes are in the scope of say **pod X**, you bump its the version number  (in **pod X**'s podspec file).
1. Tag the commit that contains the podspec file with the new version number.
1. [Publish](https://guides.cocoapods.org/making/making-a-cocoapod.html) the new podspec file (`pod trunk push podX.podspec`)
1. Create a PR to merge your changes into the master branch.
