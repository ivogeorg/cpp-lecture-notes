### Exceptions (5)

Any type can be thrown. To catch, the type of the `catch` statement argument type has to match. Catch blocks are called _handlers_.

```c++
#include <iostream>
#include "MyException.h"

using std::cout;
using std::cerr;
using std::endl;

int foo () throw (MyException, int, char *) {
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

class DimensionalityMismatchEx {
    unsigned int __current, __rhs;
public:
    DimensionalityMismatchEx(unsigned int c, unsigned int r) :
            __current(c),
            __rhs(r)
    {}
    unsigned int getCurrent() const { return __current; }
    unsigned int getRhs() const { return __rhs; }
    friend std::ostream &operator<<(std::ostream &os, const DimensionalityMismatchEx &ex) {
        os << "DimensionalityMismatchEx ("
        << "current = " << ex.__current << ", "
        << "rhs = " << ex.__rhs << ')';
        return os;
    }
};

enum class Throwable {
    INT, CSTR, STR, EXPTN
};

const Throwable somethingThrown = Throwable::EXPTN;

int main() {

    try {

        switch (somethingThrown) {

            case Throwable::INT:    throw 5;

            case Throwable::CSTR:   throw "Handle this!";

            case Throwable::STR:    throw std::to_string(3.14);

            case Throwable::EXPTN:  throw DimensionalityMismatchEx(4, 5);

            default:                throw -1;      // Note: no break necessary

        }

    } catch (const int i) {

        cerr << "Handling a caught integer (value = " << i << ')'
        << endl;

    } catch (const char * msg) {

        cerr << "Handling a caught C-string (value = " << msg << ')'
        << endl;

    } catch (const std::string msg) {

        cerr << "Handling a caught C++ string (value = " << msg << ')'
        << endl;

    } catch(const DimensionalityMismatchEx &e) { // catch/handle a thrown MyException

        cerr << "Handling a caught exception (value = " << e << ')'
        << endl;

    } catch (...) {

        cerr << "Handling a default or uncaught exception" << endl;

    }

    return 0;
}
```

Usually, it is best to define our own exception types (classes) so that we can capture any necessary data from the execution environment of the throwing block to the handler.

### Exceptions (6)

Cascaded `catch` statements are checked sequentially from top to bottom. The first match is executed and no further checking is done. Note that the usual automatic type conversion is NOT done in `catch` statements.

Obviously, `catch` statements have to be ordered carefully. For example, the catch-all ellipsis (**...**), if used, should be placed last. If there are inheritance relationships among the exception classes, parents should be placed after children. (Note that inheritance is a one-way _is-a_ relationship, so `class Child` is a `class Parent`, but not vice versa.)

```c++
// Exceptions.h
//
// Created by Ivo Georgiev on 10/27/15.
//

#ifndef EXCTEST_EXCEPTIONS_H
#define EXCTEST_EXCEPTIONS_H


class ParentException { };

class ChildException : public ParentException { };


#endif //EXCTEST_EXCEPTIONS_H

// main.cpp
int main() {
    
    try {
        
        // ...
    
    } catch (const ChildException &ex) {
        
        // ...
        
    } catch (const ParrentException &ex) {
        
        // ...
        
    } catch (...) {
        
        // ...
        
    }
    
    return 0;
}
```

Uncaught exceptions cause termination of the program. The C++ runtime calls the function  `std::terminate()` in this case, signaling the operating system to stop the process. (As a result, the OS sends a `terminate` signal to the process.)

The same function is thrown in several other cases, most notably:

  1. When an exception is thrown during exception handling.
  2. When a `noexcept` specification is violated (see below).

`std::terminate()` can also be called by the program.

### Exceptions (7)

`try...catch` blocks can be nested. An inner handler can forwarding and exception to the outer (which can potentially be repeated) by re-throwing it as follows:

```c++
int main() {
    
    try {
        
        try {
        
            // ...
            
        } catch (const MyException &ex) {
            
            // optionally, do some handling
            throw;
        
        }
        
    } catch (const MyException &ex) {
        
        // do more handling
        
    }
}
```

### Exceptions (8)

An exception can be thrown from anywhere and from arbitrarily deep on the call stack. A thrown exception unwinds the stack until the first matching `catch` statement, if any. Note the code that threw has to be wrapped, sometimes many levels up the stack, in a `try...catch` block.

```c++
#include <string>
#include <iostream>

using namespace std;

class MyException { };

class Dummy {
public:
    Dummy(string s) : MyName(s) { PrintMsg("Created Dummy:"); }
    Dummy(const Dummy& other) : MyName(other.MyName) { PrintMsg("Copy created Dummy:"); }
    ~Dummy() { PrintMsg("Destroyed Dummy:"); }
    void PrintMsg(string s) { cout << s  << MyName <<  endl; }

    string MyName; 
    int level;
};


void C(Dummy d, int i) { 
    cout << "Entering FunctionC" << endl;
    d.MyName = " C";
    throw MyException();   

    cout << "Exiting FunctionC" << endl;
}

void B(Dummy d, int i) {
    cout << "Entering FunctionB" << endl;
    d.MyName = "B";
    C(d, i + 1);   
    cout << "Exiting FunctionB" << endl; 
}

void A(Dummy d, int i) { 
    cout << "Entering FunctionA" << endl;
    d.MyName = " A" ;
  //  Dummy* pd = new Dummy("new Dummy"); //Not exception safe!!!
    B(d, i + 1);
 //   delete pd; 
    cout << "Exiting FunctionA" << endl;   
}


int main() {
    cout << "Entering main" << endl;
    try {
        
        Dummy d(" M");
        A(d,1);
    
    } catch (MyException& e) {
        
        cout << "Caught an exception of type: " << typeid(e).name() << endl;
    
    }

    cout << "Exiting main." << endl;
    char c;
    cin >> c;
    
    return 0;
}

/* Output:
    Entering main
    Created Dummy: M
    Copy created Dummy: M
    Entering FunctionA
    Copy created Dummy: A
    Entering FunctionB
    Copy created Dummy: B
    Entering FunctionC
    Destroyed Dummy: C
    Destroyed Dummy: B
    Destroyed Dummy: A
    Destroyed Dummy: M
    Caught an exception of type: class MyException
    Exiting main.

*/
```

Notice that any code after the thrown exception as well as any code from the nested functions on the stack that is after the call which threw the exception, will remain unexecuted. This can have profound effect on object state validity and should be a major design consideration. In general, design should avoid "far" (or "deep") throws. The closer the `try...catch` to the throwing code, the better. The "closer" the handler to the throwing code, the less object state and environment has to be potentially repaired. Also in general, exceptions should not be overused.

### Exceptions (9)

Some exceptions should be "internal" and should be caught in the codebase. Others, should be thrown out of the codebase and the user should be explicitly informed in the documentation to inspect for them. Which one is of which kind is a matter of design, but the following are good basic guidelines:

  1. The codebase can handle the exception gracefully and return to normal functionality.
  2. The codebase can give more informative feedback to the user by continuing past the exception.

For example, a few rows from thousands in a data file contain fewer than the expected dimensions. If an exception is thrown each time the wrong dimension is read, then it is much smarter to discard those rows and count them so as to report the total number of failed rows to the user. Terminating on the first dimensionality mismatch will be of far lesser utility. 

### Exceptions (10)

As mentioned in an earlier lecture, the so called dynamic exception specifications - basically lists of exception types a function can throw that follow the function signature - have been deprecated. Don't use them. The only exception specification for a function is now noexcept, which naturally signifies that the function will not throw any exceptions. Remember that if this condition is violated, the C++ runtime calls `std::terminate()` to terminate the program.

```c++
void f() noexcept;

void f(); // error, incompatible exception specifications

void g() noexcept(false);

void g(); // ok
```

As we have mentioned already, destructors are `noexcept` by default (implicitly).

`noexcept` doubles up as an operator that can check if a function is specified noexcept and returns a `bool`.

```c++
void may_throw();
void no_throw() noexcept;

class T {
public:
    ~T() { } // dtor prevents move ctor
             // copy ctor is noexcept
};

int main()
{
    T t;
 
    std::cout << std::boolalpha
    << "Is may_throw() noexcept? " << noexcept(may_throw()) << '\n'
    << "Is no_throw() noexcept? " << noexcept(no_throw()) << '\n'
    << "Is ~T() noexcept? " << noexcept(std::declval<T>().~T()) << '\n'
    
    return 0;
}

/* Output:
Is may_throw() noexcept? false
Is no_throw() noexcept? true
Is ~T() noexcept? true
*/
```

### Exceptions (11)

The following is a selection from the [C++ Reference on exception usage and exception safety guarantees](http://en.cppreference.com/w/cpp/language/exceptions). I can't summarize it better than it is done there. (The links are from the reference and will likely lead you there.)

  **Error handling**

  Throwing an exception is used to signal errors from functions, where "errors" are typically limited to only the following[1] (Links to an external site.)[2] (Links to an external site.):

    1. Failures to meet the postconditions, such as failing to produce a valid return value object
    2. Failures to meet the preconditions of another function that must be called
    3. (for non-private member functions) Failures to (re)establish a class invariant

  In particular, this implies that the failures of constructors (see also RAII (Links to an external site.)) and most operators should be reported by throwing exceptions.

  In addition, so-called wide contract functions use exceptions to indicate unacceptable inputs, for example,std::string::at (Links to an external site.) has no preconditions, but throws an exception to indicate index out of range.

  **Exception safety**

  After the error condition is reported by a function, additional guarantees may be provided with regards to the state of the program. The following four levels of exception guarantee are generally recognized, which are _strict supersets_ of each other:

    1. Nothrow (or nofail) exception guarantee -- the function never throws exceptions. Nothrow (errors are reported by other means or concealed) is expected of destructors (Links to an external site.) and other functions that may be called during stack unwinding. The destructors (Links to an external site.) are noexcept (Links to an external site.) by default. (since C++11) Nofail (the function always succeeds) is expected of swaps, move constructors (Links to an external site.), and other functions used by those that provide strong exception guarantee.
    2. Strong exception guarantee -- If the function throws an exception, the state of the program is rolled back to the state just before the function call. (for example, std::vector::push_back (Links to an external site.))
    3. Basic exception guarantee -- If the function throws an exception, the program is in a valid state. It may require cleanup, but all invariants are intact.
    4. No exception guarantee -- If the function throws an exception, the program may not be in a valid state: resource leaks, memory corruption, or other invariant-destroying errors may have occurred.

  Generic components may, in addition, offer exception-neutral guarantee: if an exception is thrown from a template parameter (e.g. from the Compare function object of `std::sort` or from the constructor of `T` in `std::make_shared`), it is propagated, unchanged, to the caller.

### Exceptions (12)

The Standard Library provides a base class `std::exception` and a number of standard subclass exceptions as a consistent interface to handle errors through the `throw` expression. You can also inherit from these exceptions, if you like.
