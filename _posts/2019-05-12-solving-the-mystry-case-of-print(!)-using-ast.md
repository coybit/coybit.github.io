---
layout: post
comments: true
title:  "solving the mystry case of `print(!)` using ast"
categories: 
tags: Swift iOS compiler
---
It all started when I mistakenly removed `date` from this line of code just before pressing Run button of XCode:
``` swift
print(date!)
```
The app got compiled successfully and this line:
``` swift
print(!)
```
produced this output:
```
(Function)
```

Out of curiosity, I started replacing `!` with other operators, like `*`, `%`, `^`, ... that all led to the same compiler error message:
```
error: ambiguous use of operator '*'
error: ambiguous use of operator '%'
error: ambiguous use of operator '^'
```

At this point, I had two questions:
1. Where does that `!` come from?
2. Why doesn't substituting '!' with other operator doesn't have the same result?

Unfortunately, Jump To Defenition (^ + command) feature doesn't work for operators. So how can I do now? I decided to dump **AST** output of the compiler. Because the generated AST is easily readable and it's the output of **Semantic Analysis** (run right after **Parsing** phase) and by end of this phase the compiler knows where the declaration of called methods can be found:

>**Parsing**: The parser is a simple, recursive-descent parser (implemented in lib/Parse) with an integrated, hand-coded lexer. The parser is responsible for generating an Abstract Syntax Tree (AST) without any semantic or type information, and emit warnings or errors for grammatical problems with the input source.

>**Semantic analysis**: Semantic analysis (implemented in lib/Sema) is responsible for taking the parsed AST and transforming it into a well-formed, fully-type-checked form of the AST, emitting warnings or errors for semantic problems in the source code. Semantic analysis includes type inference and, on success, indicates that it is safe to generate code from the resulting, type-checked AST.

If you want to know about AST, check out [this page](https://swift.org/compiler-stdlib/#compiler-architecture). 
So I ran this command:
``` sh
swiftc -dump-ast -O print.swift
```

`print.swift` was a simple and short swift file:
``` swift
import Foundation
print(!)
```

The output was:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/shot-ast1.png)

The interseing part is:
``` sh
(declref_expr type='(Bool.Type) -> (Bool) -> Bool' location=print.swift:2:7 range=[print.swift:2:7 - line:2:7] decl=Swift.(file).Bool.! function_ref=unapplied)
```

And here you can see that `!` has been declared in `Swift.(file).Bool.!`. Let's go to where `Bool` type has been declared to see what can we find [there](https://github.com/apple/swift/blob/2efbeb391281c432b40c7b4eba10fd3529112568/stdlib/public/core/Bool.swift):
``` swift
extension Bool {
  /// Performs a logical NOT operation on a Boolean value.
  ///
  /// The logical NOT operator (`!`) inverts a Boolean value. If the value is
  /// `true`, the result of the operation is `false`; if the value is `false`,
  /// the result is `true`.
  ///
  ///     var printedMessage = false
  ///
  ///     if !printedMessage {
  ///         print("You look nice today!")
  ///         printedMessage = true
  ///     }
  ///     // Prints "You look nice today!"
  ///
  /// - Parameter a: The Boolean value to negate.
  @_transparent
  public static prefix func ! (a: Bool) -> Bool {
    return Bool(Builtin.xor_Int1(a._value, true._value))
  }
}
```
This is how `!` operator works under the hood then.

It's time to answer the next question. Why do we get a compiler error when we try to compile `print(*)`? If we dump AST again for this source code:
``` swift
import Foundation
print(*)
```

We get this output:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/shot-ast2.png)

In the last part of the output, there is the list of all available declarations of `*` that Swift compiler cannot decide which one we mean to use. How can we solve this ambiguous case? I found the answer [here](https://ericasadun.com/2018/03/09/the-curious-case-of-operator-assignment/). The answer is neither trivial nor documented somewhere in Swift documentation:

> The answer is to both parenthesize and type the operator

I changed the source code accordingly and the problem solved:

``` swift
let op: (Int, Int) -> Int = (*)
print((*)) // Output: (Function)
```
