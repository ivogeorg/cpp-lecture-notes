### Pointers (10)

Pointers hold addresses. At those addresses, there could be data or code. We are familiar with the former case - there are the well known pointers to dynamically allocated objects.

```c++
// Pointers to data

int *intPtr = new int(10);

double *dblPtr = new double;

char *cString = new char[100];

Point *ptr = new Point(20);

Cluster *ctr = new Cluster<Point<double, 20>, 20>;
```

The latter case is also used extensively in C++ - these are the so called pointers to functions. They have a similar but slightly different syntax.

```c++
#include <iostream>

int foo() {
    std::cout << "foo() called" << std::endl;
    return 0;
}

double bar(int i) {
    std::cout << "bar(" << i << ") called" << std::endl;
    return 0;
}

int main() {
    int (*fn1)() = foo;         // fn1 is a pointer to a function with no raguments and returning int
    double (*fn2)(int) = bar;   // fn2 is a pointer to a function taking an int and returning a double

    fn1();          // function call via implicit dereference
    fn2(10);        // function call via implicit dereference
    (*fn2)(10);     // function call via explicit dereference

    return 0;
}
```

They are used most frequently to implement a callback, essentially a part of the behavior that the user can specify while most of the algorithmic framework is provided by the codebase/library. A couple of examples:

  1. A comparison function that is used by a generic sorting function or class.
  2. A centroid calculation function that will be used in a clustering application.

Naturally, for such applications to work, function pointers need be able to appear as function arguments. Indeed they can:

```c++
// Function pointers

// (1) Comparison function as an argument to a sorting algorithm
void SelectionSort(int *anArray, int nSize, bool (*pComparison)(int, int));

// (2) Centroid calculation function as an argument to a clustering algorithm
void ClusterSpace(Point *pointSpace, int size, const Point (*computeCentroid)(Point *, int));

// Just as data pointers, function pointers can be aliased using typedef

// Data pointer alias
typedef Point * PointPtr;

// Function pointer alias
typedef const Point (*pCompCent)(Point *, int);

// With this function alias, the above ClusterSpace declaration becomes
void ClusterSpace(Point *pointSpace, int size, pCompCent computeCentroid);

// A little cleaner!
```

### Templates (6)

If we want to pass a function as a template argument, we have to create a functor, an class that implements the function operator operator().

**//TODO - CODE...**

### Inheritance (7)

the mechanics of `virtual` function implementations

the corresponding overhead of the mechanism 

virtual tables

inheritance, object storage, and addresses

object memory: base object, extension object, virtual table

`sizeof()`

memory alignment

### Inheritance (8)

slicing problem `Base b; Derived d; b = d; // d's members lost`

### Pointers (11)

pointers are the key to polymorphism

solution: dynamic memory `Base *b; Derived *d; b = d; // not lost, but need virtual functions to access them`

note that it's not `*b = *d; // that would have the same slicing problem`

### Inheritance (9)

which destructor is called upon `delete b;` ?

`virtual` destructors

### Inheritance (10)

upcasting (from derived to base)

downcasting (from base to derived)

`dynamic_cast<Derived *>`

RTTI

virtual table of object is used for dynamic dispatch, not the type of the pointer used

