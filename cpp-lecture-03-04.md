### Exceptions (1)

Exceptions are a mechanism for interrupting the normal flow of code execution upon encountering an irrecoverable error and sending a signal to a (possibly remote) section of code which can handle the situation. The exception is generated in the code itself, usually after a conditional statement testing for correct state comes up negative. The code that receives the exception is written ahead of time and designed to achieve the best recovery from the invalid state as possible under the current conditions. If that is impossible, the exception can be propagated farther up the call stack hierarchy, potentially reaching the top level and exiting to the operating system, which terminates the program.

```c++
// PointEx.h
class DimensionalityMismatchEx {
    unsigned int __current, __rhs;
public:
    DimensionalityMismatchEx(unsigned int c, unsigned int r) :
        __current(c),
        __rhs(r)
    {}
    unsigned int getCurrent() const { return __current; }
    unsigned int getRhs() const { return __rhs; }
};

// Point.cpp
#include "PointEx.h"

// Point comparison
bool operator==(const Point &one, const Point &another) throw (DimensionalityMismatchEx) {
    if (one.dim != another.dim)
        throw DimensionalityMismatchEx(dim, rhs.dim); // comparison of points with different dimensionality is undefined

    bool isEqual = true;
    for (int i = 0; i < one.dim; i++) {
        if (one.values[i] != another.values[i]) {
            isEqual = false;
            break;
        }
    }

    return isEqual;
}

// main.cpp
#include <iostream>
#include "PointEx.h"

int main() {
    Point p5(5), p10(10); // points of different dimensionality
    
    try {
        if (p5 == p10)
            std::cout << "Points are equal." << std::endl;
    } catch (DimensionalityMismatchEx &e) {
        std::err << "EXCEPTION: Dimensionality Mismatch: Comparison of points with different dimensionality (" << e.getCurrent() << ", " << e.getRhs() << ") is undefined!" << std::endl;
    }
    
    return 0;
}
```

The example shows how the code detects an anomalous comparison of points with mismatched number of dimensions (which doesn't make sense and is therefore neither true nor false) and uses a predefined exception class (DimensionalityMismatchEx) to abort the execution (the throw statement in operator==) and convey that information to a section of code where the situation can be handled (the catch block in the main function).

Note that any code between the throw and catch will remain unexecuted if the exception is actually thrown. If the exception is caught, the matching handler code block is executed, after which the execution resumes after the last catch block (NOT after the throw statement which threw the exception).

### Exceptions (2)

The exception syntax adds three new keywords: `try`, `catch`, and `throw`. The basic structure of code with exception handling is as follows:

```c++
#include <iostream>
#include <string>
#include "MyException.h"

using namespace std;

int foo() throw (MyException, int, char *) { // DEPRECATED as of C++11
    // ...
    // some state check fails
    if (true) throw MyException();
    // nothing gets executed after the thrown exception
    // ...
    if (!false) throw 10;
    // ...
    if (!false) throw "Ouch!";
    return 1;
}

int main() {

    try {
        // ...
        foo();
        // no code past the exception throw is executed
        // ...
    } catch (const int i) { // catch/handle a thrown int
        cout << i << endl;
    } catch (const string msg) { // catch/handle a thrown string
        cout << msg << endl;
    } catch(const MyException &e) { // catch/handle a thrown MyException
        cerr << "caught mi exception" << endl;
    } catch (...) { // catch/handle anything else thrown (default exception)
        cerr << "here" << endl;
    }

    return 0;
}
```

The `try...catch` block is the facility for catching any thrown exception. A number of throw statements can be embedded in that block, often nested in function calls, at any depth of nesting. The example shows only one level of nesting, where foo() can throw a number of exceptions. When a thrown exception is caught in the block, its type is checked sequentially against the cascaded catch blocks. The first block whose argument matches is executed and no others. The code continues execution after the last catch block. The last catch block has an ellipsis, which makes it a "catch-all": anything uncaught by preceding catch clauses will be "caught" here. Notice that this is a "default exception" block where you can't use the contents of the thrown object. 

**Note:** The dynamic exception specification (the statement following the argument list in the declaration of foo(), which starts with throw and specifies the types of exceptions the function may throw) is a **deprecated** feature in C+11. This means that, while still supported for backward compatibility with legacy code, the practice is strongly discouraged for new code.

### Exceptions (3)

Notice that in our first example, we used the `DimensionalityMismatchEx` to further constrain the usage of our `Point` class within reasonable and sensible bounds. Exceptions are thus a part of a class' interface. They not only allow the code writer to recover from anomalous situations without terminating the program, but they give hints to the user about how to best use the code base.
