### Memory management (1)

Variables and constants.

Local variables. Global variables. Static variables.

Constants.

**//TODO**

### Memory management (2)

Variable allocation.

Static (aka automatic) vs dynamic.

Stack vs heap.

**//TODO**

### Memory management (3)

Pointers and references.

**//TODO**

### Memory management (4)

The call stack.

A representation of the call stack can be seen when a breakpoint is inserted in code and it is examined in the debugger:

![clion-debugger-call-stack](https://cloud.githubusercontent.com/assets/6043344/15994731/f7f6975a-30c0-11e6-89fa-24df40a5d4d6.png)

The call stack is composed of stack frames, one for each function that has been called and has not yet terminated. Naturally, the stack frames are allocated on the stack in first-in-last-out (FILO) manner. This means that if main called foo, and foo called bar, and bar has not yet finished executing, there will be 3 stack frames on the call stack for, from bottom to top, main, foo, and bar.

Stack frames contain: 

  * function arguments
  * local (automatic) variables
  * return address
