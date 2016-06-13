### Pointers (4)

Pointers are powerful, so you have to be careful. Pointers hold addresses of:

  1. Statically or locally allocated variables of some base type (primitive or complex, standard or user) and are assigned by the use of the "address-of" operator &. The objects they point to are on the stack and are deallocated automatically.
  2. Dynamically or locally allocated variables of some base type (primitive or complex, standard or user) and are assigned by the use of the new operator. The objects they point to are on the heap and have to be deallocated (aka freed) explicitly by the programmer.

```c++
class Point {
    double x, y;
public:
    static Point origin; // book pp.310-311

    Point() : Point(0, 0);
    Point(double, double);
};

Point Point::origin = Point(); // statically allocated

int main() {
    Point p; // locally allocated
    
    Point *pt = &p;
    Point *ps = &Point::origin; // strange and useless, but legal
    
    // both pt and ps point to statically allocated memory,
    // that is, the objects *pt and *ps will be cleared
    // automatically by the program when they go out of
    // scope and/or the program terminates
    
    Point *pd = new Point(); // don't forget to delete!
    
    
    // ...
    // some code using the Point objects and their pointers
    // pt, ps, and pd
    // ..
    
    delete pd;
    pt = ps = pd = nullptr;
}
```

Being powerful, pointers can break encapsulation and isolation if you aren't careful. For example, passing pointers by value as arguments to functions allow the functions to modify objects that are otherwise outside of their scope!

```c++
#include <iostream>
#include "Point.h"

void foo(Point *); // forward declaration

int main() {
    Point p;
    
    cout << p.getX() << ", " << p.getY() << endl;
    
    foo(&p); // this is just like Point *pt = &p;
    
    cout << p.getX() << ", " << p.getY() << endl;
}

void foo(Point *pt) {
    pt->setX(3.14);
    pt->setY(-3.14);
}
```

You can also return pointers from functions. Never return pointers to local variables because they go out of scope when the function completes and is popped off the call stack. You can give otherwise unintended access to private members of a class in a similar way. 

This is not accidental. In C, the predecessor of C++, this was a way to "return more than one value", by passing in pointers to outside variables. In fact, structures are just a formalization of this general need, and a clear motivation for complex types. Later they evolved into classes and, since this "open-doors" access was extremely error-prone, the default access for classes was set to private for classes. Explicit pointers have in many situations been replaced by references in C++. References have the syntax of local variables, but have pointer semantics (that is, operations on them are operations on the objects they point to, just like pointers). However, pointers remain one of the primary facilities of the language to give bare-bones access to the underlying memory for time-critical (e.g. online stock trading), big-data or severely constrained (e.g. space hardware) applications.

### Pointers (5)

Remember: arrays are pointers. Passing an array as an argument to a function is exactly the same as passing a pointer, though the array argument is explicit (e.g. `Point pts[]` instead of `Point *pt`). Inside the function, the array can be manipulated directly as if it is local.

However, one cannot return an array. Instead, one can return a pointer of the same type!

```c++
#include "Point.h"

Point *foobar(Point [], int); // forward declaration

int main() {
    Point pts[100], *dbl;
    
    dbl = foobar(pts, 100);
    
    dbl[145].setX() = dbl[167].getY();
    
    delete [] dbl;
}

// caller has to free the returned dynamic array
Point *foobar(Point pts[], int size) {
    Point *doublePts = new Points[size];
    for (int i=0; i<size; i++)
        doublePts[i] = pts[i]; // notice same syntax
    return doublePts;
}
```

Notice that static/local and dynamic arrays have the same syntax, as in the `foobar()` function above. 

### Pointers (6)

Working with raw pointers can get tedious and error-prone due to the less-than-intuitive pointer syntax. One way to deal with that is to define pointer type aliases with the typedef keyword (e.g. typedef Point * PointPtr;). Subsequent use of the type alias follows exactly the same syntax as for builtin types and static/local allocation. Of course, this is a pointer and, when dynamic allocation is required, the new operator has to be used, and cleanup has to be performed with the **delete** operators.

Specifically, working with multidimensional arrays and, equivalently, dynamic arrays of pointers, is very tedious without a pointer type alias, because it calls for double star * operators (__**__).

```c++
#include "Point.h"

int main() {
    // a matrix of integers without Point * type alias
    int **matrix = new int *[100];
    for (int i = 0; i < 100; i++)
        matrix[i] = new int[100];

    for (int i = 0; i < 100; i++) {
        for (int j = 0; j < 100; j++) {
            matrix[i][j] = 10*i + j;
        }
    }

    for (int i = 0; i < 100; i++) {
        for (int j = 0; j < 100; j++)
            cout << "(" << matrix[i][j] << "), ";
        cout << endl;
    }

    for (int i = 0; i < 100; i++)
        delete [] matrix[i];
    delete [] matrix;

    // a matrix of integers without Point * type alias
    typedef int * IntPtr;

    IntPtr *matrix2 = new IntPtr[100];
    for (int i = 0; i < 100; i++)
        matrix2[i] = new int[100];

    for (int i = 0; i < 100; i++) {
        for (int j = 0; j < 100; j++) {
            matrix2[i][j] = 20*i + j;
        }
    }

    for (int i = 0; i < 100; i++) {
        for (int j = 0; j < 100; j++)
            cout << "(" << matrix2[i][j] << "), ";
        cout << endl;
    }

    for (int i = 0; i < 100; i++)
        delete [] matrix2[i];
    delete [] matrix2;

}
```

`IntPtr` is not a new type, just an alias for the type integer pointer type `int *`. With `IntPtr` the integer matrix (a 2-dimensional array) looks much more like the straightforward one-dimensional dynamic array. Of course, you have two levels of **new** declarations and **delete []** statements.

Let's look how this works with an alias for a `Point` pointer type `Point *` for a one-dimensional dynamic array of pointers to `Point` objects. This is different from the dynamic array of Point objects where the dynamic array is allocated on the heap and it contains a sequence of adjacent Point memory slots. In the case of the pointers to `Point`, the dynamic array contains adjacent pointers to `Point` which, when allocated, will point to separate dynamic `Point` objects, also on the heap. 

```c++
#include "Point.h"

int main() {
    typedef Point *PointPtr; // pointer type alias definition
    
    PointPtr *points = new PointPtr[100]; // 100 pointers to Point objects, yet unallocated
    
    for (int i = 0; i < 100; i++)
        points[i] = new Point(2.3, 4.5); // allocate objects

    // note that we use the --> member selection operator here
    for (int i = 0; i < 100; i++)
        cout << "points[" << i << "] = (" << points[i]->getX() << ", " 
        << points[i]->getY() << ")" << endl;

    // two levels of cleanup!
    for (int i = 0; i < 100; i++)
        delete points[i];
    delete[] points;
}
```

This will come in handy for the Cluster class when we want to keep track of the same points but move them from one cluster to another. Since each Point object is now separately allocated on the heap, all we need to move points around is to transfer pointers of type PointPtr from the array of one `Cluster` to the array of another. 

### Constructors (1)

Constructors are the class members executed to initialize objects of class types. Before the constructor is called, nothing in an object is initialized. Just raw memory, allocated for its use!

Constructors don't have a return type (the return type is in their name), but return anonymous objects that can be assigned to (named) variables.

If no constructors are specified in a class, a default constructor is generated by the compiler. If at least one constructor is specified, no default constructor is generated. It is a good practice for a programmer to create a default constructor if she creates any constructors at all, to avoid the almost inevitable case that an object of the class won't be able to be initialized by default for lack of a default constructor). The default constructor is needed to initialize class objects without any parameters.

```c++
class Point {
    double x, y;
public:
    Point();
    Point(const Point &);
    Point(double, double);
};

int main() {
    Point p; // note: no parentheses for default constructor
    
    p = Point(); // note: parentheses on explicit call to default constructor
    
    Point p1 = p; // note: copy constructor called, not assignment operator
    
    p1 = Point(4, 5); // note: assignment operator called
}
```

### Constructors (2)

One constructor can delegate the initialization to another by specifying it in its initialization list. Constructor has to be the only one in the list!

```c++
class Cluster {
    Point *cloud;
    int size, capacity;
public:
    Cluster() : Cluster(10) {} // inline delegation
    Cluster(int, int);
    Cluster(const Cluster &);
    ~Cluster();
};
```

Note that constructor delegation can be done inline or not. If inline, don't forget the empty body _{ }_.

### Class design (3)

Our `Cluster` class needs to hold a dynamic array of Point objects, which can be added or removed, without regard of order. Depending on how we want to use the Cluster and what we want to be able to do with Point objects, we are faced with some design decisions.

First, do we use an array or a different aggregate data structure. Because we don't care about the order of Point objects in the Cluster, and we want to be able to remove any object and to add any number of objects, an array is not the best choice. For example, if we want to remove a Point which is not at the end of the array, we need to move all the Point objects after that point one place up to fill the gap left after the deletion. From the classic data structures, the linked list is the best one to support random removal. It is inherently ordered but we can ignore order for the Point objects, since there is no straightforward way to order multidimensional points. We will leave this implementation for PA2, and implement the Cluster class with an array for illustration. The consequence of this design choice is that we need to take care of the memory management, including expanding and shrinking the array, whenever allocated space runs out or can be cleaned up. Since the allocated array will have at all times room for at least as many Point objects as have been "added" to it, but could keep extra capacity, we have to keep track of both size and capacity.

Second, we need to decide if we are going to manipulate Point objects directly or through pointers. Since the number of Points will be known at the beginning of the program run, so no Point objects will be created and deleted during the run, but only their memberships in Cluster objects, it makes much more sense to allocate the Point objects once and only manipulate pointers to them to keep track of which Point belongs to each Cluster during the run. So, we are using a dynamically allocated array of pointers to Point (PointPtr).

Because we have dynamic memory to keep track of, we need to overload:

  1. The copy constructor `Cluster(const Cluster &);`
  2. The assignment operator `Cluster &operator=(const Cluster &);`
  3. The destructor `~Cluster();`

These are called "The Big Three" and the rule is that if a programmer needs one of them, she needs all three.

Let's look at the class definition so far:

```c++
#include "Point.h"

typdef Point *PointPtr;

class Cluster {
    PointPtr *points; // array of pointers to Point
    int size, capacity;
    
    void expand(float by); // expand by factor
    void shrink(float by); // shrink by factor

public:
    Cluster(); // default ctor
    Cluster(int, int); // ctor by size and capacity
    Cluster(const Cluster &); // copy ctor
    Cluster &operator=(const Cluster &); // assignment operator
    ~Cluster(); // dtor
    
    void add(const PointPtr &); // add a point
    PointPtr &remove(const PointPtr &); // remove a point and return it so we can add it to another cluster
};
```

Notice that the `expand()` and `shrink()` methods are private, since we only need them for the array-based implementation of the Cluster behavior. Another implementation, say based on a linked list, will not require explicit expansion and shrinking.

The copy constructor is invoked whenever a new object has to be created as a copy of another of the same class:

  1. When a new object is initialized with an existing object (e.g. `Cluster c2(c1);` and `Cluster c3 = c1;`).
  2. When an object is passed by value as an argument.
  3. When an object is returned by value.

Note that all constructors are not void functions but return anonymous objects of their class type.

```c++
#include "Cluster.h"

// Cluster.cpp
// Note: not all methods shown

// copy ctor
Cluster(const Cluster &rhs) 
: size(rhs.size), capacity(rhs.capacity) {
    points = new PointPtr[capacity];
    for (int i = 0; i < capacity; i++)
        points[i] = nullptr;        
    for (int i = 0; i < size; i++)
        points[i] = new Point(rhs.points[i]);        
}

// assignment oper
Cluster &operator=(const Cluster &rhs) {
    if (this == &rhs) {
        return *this;
    } else {
        if (points != nullptr) {
            for (int i = 0; i < size; i++)
                delete points[i];
            delete [] points;
        }
        
        this->size = rhs.size;
        this->capacity = rhs.capacity;

        this->points = new PointPtr[capacity];
        for (int i = 0; i < capacity; i++)
            this->points[i] = nullptr;        
        for (int i = 0; i < size; i++)
            this->points[i] = new Point(rhs.points[i]);        
    }
    
    return *this;
}

// dtor
~Cluster() {
    if (points != nullptr) {
        for (int i = 0; i < size; i ++)
            delete points[i];
        delete [] points;
    }
    points = nullptr;
}
```

The copy constructor and the assignment operator have equivalent function, but the former works on new objects, the latter on existing ones.

The overloaded assignment operator= is invoked when one object is assigned to another already existing object. It returns an object of the base type by reference. It has to support chaining (e.g. c1 = c2 = c3; and even crazy expressions like (c1 = c2) = c3;). It takes a const reference to the base object, because it does not modify the object passed as argument. It checks for self assignment (e.g. c1 = c2;) to prevent deleting its own members and attempting to recreate them. The assignment operator has to be overloaded as a member of the class because if it is not (the other two ways to overload are through non-member functions) the compiler will generate a default, which will do just shallow copy.

### Pointers (7)

The this pointer is a special pointer to the current object. It is accessible from inside the class member methods.

You can see its use in the above overloading of operator=. Let's look at it a bit closer and understand this usage:

```c++
#include "Cluster.h"

// assignment oper
Cluster &operator=(const Cluster &rhs) { // rhs - right-hand side
    if (this == &rhs) { // check for self-assignment (like c1 = c1;)
        return *this; // return the current object
    } else {
        // clean up the current object's members
        // copy over the static members of rhs
        // allocate new copies of the dynamic members or rhs
    }
    
    return *this; // return the current object
}
```

There are two statements that need to be understood: the self-assignment check, and returning the current object by reference.

The comparison (`this == &rhs`) compares two pointers. Since rhs is passed by reference, but references have local-variable semantics, it is &rhs (that is, the address of rhs) and not rhs itself, which is a pointer, comparable to this. If the two pointers are equal, then we have a case of self-assignment and we just return the current object without doing anything else.

Since this is a pointer to the current object, *this is what the pointer points to (that is, the pointer is being dereferenced), namely, the current object. Since the `operator=` signature specifies a return by reference of a `Cluster` object, the statement return *this; returns the current object by reference.

