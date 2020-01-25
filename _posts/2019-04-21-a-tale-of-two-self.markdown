---
layout: post
comments: true
title:  "a tale of two `self`"
categories: 
tags: Swift Objective-C
---
A while ago, I came across a problematic piece of code that turned out to be an interesting one:

``` swift
extension UIViewController {
    static var screenName = String(describing: self)
}
```

The value of screenName is always equal to `(Function)`. But what I needed was the type of `UIViewController` instance as a String. For example:

``` swift
class ReaderViewController: UIViewController { ... }
print(ReaderViewController)

// Expected Output: ReaderViewController
// Actual Output: (Function)
```

But why is that so?

I tried to simplify the problematic piece of code and reduce the variables that were involved. In the first step, I took out `screenName` from the extension and put it inside a class:

``` swift
class ReaderViewController: UIViewController {
    static var screenName = String(describing: self)
}
```

The result was the same.

What if `ReaderViewController` doesn’t inherit from `UIViewController`?

``` swift
class ReaderViewController {
    static var screenName = String(describing: self)
}
```

Boom! Compiler Error:

```
Use of unresolved identifier 'self'.
```

By trial and error I managed to narrow down the problem to the point that led me to this conclusion:

- If the root class of `ReaderViewController` is `NSObject`, the value of `screenName` is `(Function)`.
- Otherwise, we get the compiler error

It looks like we have two different problems here that need to be investigated separately. But actually, when I was looking for the root cause of the former one, I found an explanation for the later one too.

When I jumped to the definition of `NSObject`, I faced this (summarized) code:

``` swift
public protocol NSObjectProtocol {

    //...

    func `self`() -> Self

    //...

}

@available(iOS 2.0, *)
open class NSObject : NSObjectProtocol {
    //...
}
```

`self` is a method that returns the object which it is called on.
It explains why `String(describing: self)` returns `(Function)` within a subclass of `NSObject`. But it doesn’t explain why within the initializer expression of a stored type property, `self` isn’t available.

I replaced the stored type property with a computed one, and it silenced the compiler-error problem:

``` swift
class ReaderViewController {
    static var screenName: String {
        return String(describing: self)
    }
}
```

Still, I don’t know why referring to self within type property initializer expression caused that compiler error and I have asked it here.

By the way, at least now I know that self within type property initializer expression is not available, so it is an unresolved identifier. If the class is a subclass of `NSObject`, in the absence of self, `NSObject`’s `self` ( which is a method ) is available and String(describing: self) returns the type of it which is `(Function)`.

---
P.S. While I was writing this post, I found [this](https://forums.swift.org/t/self-as-initializer-bug/17107) related question on Swift Forum:
