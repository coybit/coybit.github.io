---
layout: post
title:  "api design: in search of confidence"
categories:
---
A given API to be well-designed has to fulfill many requirements and we can talk about these requirements for hours. But here I'm going to talk about one aspect of a well-thought and well-design API that I believe is really important: How confident we are in what we are doing?
When we are using an API, as we are writing code, for a second we ask ourselves "Am I doing it right?", "Is it the right method", "Don't I have to call another method before/after this one?", etc. The API should be designed in such a way that minimizes the frequency of these uncertainty moments. Here I like to talk about three techniques that I utilized just recently:

### 1. Defamiliarization: Display or Present
One time I had a discussion with one of my teammate about the name of a method. What this method ought to do was simple. It presented a view controller but before that, it dismissed the presented view controller if there was any.

``` objective_c
- (void)dispay:(UIViewController *)viewControllerToPresent
      animated:(BOOL)flag
    completion:(void (^)(void))completion;
```

I said `display` could be a good option. But my college insisted that we should stick to iOS terminology and pick a name which familiar for every iOS developer, something like `present` for example. But I believed that if we did so it would be misleading for one who would use this method. Because probably intuitively they would think that this method would just present a view controller and not being aware of the other responsibility of this method ( dismissing the already presented view controller ) would cause problems like this: [dismissViewControllerAnimated completion block is not called](https://stackoverflow.com/a/17320118/946835). Eventually, we found something between and named the method:
``` objective_c
    - (void)presentViewController:(UIViewController *)viewControllerToPresent
                         animated:(BOOL)flag
andDismissPresentedViewController:(BOOL)dismissPresentedViewController
                       completion:(void (^)(void))completion;
```
I was happy with the result. The new signature of the method is not familiar enough that users deceive that they know it and is not too unfamiliar that makes them check the documentation.

### 2. Consistency: User or UserID or UserIdentifier
It is not uncommon that in a codebase an entity is being referred to with different names. For example, you have an entity that represents a unique identifier associated with each user. Somewhere you may see:
``` objective_c
- (void)sendMessageTo:(UInt64)userID;
```
And in another file you face this line:
``` objective_c
- (User*)findUser:(UInt64)ID;
```
And a few lines below that it reads:
``` objective_c
- (void)invoicesOfUserWithIdentifier:(UInt64)userIdentifier;
```

When we assign different names to the same entity across the code base, we weaken the confidence of users of our API every time they encounter this entity and make them ask themselves this kind of question every time:
> Are X and Y different name for the same thing?
Asking this question and find an answer to it might take less than one second. But it doesn't take the sting out of it. Everytime we choose a different name for the same thing, actually, we're implicitly attaching an explanation placeholder to the newly chosen name and all other chosen name which must be filled out by users of our API:
``` objective_c
- (void)invoicesOfUserWithIdentifier:(UInt64)userIdentifier; // userIdentifier is the same thing that userID is and the same thing that ...
```

Not using a generic type like `UInt64` as a type can be helpful too. In this case, having a slightly different name for the same thing could be more toleratable:

``` objective_c
typedef UInt64 UserID;
...

- (void)sendMessageTo:(UserID)userID;

...

- (User*)findUser:(UserID)ID;

...

- (void)invoicesOfUserWithIdentifier:(UserID)userIdentifier;
```


### 3. Self-Explanatory: Can I pass `nil`
We can have methods signature expose more information about the criteria of their parameters. Things like whether  `nil` can be passed, zero is an acceptable value, or the passed array can be empty, etc. It doesn't mean that we toss the necessity of invalidation within the body of methods. It's just minimal documentation and let the user of our API use it more confidently as they are writing code.
It's a subtle technique that reduces the necessity of reading documentation when you are using an API. Let's look at an example from UIKit. Here is one of the constructors of `UIViewController`:
``` objective_c
- (instancetype)initWithNibName:(NSString *)nibNameOrNil
                         bundle:(NSBundle *)nibBundleOrNil;
```
Once you start writing the name of this function, by looking at the opened autocompletion popup, you learn that for the first parameter you can send either a `nibName` or you can pass `nil`. If you are worried that `nibNameOrNil` and `nibBundleOrNil` are not very suitable name to be used in the implementation of the method, at least in Objective-C we have this chance to assign different names to them within the implementation of the method ([link](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/DefiningClasses/DefiningClasses.html)):

> Note: The value1 and value2 value names used above aren’t strictly part of the method declaration, which means it’s not necessary to use exactly the same value names in the declaration as you do in the implementation. The only requirement is that the signature matches, which means you must keep the name of the method as well as the parameter and return types exactly the same.
> As an example, this method has the same signature as the one shown above:
>
> ``` objective_c
> - (void)someMethodWithFirstValue:(SomeType)info1 secondValue:(AnotherType)info2;
> ```
>
> These methods have different signatures to the one above:
>
> ``` objective_c
> - (void)someMethodWithFirstValue:(SomeType)info1 anotherValue:(AnotherType)info2;
> ```
>
> ``` objective_c
> - (void)someMethodWithFirstValue:(SomeType)info1 secondValue:(YetAnotherType)info2;
> ```

I told that this technique is subtle because you should be very careful regarding the way and the amount of information you are going to have a method reflect. For instance imagine how bad it would be if instead of method's value names, method's parameter names carry this information:
``` objective_c
- (instancetype)initWithNibNameOrNil:(NSString *)nibName
                         bundleOrNil:(NSBundle *)nibBundle;
```

Or if exposed information was a little more:
``` objective_c
- (instancetype)initWithNibName:(NSString *)nibNameWithoutLeading path informationOrNil
                         bundle:(NSBundle *)nibBundleInLanguageSpecificProjectDirectoriesOrResourcesDirectory. OrNil;
```

Probably, you have noticed that the UIKit example is an inclusive manner: "You can pass `nil` too". But what if we employ this technique in an exclusive manner: "You can not pass XYZ". Let's look at a few examples:

``` objective_c
- (Circle)circleWithRadius:(float)positiveRadius;
```

``` objective_c
- (void)addItems:(NSArray*)nonEmptyArrayOfItems;
```

``` objective_c
- (BOOL)writeToFile:(NSString *)absolutePath;
```
It's not as much appealing as the inclusive manner, is it?
