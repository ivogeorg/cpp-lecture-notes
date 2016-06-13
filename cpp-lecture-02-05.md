### Class design (5)

Header for the `Point` class. This is a multi-dimensional point. The `distanceTo` method assumes Euclidean space.

```c++
#ifndef CLUSTERING_POINT_H
#define CLUSTERING_POINT_H

#include <iostream>

namespace Clustering {

    class Point {
        int dim;        // number of dimensions of the point
        double *values; // values of the point's dimensions

    public:
        Point(int);
        Point(int, double *);

        // Big three: cpy ctor, overloaded operator=, dtor
        Point(const Point &);
        Point &operator=(const Point &);
        ~Point();

        // Accessors & mutators
        int getDims() const { return dim; }
        void setValue(int, double);
        double getValue(int) const;

        // Functions
        double distanceTo(const Point &) const;

        // Overloaded operators

        // Members
        Point &operator*=(double);
        Point &operator/=(double);
        const Point operator*(double) const; // prevent (p1*2) = p2;
        const Point operator/(double) const;

        double &operator[](int index) { return values[index - 1]; } // TODO out-of-bds?

        // Friends
        friend Point &operator+=(Point &, const Point &);
        friend Point &operator-=(Point &, const Point &);
        friend const Point operator+(const Point &, const Point &);
        friend const Point operator-(const Point &, const Point &);
        
        friend bool operator==(const Point &, const Point &);
        friend bool operator!=(const Point &, const Point &);
        
        friend bool operator<(const Point &, const Point &);
        friend bool operator>(const Point &, const Point &);
        friend bool operator<=(const Point &, const Point &);
        friend bool operator>=(const Point &, const Point &);

        friend std::ostream &operator<<(std::ostream &, const Point &);
        friend std::istream &operator>>(std::istream &, Point &);
    };

}
#endif //CLUSTERING_POINT_H
```

Header for the `Cluster` class. This is a container for `Point` objects. It preserves the set of `Point` objects in space by holding pointers to them. It does not allocate the `Point` objects itself.

```c++
#ifndef CLUSTERING_CLUSTER_H
#define CLUSTERING_CLUSTER_H

#include "Point.h"

namespace Clustering {

    typedef Point *PointPtr;
    typedef struct LNode *LNodePtr;

//    struct LNode;
//    typedef LNode *LNodePtr;

    struct LNode {
        PointPtr p;
        LNodePtr next;
    };

    class Cluster {
        int size;
        LNodePtr points;

    public:
        Cluster() : size(0), points(nullptr) {};

        // The big three: cpy ctor, overloaded operator=, dtor
        Cluster(const Cluster &);
        Cluster &operator=(const Cluster &);
        ~Cluster();

        // Set functions: They allow calling c1.add(c2.remove(p));
        void add(const PointPtr &);
        const PointPtr &remove(const PointPtr &);

        // Overloaded operators

        // IO
        friend std::ostream &operator<<(std::ostream &, const Cluster &);
        friend std::istream &operator>>(std::istream &, Cluster &);

        // Set-preserving operators (do not duplicate points in the space)
        // - Friends
        friend bool operator==(const Cluster &lhs, const Cluster &rhs);

        // - Members
        Cluster &operator+=(const Cluster &rhs); // union
        Cluster &operator-=(const Cluster &rhs); // (asymmetric) difference

        Cluster &operator+=(const Point &rhs); // add point
        Cluster &operator-=(const Point &rhs); // remove point

        // Set-destructive operators (duplicate points in the space)
        // - Friends
        friend const Cluster operator+(const Cluster &lhs, const Cluster &rhs);
        friend const Cluster operator-(const Cluster &lhs, const Cluster &rhs);

        friend const Cluster operator+(const Cluster &lhs, const PointPtr &rhs);
        friend const Cluster operator-(const Cluster &lhs, const PointPtr &rhs);

    };

}
#endif //CLUSTERING_CLUSTER_H
```

### Operator overloading (2)

Overloaded operators are syntactic sugar: C++ provides this feature to allow us to write more natural code and to express operations with user-defined types more intuitively. For this to work, overloading has to follow these general principles:

  1. Whenever the meaning of an operator is not obviously clear and undisputed, it should not be overloaded.
  2. Always stick to the operator’s well-known semantics.
  3. Always provide all out of a set of related operations.

In short, to overload an operator for a particular type, the operator has to make sense, it should work predictably (that is, just like it works for primitive types), and all related operators should also be provided.

The comparison operators form a natural group and are useful for a great number of types. In addition, they only require actual implementation of two of them to provide all:

```c++
inline bool operator==(const X& lhs, const X& rhs){ /* do actual comparison */ }
inline bool operator!=(const X& lhs, const X& rhs){return !operator==(lhs,rhs);}
inline bool operator< (const X& lhs, const X& rhs){ /* do actual comparison */ }
inline bool operator> (const X& lhs, const X& rhs){return  operator< (rhs,lhs);}
inline bool operator<=(const X& lhs, const X& rhs){return !operator> (lhs,rhs);}
inline bool operator>=(const X& lhs, const X& rhs){return !operator< (lhs,rhs);}
```

Notice that these are non-member functions (they take two arguments), so they should be declared friend. Also, if the comparison is not very simple, they should not be declared inline.

The arithmetic operators form another group but make sense for fewer types. They make sense for our double-valued Point class. Moreover, the right-hand side argument can be of two different types, double and Point, and the operators have different meanings in these cases. For example:

```c++
// Point.h
Point &operator/=(double);
const Point operator/(double) const;
friend Point &operator+=(Point &, const Point &);
friend const Point operator+(const Point &, const Point &);

// Point.cpp
Point &Point::operator/=(double d) { // TODO handle div-by-0 or let runtime handle it
    for (int i = 0; i < dim; i++)
        values[i] /= d;
    return *this;
}

const Point Point::operator/(double d) const {
    return Point(*this) /= d;
}

Point &operator+=(Point &lhs, const Point &rhs) { // TODO exception
    if (&lhs == &rhs) {
        return lhs *= 2;
    } else if (lhs.dim == rhs.dim) {
        for (int i = 0; i < lhs.dim; i++)
            lhs.values[i] += rhs.values[i];
    }
    return lhs;
}

const Point operator+(const Point &lhs, const Point &rhs) {
    Point p(lhs);
    return p += rhs;
}

// main.cpp
Point p1, p2;
// ...
Point p3 = (p1 + p2) / 2;
// ...
```

By overloading arithmetic operators for `Point`, we can have the intuitive expression `(p1 + p2)/2;` where `p1` and `p2` are objects of class `Point`. 

Notice:

  1. That there are operators that take a double on the right-hand side. In this case, `operator/=` and `operator/`.
  2. That `operator/=` and `operator/` are implemented as members.
  3. That there are operators that take a Point on the right-hand side. In this case, `operator+=` and `operator+`.
  4. That `operator+=` and `operator+` are implemented as non-member `friend` functions.
  5. That `operator/` is implemented using `operator/=` and `operator+` is implemented using `operator+=`. This is a **general rule** for code reuse.
  6. That `operator/` and operator+ return new objects (by value, which is copied to a temporary object that can be assigned to a variable) and do not modify the, while the compound (aka shorthand) assignment operators `operator/=` and `operator+=` return a reference to the left-hand (or calling) object, which they modify.
  7. With these overloaded compound (aka shorthand) assignment operators we can also have expressions like this: `p += p5; p /= 2;` where `p` and `p5` are objects of class `Point`.

For the multidimensional `Point` class it makes sense to overload the array selection (aka direct access) operator `operator[]`. The canonical implementation is as follows:

```c++
double &operator[](int index) { // TODO: out-of-bounds exception
    return values[index]; 
}
```

Notice that this implementation allows assignment.

### Constructors (3)

For binary operators, it make sense to overload them as friend non-member functions, because of their symmetry. Specifically, this has to do with constructors and automatic type conversion. For example:

```c++
class Type {
    public:
    Type(int foo) { }

    Type operator+(const Type& type) const { // member operator+
        return *this; 
    }
};

// You could then write:
Type a = Type(1) + Type(2); // OK
Type b = Type(1) + 2;       // Also OK: conversion of int(2) to Type

// But you could NOT write:
Type c = 1 + Type(2); // DOES NOT COMPILE: 1 cannot be converted to Type
// Having operator+ as a non-member function allows the last case as well.
```

### Overloading operators (3)

Any of the following 38 operators can be overloaded:

```c++
+ 
- 
* 
/ 
% 
ˆ 
& 
| 
~ 
! 
= 
< 
> 
+= 
-= 
*= 
/= 
%= 
ˆ= 
&= 
|= 
<< 
>> 
>>= 
<<= 
== 
!= 
<= 
>= 
&& 
|| 
++ 
-- 
, 
->* 
-> 
( ) 
[ ]
```

Restrictions:

  1. operator=, operator[], and operator-> have to be overloaded as members.
  2. The operators :: (scope resolution), . (member access), .* (member access through pointer to member), and ?:(ternary conditional) cannot be overloaded
  3. New operators such as **, <>, or &| cannot be created
  4. The overloads of operators &&, ||, and , (comma) lose their special properties: short-circuit evaluation andsequencing (Links to an external site.).
  5. The overload of operator -> must either return a raw pointer or return an object (by reference or by value), for which operator -> is in turn overloaded.
  6. It is not possible to change the precedence, grouping, or number of operands of operators.

### L-values and R-values

In C++ you might notice a lot of talk about right-hand side and left-hand side, as well as, more generally, L-values and R-values. This comes especially in assignments and operator overloading and is due to a certain lack of symmetry. The two sides are not equivalent because, for example, assignment is a non-symmetric operation: the convention is that the right-hand side is assigned to the left-hand side. When the consequences from this asymmetry are traced down through the rest of the features of the language, a number of issues come up. There is a good discussion [here](http://eli.thegreenplace.net/2011/12/15/understanding-lvalues-and-rvalues-in-c-and-c).


