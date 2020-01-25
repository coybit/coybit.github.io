---
layout: post
comments: true
title:  "the curious case of nil-coalescing operator"
tags: Swift
---
The Curious Case of Nil-Coalescing Operator

Using `??` within a class initializer under some circumstances leads to an ambiguous compiler error message. I encountered this problem when I was writing a class like this:
``` swift
class MyClass {
    private var fname: String?
    private var lname: String
    private var displayName: String
    
    init(firstName: String?, lastName: String) {
        self.fname = firstName
        self.lname = lastName
        displayName = self.fname ?? self.lname // Compiler-Error: 'self' captured by a closure before all members were initialized
    }
}
```

The compiler error message looks odd. There isn't any closure there!
Rewriting that problematic line in another way without using `??` operator, for example:
``` swift
displayName = self.fname != nil ? self.fname! : self.lname
```
resolves the issue. Also, replacing `self.lname` with anything that is not a property of `self`, resolves the issue too. So, apparently, the issue has something do to with `self` and `??` operator.

After looking at the implementation of nil-coalescing operator in [Optional.swift](https://github.com/apple/swift/blob/master/stdlib/public/core/Optional.swift#L656), you will know why this is so:

``` swift
public func ?? <T>(optional: T?, defaultValue: @autoclosure () throws -> T?)
    rethrows -> T? {
  switch optional {
  case .some(let value):
    return value
  case .none:
    return try defaultValue()
  }
}
```

The second parameter of this operator is a `@autoclosure`. It means it is a closure even though we don't want :)
So this is how the compiler sees that line:
``` swift
displayName = self.fname ?? (){ self.lname }
```
Because the value of `displayName` is not known unless the expression at the right-hand side of `=` is evaluated and the expressed doesn't get evaluated unless the right-hand side of `??` (the auto-closure) is executed. But this closure captures `self`. So we are capturing `self` while `self` is not fully initialized. Now we can understand what the error message means:

```
'self' captured by a closure before all members were initialized
```
