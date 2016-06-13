### Class design (4)

When we are working with a dynamic array of `PointPtr` as a private member of the `Cluster` class, how should we treat it in the copy constructor and assignment operator? In general, we need to make full copies of the dynamic data structure members for the newly initialized object. In the context of the Cluster class example, if we are initializing a new Cluster object with an existing one, as in `Cluster c2(c1);` or `Cluster c2 = c1;`, we would allocate a copy of the points structure of c1 for c2. That way, c1 and c2 would have the same, but independent, `Point` data right after this initialization. In general, this is the behavior we want from the copy constructor and the assignment operator when we are dealing with dynamic member variables, since we don't want two different objects to point to the same allocated objects and risk garbling each other's data.

However, this might not always be the behavior we want. If we want to do clustering, where the total number of `Point` objects is fixed during the run of our program, and we want to divide them into several clusters, this is not the behavior we want. Our `Cluster` objects are merely lightweight collections of Point objects, and they are just used to divide the Points into groups based on the spatial proximity among them. We don't want to make copies of `Point` to assign to each `Cluster`, but to divide the total fixed number of `Point` which are allocated during the program initialization, into a certain number of `Cluster`. So, we actually don't need to make copies of non-empty clusters and initialize one from another (which is the purpose of the copy constructor and the assignment operator). Instead, to start the clustering algorithm, we create a number of empty `Cluster` and then divide the Point-s among them largely arbitrarily. Through subsequent iterations of the algorithm, the `Point` get removed from one `Cluster` and added to another until the `Cluster` approximate the natural grouping of the `Point` in space. This is the purpose of the clustering algorithm.

In summary, you are encouraged to implement the copy constructor and assignment operator as in the general case, that is, making proper copies of dynamic data structures, and then just be careful not to use them when you don't want to make duplicates of points _(which would amount to creating new fake data)_.

### Operator overloading (1)

Operators are functions. Overloading is one more way to support our intuition about object behavior. For example, if there is a _clear, unambiguous, intuitive, undisputed_ way to "add points", we should consider overloading `operator+` in the `Point` class.

Three ways to do it:

  * non-member functions, _but no access to object internals, so incur overhead of using accessors_

```c++
// File: Point.h

class Point {
    int dimensions;
    double *values;
    
public:
    // constructors, operator=, destructor
    // other public methods
    
    // ACCESSORS!!!
    int getDims() { return dimensions; }
    const double *getVals() { return values; }
};

bool operator=(const Point &a, const Point &b);

// ...

// File: Point.cpp or other

#include "Point.h"

bool operator=(const Point &a, const Point &b) {
    bool equal = false;
    
    if (a.getDims() == b.getDims()) {
        bool equal = true;
        double *avals = a.getValues(), 
            *bvals = b.getValues();
        for (int i = 0; i < a.getDims() ; i ++) {
            if (avals[i] != bvals[i]) {
                equal = false;
                break;
            }
        }
    }
    
    return equal;
}
```

  * member methods, _but asymmetric in relation to automatic type conversion (e.g. `25 + object`) is illegal since 25 cannot be a calling object_

```c++
// File: Point.h

class Point {
    int dimensions;
    double *values;
    
public:
    // constructors, operator=, destructor
    // other public methods
    
    int getDims() { return dimensions; }
    
    // operators
    bool operator==(const Point &);
};

bool operator=(const Point &a, const Point &b);

// ...

// File: Point.cpp

#include "Point.h"

bool Point::operator=(const Point &b) {
    bool equal = false;
    
    if (dimensions == b.dimensions) {
        bool equal = true;
        for (int i = 0; i < dimensions ; i ++) {
            if (values[i] != b.values[i]) {
                equal = false;
                break;
            }
        }
    }
    
    return equal;
}
```

  * `friend` functions, _but breaks OOP encapsulation; despite that, it's the preferred choice for operators that can be overloaded as non-member functions_

```c++
// File: Point.h

class Point {
    int dimensions;
    double *values;
    
public:
    // constructors, operator=, destructor
    // other public methods
    
    int getDims() { return dimensions; }
    
    // operators
    friend bool operator=(const Point &a, const Point &b);

};

bool operator=(const Point &a, const Point &b);

// ...

// File: Point.cpp or other

#include "Point.h"

bool operator=(const Point &a, const Point &b) {
    bool equal = false;
    
    if (dimensions == b.dimensions) {
        bool equal = true;
        for (int i = 0; i < dimensions ; i ++) {
            if (values[i] != values[i]) {
                equal = false;
                break;
            }
        }
    }
    
    return equal;
}
```

The overloaded assignment operator (`operator=`) has to be a member. Why? Because the compiler always creates a default member `operator=` if there isn't one. This includes the case when there is one implemented as a non-member. Due to the specific algorithm C++ uses to determine which implementation it should use for a method (aka overload resolution), this situation might easily lead to inconsistent usage of different `operator=` implementations usage (depending on the scope and implementation visibility of the code executed).

### Pointers (8)

Use `typedef` to define pointer types. It makes passing pointer types by reference (e.g. use for `Point` objects in clustering where we don't want to have duplicates of points) much cleaner and readable.

```c++
typedef Point * PointPtr;

void foo(Point *&p); // without type alias

void bar(PointPtr &q); // using type alias
```

