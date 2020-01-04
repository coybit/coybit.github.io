---
layout: post
title:  "bash: going down the rabbit hole"
tags: bash
---
### Question
Last week when I was looking at a bash script file, I saw this line:
``` sh
[[ $FOLDER_NAME == "ios" ]] && lib="lib-osx-amd64" || lib="lib-linux32"
```
I'm not a bash expert, so this line caught my attention. At first, I thought what `A && B || C` does is similar to what `A ? B : C` in many programming languages do. It takes a few hours of debugging, reading and thinking to realize this hypothesis was wrong!
Actually, if you're using `A && B || C` in bash how you use `A ? B : C`  in other programming languages without knowing what is happening under the hood, and it works for you, you are just lucky!

### Contradiction
After seeing that line, I started searching on the web to learn what that line means. The first thing I noticed after reading some articles and answers was that many people don't know that line gets evaluated by bash and they have given up to understand it. The vaguest aspect of evaluation was the precedence of operators that looks is different than when they are being used within a condition expression. Actually, you can not find any piece of information about it. The more you search, the less you find. The only thing you can find is the table of Operator Precedence  ( for instance [this one](https://www.tldp.org/LDP/abs/html/opprecedence.html) ) that doesn't say anything about when these operators are used in this kind of usages.
But at least you know that if `&&` and `||` getting used within a condition, how they get evaluated. `&&` has a higher precedence than `||`. So:
``` sh
[[ A || B && C ]]
```
First `B && C` is evaluated ( lets call this expression `q` ). Then `A || q` is evaluated.

``` sh
[[ A && B || C && D ]]
```
1. First this expression `q := (A && B)` (  note: `q := X` means from now on, we refer to `X` as `q` )
2. Then this one `p := (C && D)`
3. And finally, `q || p`

But what about when these two operators are used outside of a conditional expression? Are `&&` and `||` we are using in a conditional expression and the ones I saw in the bash file, in essence, the same thing. Are we tricking Bash and uses AND and OR logical operators to simulate `?:`?

When you can not find your answer by searching on the web, it means you need to get your hand dirty! Let's take a look into the bash source code.


### How to set up our lab
Obviously, there are thousands of thousands of way to set up your lab, but here I'm showing you how I did it for this example:

1. Download [Visual Studio Code](https://code.visualstudio.com/)

2. Download [C/C++ Extension](https://code.visualstudio.com/docs/languages/cpp#_debugging) for Visual Studio Code

3. Clone [bash source code](https://github.com/bminor/bash) ( it's a mirror of the source code )

4. Build it.
It's really simple.
a. go to the source code dir
b. run `./configure`
c. run `make`
and once it is done, you find a `bash` executable file in the directory
If you're interested in learning more about the process, look at [this file](https://github.com/bminor/bash/blob/master/INSTALL).

5. Run it
Be Careful: if you run `bash` in your bash, your installed bash is executed. But if instead run `./bash`, the executable bash file in the current directory is executed. So run `./bash`. 

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-start.png)

Welcome to your brand new bash!


6. Attach the debugger
To attach a debugger to the running bash, you need to config `launch.json` file of VSCode. It's super easy. Just follow [this instruction](https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md): 

This is my `launch.json` file:
``` json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        { 
            "name": "(lldb) Attach",
            "type": "cppdbg",
            "request": "attach",
            "program": "${workspaceFolder}/bash",
            "processId": "${command:pickProcess}",
            "MIMode": "lldb"
        }
    ]
}
```
After that, the only thing you need to do is running the debuger.

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-attaching-debugger.png)

Once the debugger is attached to the running bash, you can add a breakpoint, watch variables, executing the code line by line,...


### Connectional AND vs Conditional AND
I made the running bash to run these two commands and by adding a breakpoint and checking variables while the commands were getting executed, I tried to understand how bash evaluates them.
``` sh
[[ A == B && ls || cd ]] 
```
``` sh
[[ A == B ]] && ls || cd
```

To save your time, I want to show you where you should add your breakpoint. `shell.c` is the bash entry point ( it's where you can find the bash main() function ). `main` function calls `reader_loop` function that is in `eval.c`. This function waits till you enter a command in bash after parsing it and building a `command` object, sends it to `execute_command` in `execute_command.c` file and from there the command is sent to `execute_command_internal` in the same file. In the scope of this function, there is a variable called `command` which is of `command` type ( the definition in `command.h` ) and it contains the parsed command entered into bash by a user. the `command` type is really interesting. It is a `struct` with 4 general-purpose fields and one union, which represents details of command according to the command type.

``` c
/* What a command looks like. */
typedef struct command {
    enum command_type type;    /* FOR CASE WHILE IF CONNECTION or SIMPLE. */
    int flags;            /* Flags controlling execution environment. */
    int line;            /* line number the command starts on */
    REDIRECT *redirects;        /* Special redirects for FOR CASE, etc. */
    union {
        struct for_com *For;
        struct case_com *Case;
        struct while_com *While;
        struct if_com *If;
        struct connection *Connection;
        struct simple_com *Simple;
        struct function_def *Function_def;
        struct group_com *Group;
        #if defined (SELECT_COMMAND)
        struct select_com *Select;
        #endif
        #if defined (DPAREN_ARITHMETIC)
        struct arith_com *Arith;
        #endif
        #if defined (COND_COMMAND)
        struct cond_com *Cond;
        #endif
        #if defined (ARITH_FOR_COMMAND)
        struct arith_for_com *ArithFor;
        #endif
        struct subshell_com *Subshell;
        struct coproc_com *Coproc;
    } value;
} COMMAND;
```

For example, when you run this command: 
``` sh
[[ A == B ]] && echo 1 || echo 2 
```
the parser builds a `command` object like this:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-command.png)


Back to the question with which we started this section when the entered command is:
``` sh
[[ A == B && ls || cd ]] 
```
The bash parser generates this `command` and passes it down to `execute_command`.:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-cm_cond.png)

`cm_cond` means this command is a of `cond_com` type. By looking at the definitaion of this structure:

``` c
typedef struct cond_com {
    int flags;
    int line;
    int type;
    WORD_DESC *op;
    struct cond_com *left, *right;
} COND_COM;
```

we learn that:

- `left` and `right` are of `cond_com` type.
- `type` can be any of this values:

``` c
#define COND_AND    1
#define COND_OR        2
#define COND_UNARY    3
#define COND_BINARY    4
#define COND_TERM    5
#define COND_EXPR    6
```

So it is a tree. I've drawn this tree to make it easier what the parser has generated:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-tree.png)

But when you enter:
``` sh
[[ A == B ]] && ls || cd
```
the parset generates this `command` object:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-cm_connection.png)

The first difference you have probably noticed is the value of the `type` field. This value shows that the union part of the `command` structure represents a `connection` structure which is:

``` c
/* Structure used to represent the CONNECTION type. */
typedef struct connection {
    int ignore;            /* Unused; simplifies make_command (). */
    COMMAND *first;        /* Pointer to the first command. */
    COMMAND *second;        /* Pointer to the second command. */
    int connector;        /* What separates this command from others. */
} CONNECTION;
```

A `connection` command has two sub-commands, `first` and `second`.

I have drawn what the parser generates:

![](https://github.com/coybit/coybit.github.io/raw/master/assets/bash/bash-list.png)

By continuing debugging, we will see that how a `connection` command gets evaluated is different from how a `cond_com` command does. Actually, `cond_com` has a tree structure ( left node, right node and op ) and is made of the expression according to the  Operator Precedence table, but `connection` commands are more like a list ( it has first and second sub-command ) and gets evaluated from left to right given these two principles:

1. command1 && command2
Command2 is executed if, and only if, command1 returns an exit status of zero.

2. command1 |│ command2
Command2 is executed if and only if command1 returns a non-zero exit status.


In our example:
``` sh
[[ A == B ]] && ls || cd
```

1. `[[ A == B ]]` is evaluated. It is `false`, so according to principle #1 `ls` will be ignored.
2. Because the result of `[[ A == B ]] && ls` was false, according to princple #2, `cd` will be executed

But what about the other example:
``` sh
[[ A == B && ls || cd ]]
```
1. According to the generated tree, the expression is made of two sub-expression with the same precedence, so they gets evaluated from left to right:
    a. exp1: `A == B && ls`
    b. exp2: `cd`
2. `exp1` is made of two sub-expression with the same precedence, so they gets evaluated from left to right:
    a. exp1.1:  `A == B`
    b. exp1.2:  `ls`
3. `exp1.1` is `false`. So the `exp1` is `false`.
4. `exp2` gets evaluated.

As you see, the result of both of these two examples are the same.


### Summary
In spite of my first guest, `&&`/`||` in a conditional expression is not the same as `&&`/`||` in a list command. They get parsed and evaluated differently, but most of the time, the results are the same.
