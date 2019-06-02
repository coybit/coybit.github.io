---
layout: post
title:  "__NSSingleObjectArrayI: an existential crisis"
---

When you create an new instance of [NSArray](https://developer.apple.com/documentation/foundation/nsarray/1460096-arraywithobjects?language=objc), inside [arrayWithObjects:count:](https://developer.apple.com/documentation/foundation/nsarray/1460096-arraywithobjects?language=objc)  (which can be called directly and is called indirectly by other constructors and also is that method is used internally to create an array from a literal array) depending on how many items it has, under the hood `NSArray` may create either an instance of `__NSArrayI`([Runtime Header file](https://github.com/nst/iOS-Runtime-Headers/blob/master/Frameworks/CoreFoundation.framework/__NSSingleObjectArrayI.h)) or an instance of  `__NSSingleObjectArrayI` ([Runtime Header file](https://github.com/nst/iOS-Runtime-Headers/blob/master/Frameworks/CoreFoundation.framework/__NSSingleObjectArrayI.h)). Actually, if the array contains one item, `NSArray` create an instance of `__NSSingleObjectArrayI`, otherwise `__NSArrayI` gets used.

```objective-c
NSArray *arr1 = @[@(1)]; // arr1 is __NSSingleObjectArrayI
NSArray *arr2 = @[@(1), @(2)]; // arr2 is __NSArrayI
```

By looking at `__NSSingleObjectArrayI.h` you will notice that this class has a protected variable, called `_object`, that I guess it is where the single item of the array is stored.
```objective-c
@interface __NSSingleObjectArrayI : NSArray {
    id  _object;
}
```

As soon as you learn that single object arrays are treated differently, an important question arises: **What is the necessity of having a separate type for array that has just one object?**
My first guess was Performance, either in terms of speed or memory usage, or maybe both. I tested my hypothesizes and I like to share the results with you. If you like to learn more about benchmarking in Objective-C/Swift, read "[Benchmarking](https://nshipster.com/benchmarking/)" by Mattt. To test my hypothesizes I utilizeed what I had learned from this article.

### Hypothesis1: It is faster

I measured how much it takes to create an instance of `NSArray`. The varibale of this experiment was the number of items the array contains. Also, I was curious if specifing the type of array's items make any difference. So I repeated experiments once again but this type specified the type of arrays' items: 

Case#   | Array   |  Time
--------|----------|----
Case1    | `NSArray *arr = @[@(1)];`                                                 | 2.62712e-05 s
Case2    | `NSArray<NSNumber *> *arr = @[@(1)];`                                     | 2.59094e-06 s
Case3    | `NSArray *arr = @[@(1), @(2)];`                                           | 1.91713e-06 s
Case4    | `NSArray<NSNumber *> *arr = @[@(1), @(2)];`                               | 2.41189e-06 s
Case5    | `NSArray *arr = @[@(1), @(2),@(3),@(4),@(5),@(6),@(7),@(8)];`             | 2.89804e-06 s
Case6    | `NSArray<NSNumber *> *arr = @[@(1), @(2),@(3),@(4),@(5),@(6),@(7),@(8)];` | 3.00701e-06 s

In general, create an instance of  `__NSSingleObjectArrayI` is slower than `__NSArrayI`, and specifying the type of array's items makes the creation faster. 
Then I measured the amount of time it takes to access the first item of an array. The varible was the type of array, but the access method was the same:

Case#   | Array   |  Time
--------|----------|----
Case1    | `NSArray *arr = @[@(1)];`                                                 | 4.65124 s
Case2    | `NSArray<NSNumber *> *arr = @[@(1)];`                                     | 4.61453 s
Case3    | `NSArray *arr = @[@(1), @(2)];`                                           | 4.11629 s
Case4    | `NSArray<NSNumber *> *arr = @[@(1), @(2)];`                               | 4.04419 s
Case5    | `NSArray *arr = @[@(1), @(2),@(3),@(4),@(5),@(6),@(7),@(8)];`             | 4.05391 s
Case6    | `NSArray<NSNumber *> *arr = @[@(1), @(2),@(3),@(4),@(5),@(6),@(7),@(8)];` | 4.09293 s

As you can see, accessing the first item of an array of `__NSSingleObjectArrayI` type is slower than `__NSArrayI`, and specifying the type of array's items enables us to access the first item faster.


### Hypothesis2: It reduces memory usage 

I ran another experiment. This time in each test case, I created 1000 instance of array with `N` items and used Instruments to measure how much memory was used to store these instances. This time, the variable of the experiment was `N`: Here is the result:

Case#   | Array   |  Memory
--------|----------|----
Case1    | `@[@(1)]`                          | 156,25 Kb
Case2    | `@[@(1),@(2)]`                     | 312,50 Kb
Case3    | `@[@(1),@(2),@(3)]`                | 468,75 Kb
Case4    | `@[@(1),@(2),@(3),@(4)]`           | 468,75 Kb
Case5    | `@[@(1),@(2),@(3),@(4),@(5)]`      | 625,00 Kb
Case6    | `@[@(1),@(2),@(3),@(4),@(5),@(6)]` | 781,25 Kb

Even when we compare memory usage, there is no significant difference between these two (size of `@[@(1),@(2)]` x 2 = size of  `@[@(1)]`, that makes sense).

### Conclusion
I don't still understand the reason behind having a separate type for immutable arrays that have just one item, though I know now that neither is faster nor used less memory. On the other hand, I couldn't find a use case where someone needs to create thousands instance of single object array so might this subtle difference become important for the developer. Maybe this is why no one has ever complained about that.
The only case where I can image this difference might be important is where a JSON gets decoded. Because if a JSON array has just one item, the decoded type will be `__NSSingleObjectArrayI`. So if you are working on a program that its performance is critical and somewhere in this program you have to decode a huge JSON which has many single item arrays, you should be more careful.
