### STL (2)

Just as we used `std::vector` (a resizable array) for the `__values` member of the templatized `Point` class, we can now show how to use `std::forward_list` (a singly-linked list) for the `__points` member of a templatized `Cluster` class:

```c++
// PointVD.h
//
// Created by Ivo Georgiev on 10/25/15.
//

#ifndef TEMTEST_POINTVD_H
#define TEMTEST_POINTVD_H

#include <vector>
#include <ostream>

template <typename T, int dim> class PointVD;
template <typename T, int dim> std::ostream &operator<<(std::ostream &os, const PointVD<T, dim> &p);
template <typename T, int dim> bool operator<(const PointVD<T, dim> &rhs, const PointVD<T, dim> &lhs);

template <typename T, int dim>
class PointVD {
    unsigned int __dimensions;
    std::vector<T> __values;

public:
    PointVD() :
            __dimensions(dim),
            __values(__dimensions)
    {}
    unsigned int getNumDimesnsions() { return __dimensions; }
    T &operator[](unsigned int i) { return __values[i]; }
    friend std::ostream &operator<< <> (std::ostream &os, const PointVD<T, dim> &p);
    friend bool operator< <> (const PointVD<T, dim> &rhs, const PointVD<T, dim> &lhs);
};

template <typename T, int dim>
std::ostream &operator<<(std::ostream &os, const PointVD<T, dim> &p) {
    auto it = p.__values.begin();
    for ( ; it != --p.__values.end(); ++ it)
        std::cout << *it << ", ";
    std::cout << *it;

    return os;
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
};

template <typename T, int dim>
bool operator<(const PointVD<T, dim> &rhs, const PointVD<T, dim> &lhs) {
    if (rhs.__dimensions != lhs.__dimensions)
        throw DimensionalityMismatchEx(rhs.__dimensions, lhs.__dimensions);

    bool isLess = false;
    for (int i = 0; i < rhs.__dimensions; i++) {
        if (rhs.__values[i] < lhs.__values[i]) {
            isLess = true;
            break;
        } else if (rhs.__values[i] > lhs.__values[i]) {
            isLess = false;
            break;
        } // equal, so continue
    }

    return isLess;
}

#endif //TEMTEST_POINTVD_H

// ClusterFD.h
//
// Created by Ivo Georgiev on 10/26/15.
//

#ifndef TEMTEST_CLUSTERFD_H
#define TEMTEST_CLUSTERFD_H

#include <forward_list>

using std::forward_list;

template<class T, int dim> class ClusterFD;
template<class T, int dim> std::ostream &operator<<(std::ostream &os, const ClusterFD<T, dim> &c);

template<class T, int dim>
class ClusterFD {
    unsigned int __dimensionality;
    forward_list<T> __points;

public:
    ClusterFD() :
            __dimensionality(dim),
            __points()
    {}
    void add(const T &t) { __points.push_front(t); }
    void sort() { __points.sort(); }
    friend std::ostream &operator<< <> (std::ostream &os, const ClusterFD<T, dim> &c);
};

template<class T, int dim>
std::ostream &operator<<(std::ostream &os, const ClusterFD<T, dim> &c) {
    for (auto it = c.__points.begin(); it != c.__points.end(); ++ it)
        std::cout << *it << std::endl;

    return os;
};

#endif //TEMTEST_CLUSTERFD_H

// main.cpp
#include <iostream>
#include "PointVD.h"
#include "ClusterFD.h"

using std::cout;
using std::endl;

int main() {
    const unsigned int NUM_POINTS = 5;
    const unsigned int NUM_DIMS = 10;

    ClusterFD<PointVD<double, NUM_DIMS>, NUM_DIMS> c;

    for (int i = 0; i < NUM_POINTS; i ++) {
        PointVD<double, NUM_DIMS> p;
        for (int j = 0; j < NUM_DIMS; j ++) p[j] = 10.4 * i - 0.35 * j;
        c.add(p);
    }

    cout << c;
    c.sort();
    cout << c;

    return 0;
}
```

This code is a springboard for the implementation of PA4 and has a number of interesting points:

**//TODO**

Templatized class `Point` with template parameters the type of the points values and the number of dimensions.
Templatized class `Cluster` with template parameters the type of the objects for clustering and the number of dimensions. Note: Since we don't know what the object is beforehand (no way to constrain that), we can't know if it even makes sense to talk about dimensionality. This is a design flaw which comes from the requirement for continuity between the assignments. In this case, the templatized `Cluster` class template is meant to be instantiated with a `Point` template class.
Templatized friend non-member operators that are specific for the type template argument and deduce their own type at compile time.
STL container iterators. Notice how we avoid writing a comma after the last point value.
An exception thrown upon dimensionality mismatch of two points being compared.
A demonstration of the `std::sort(`) method for STL containers. Note: This is a lazy and expensive way to keep the elements in a container sorted, and it is just used as a demonstration. It might or might not work for our purposes. The `ClusterFD::add()` method can still be implemented in a way to keep the objects in order.

**//TODO**

### Class design (8)

In object-oriented programming (OOP) the term object state refers to the set of values of all object data at a particular moment in its lifecycle. Possibly the highest concern in OOP is to maintain object state valid (aka consistent). Object state is valid when all assumptions, conditions, and expectations about an object's normal relation to the execution environment (aka class invariants) are satisfied. Effectively, this means that any class method applied to an object in a valid state will leave it in a valid state. Object state should be valid at all times except maximally briefly within methods calls.

We have already seen a case that illustrated the issue of object state validity: the `centroid` member of the `Cluster` class. Originally, the `Cluster` manipulated `Point` objects without being aware of their dimensionality, but when it became necessary for it to compute centroids, it also became necessary to add dimensionality to the data (and therefore, state) of the class, so that the centroid member could be initialized. This quickly led to the requirement to add a dimensionality parameter to all `Cluster` constructors. Once that is done, the `Cluster` object state can be maintained valid without any concern at what point the centroid is available for manipulation.

### Constructors (4)

Naturally, the primary role of constructors is to initialize an object into a valid state. All initialization should happen in the constructor, and no part of the initialization should be left for "later" to other methods that the user has to call.

This leads to a curious situation where a non-pointer class member does not have a default constructor. How can it be initialized? Again, the centroid member, which in the following implementation is a member of class `Point`, provides an example, since `Point` has not default constructor but requires an unsigned int as the number of dimensions.

Class members of class types are initialized by their default constructors. If the member class does not have one, the containing class's constructor has to explicitly initialize the member, providing the necessary arguments. In the following example, `Point` has no default constructor and its constructor takes an unsigned int. `Cluster` has a private member `__centroid` of class `Point`. In its constructor, it explicitly initializes `__centroid` by invoking the `Point` constructor.

```c++
// Point.h
//
// Created by Ivo Georgiev on 10/18/15.
//

#ifndef INITTEST_POINT_H
#define INITTEST_POINT_H

namespace Clustering {
    class Point {
        double *__values;
        unsigned __dimensions;

    public:
        Point(unsigned d) : __dimensions(d), __values(nullptr) {}
    };
}

#endif //INITTEST_POINT_H


// Cluster.h
//
// Created by Ivo Georgiev on 10/18/15.
//

#ifndef INITTEST_CLUSTER_H
#define INITTEST_CLUSTER_H

#include "Point.h"

namespace Clustering {
    class Cluster {
        unsigned __dimensionality; // notice the order!!!
        Point __centroid;

    public:
        Cluster(unsigned d) :
                __centroid(__dimensionality), // invokes ctor Point(int)
                __dimensionality(d)
        {}

    };
}

#endif //INITTEST_CLUSTER_H


// main.cpp
#include <iostream>
#include "Cluster.h"

using namespace std;
using Clustering::Cluster;
using Clustering::Point;

int main() {
    const unsigned DIMENSIONALITY = 10;
    Cluster c(DIMENSIONALITY);

    return 0;
}
```

Notice that class members are initialized in the order in which they appear in the class declaration, regardless of the order they appear in the constructor initialization list.

Once again, we see that constructors take care of initialization. The change of object state validity from invalid to valid happens upon execution of the constructor. Constructors should be written in a way that initialization results in a valid object state.

### Class design (9)

The above discussion on object state validity and the techniques for jumpstarting and maintaining it leads to a very important C++ _idiom_: Resource Acquisition Is Initialization (RAII). It means that any object that is acquired for any purpose should be initialized properly by the end of the execution of the constructor. RAII postulates that an object's lifecycle begins in a valid state. It also includes the requirement that an object's lifecycle also ends in a valid state (here the meaning of valid becomes global, in the sense of the relationship of the object to the total execution environment), by requiring that the class destructor performs full cleanup and completely scraps the object.

### Exceptions (4)

We mentioned earlier that exceptions are a part of a class' interface. This is precisely because they help preserve the validity of object state.

If a constructor cannot perform full initialization to satisfy RAII, an exception can be thrown from it. Throwing an exception from a constructor is part of RAII.

On the other hand, destructors should never throw exceptions, because they are the last chance for an object in any state, valid or invalid, as well as any dynamic resources the object has acquired and still holds on to, to be garbage-collected.
