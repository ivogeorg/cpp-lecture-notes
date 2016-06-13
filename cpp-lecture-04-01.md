## Overview

Module 4 consists of 3 weeks and 6 lectures. It presents inheritance and polymorphism, one of the principal features of the C++ programming language (as well as all other OO languages). It explores issues of class hierarchy design, behavior abstraction and encapsulation, and design patterns.

### Inheritance (1)

Multi-level abstraction of behavior: classes can be organized into a hierarchy with clear ancestor-descendant (parent-child, base-derived, superclass-subclass) relationships. This organization is called inheritance. Child classes inherit behavior from their parent classes and usually and extend it. For example:

```c++
//
// Created by Ivo Georgiev on 10/22/15.
//

#ifndef INHERITTEST_CENTROID_H
#define INHERITTEST_CENTROID_H

#include "Point.h"

namespace Clustering {

    class Centroid : public Point {
        bool __valid;
        // __dim inheritted
        // __values inheritted

    public:
        Centroid(int dim) :
                Point(dim), // call the base class constructor first
                __valid(false)
        {}

        Centroid(const Centroid &c) :
                Point(c),
                __valid(false)
        {}

        Centroid &operator=(const Centroid &c) {
            if (this != &c) {
                Point::operator=(c);
                __valid = c.__valid;
            }
            return *this;
        }

        ~Centroid() {
            // ~Point() called automatically before exit
        }
    };

}
#endif //INHERITTEST_CENTROID_H
```

This example shows how we can have class `Centroid` extend class `Point`. `Centroid` inherits the behavior of `Point` but also adds a new field `__valid` and extends the behavior to handle this new data member. The inherited behavior can be directly invoked by calling the base class methods.

Notice that the `Centroid` constructors call the corresponding `Point` constructors **first** in the initialization list before any other statements. Inherited state is initialized before the extended state. It is the only strategy that makes sense from the perspective of the actual implementation of the inheritance mechanism in C++.

Notice that outside of constructors, the corresponding method of the base class (in this case `operator=`) can be called inside the code block of the method. Though not strictly necessary, it is good practice to qualify the base class method with the base class name as in `Point::operator=`.

Inheritance can be done hierarchically at multiple levels, say, class B extends class A, class C extends class B, etc. In this case, upon initialization, the state inherited from A should be initialized **first**, then the state extended in B, then C, etc.

Notice that the further "down" the hierarchy a class is:

  1. The more behavior it inherits.
  2. The more functionality it implements.
  3. The more specific it becomes!

At the opposite end of the subclass object lifecycle, in the destructor, the opposite thing happens automatically: `~C()` invokes `~B()` before exiting, which in turn invokes `~A()` before exiting.

### Inheritance (2)

Inheritance is a _is-a_ relationship, as opposed to a _has-a_ one. Compare the following two implementations of the class `Centroid`:

```c++
// Inheritance: is-a relationship
class Centroid : public Point {
    bool __valid;
    // __dim inheritted
    // __values inheritted

public:
    Centroid(int dim) :
            Point(dim), // call the base class constructor first
            __valid(false)
    {}
    // ...
};

// Containment: has-a relationship
class Centroid {
    bool __valid;
    unsigned int __dimensionality;
    Point __p;

public:
    Centroid(int dim) :
            __dimensionality(dim),
            __p(__dimensionality), // call the Point consructor
            __valid(false)
    {}
    // ...
};
```

Notice that an _is-a_ relationship is strictly _unidirectional_: a `Centroid` is a `Point`, but a `Point` is **not** a `Centroid`. This is not just a statement of abstract relationship, but of functional design:

```c++
#include <iostream>
#include "Point.h"
#include "Centroid.h"

using namespace std;
using namespace Clustering;

int main() {
    const int dim = 10;
    Centroid c(dim);

    cout << c << endl;

    Point p(c); // ok: c is a Point

    cout << p << endl;

//    Centroid c1(p); // error: p is not a Centroid

    return 0;
}
```

### Inheritance (3)

What is inherited and what isn't? The usual suspects. Naturally, constructors, copy constructors, assignment operators, and destructors are not inherited, since they don't know about the new members that the subclass has defined and therefore cannot perform this extra behavior. Remember that if any of these methods are not defined in the derived class, the compiler will automatically generate default ones. The default behavior might not be what the designer wants.

All other methods are inherited, but they still might not be exactly the behavior the derived class needs. If this is the case, the class can redefine base class methods.

