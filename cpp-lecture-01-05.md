### Using const

The keyword const can be used to declare constant variables __(LOL!)__, constant functions, constant arguments, and constant return values. It is left-associative (i.e. modifies what's on the left) except when it is leftmost and then it is right-associative (i.e. it modifies what's on the immediate right).

```c++
#include <iostream>
#include <iomanip>

class Point {
    double x, y;
public:
    Point();
    Point(double x, double y);
    double getX() const; // constant method
    double getY() const;
    double distanceTo(const Point &) const;
};

Point::Point() {
    x = 0;
    y = 0;
}

Point::Point(double newX, double newY) {
    x = newX;
    y = newY;
}

double Point::getX() const {
    return x;
}

double Point::getY() const {
    return y;
}

double Point::distanceTo(const Point &other) const { // constant method taking a constant argument
    // some code that uses constant methods getX and getY
}

int main() {
    const Point origin; // constant
    Point p(5, 6);
    
    cout << fixed << setprecision(2) << p.distanceTo(origin) << endl;
}
```

### Default class methods

Since classes are full-fledged types, so everything we do with a primitive type we should be able to do with a class type, every class should have a default constructor, a copy constructor, an assignment operator, and a destructor. A class which only has static (non-pointer) members does not need to explicitly declare a default constructor, copy constructor, assignment operator (=), or destructor. They are supplied by the compiler.

```c++
class Point {
    double x, y;
public:
    Point(double, double);
    double getX() const;
    double getY() const;
    double distanceTo(const Point &) const;
};

int main() {
    Point p1, p2(p1), p3 = p2; // default, copy, assignment
}
```

### Dynamic allocation

There are two main memory regions where a program's variables are allocated (i.e. memory is reserved for them), the (call) stack and the heap. All local/static/automatic variables are allocated on the stack, whereas all dynamically/explicitly allocated variables are allocated on the heap. The heap is usually much larger than the stack. Objects on the stack use the dot (.) operator to select members and call member methods, while those on the heap use the member selection (->) operator.

### Pointers (1)

Pointers are variables which hold addresses of other variables and are the primary facility for dynamic allocation. A pointer may itself be allocated on the stack but it (usually) points to an object that is allocated on the heap. Primitives and objects are allocated on the heap with the new operator and are cleaned up with the delete or delete [] operators. The allocation and cleanup are the responsibility of the programmer. Failing to cleanup when done results in a memory leak.

```c++
#include "Point.h"

int main() {
    Point *pt = new Point; // pt on stack, new Point object allocated on heap
    Point p1(5, 6); // local Point object on the stack
    Point *px = &p1; // px points to a stack object
    
    // do something with Point objects
    
    delete pt; // release allocated Point object
    pt = nullptr; // set pointer to null pointer
    px = nullptr; // no realease necessary, since object was on stack and is cleaned up automatically when it goes out of the scope of the main function
}
```

### Arrays & dynamic arrays

An array variable is a (constant) pointer to the array base type. An array can be allocated on the heap. Such an array is called a dynamic array. A dynamic array MUST be released with the **delete []** operator.

```c++
#include <iostream>
#include "Point.h"

int main() {
    Point arrayOfPoints[100]; // normal array on the stack
    
    for (int i=0; i<100; i++)
        cout << arrayOfPoints[i].getX() << endl;
        
    Point *pt = arrayOfPoints; // pointer!
    
    for (int i=0; i<100; i++)
        cout << pt[i].getX() << endl; // same as arrayOfPoints
    
    Point *arrayOfHeapPoints = new Point[50];
    
    for (int i=0; i<100; i++)
        arrayOfPoints[i].setX(i); // NOTE: dot, not -->

    for (int i=0; i<100; i++)
        cout << arrayOfHeapPoints[i].getX() << endl;
        
    // or, equivalently
    for (int i=0; i<100; i++)
        cout << (*(arrayOfHeapPoints + i)).getX() << endl; // this is called pointer arithmetic

    delete [] arrayOfHeapPoints;
    arrayOfHeapPoints = nullptr;
}
```

A more in-depth demonstration of the three ways to declare a one-dimensional array of Point objects follows:

```c++
#include <iostream>

#include "Point.h"

using namespace std;

int main() {
    cout << "Hello, World!" << endl;

    const int ARRAY_SIZE = 10;

    // Three ways to declare/allocate an array of Point objects

    // (1) Static allocation
    Point staticArray[ARRAY_SIZE];

    for (int i = 0; i < ARRAY_SIZE; ++i) {
        staticArray[i].setX(4);
        staticArray[i].setY(4);
    }

    for (int i = 0; i < ARRAY_SIZE; ++i) {
        cout << staticArray[i] << endl;    // NOTE: staticArray is of type Point * (Point sth[])
    }
    cout << endl;

    // (2) Dynamic allocation
    Point *dynamicArray = new Point[ARRAY_SIZE];

    for (int i = 0; i < ARRAY_SIZE; ++i) {
        cout << dynamicArray[i] << endl;    // NOTE: dynamicArray is also of type Point * (Point *sth)
    }
    cout << endl;

    delete [] dynamicArray;
    cout << endl;

    // (3) Heap (double-pointer) allocation
    Point **heapArray = new Point*[ARRAY_SIZE];

    for (int i = 0; i < ARRAY_SIZE; ++i) {
        heapArray[i] = new Point(6, 6);
    }

    for (int i = 0; i < ARRAY_SIZE; ++i) {
        cout << *heapArray[i] << endl;    // NOTE: dereference each pointer to get the Point!!!
    }
    cout << endl;

    for (int i = 0; i < ARRAY_SIZE; ++i) {
        delete heapArray[i];
    }

    delete [] heapArray;
    cout << endl;


    return 0;
}
```

The Point.h file follows. __NOTE: All class methods are implemented inline and not in a corresponding source file Point.cpp. This is for compactness. You should implement your methods in a source file.__

```c++
#include <iostream>

using namespace std;

class Point {
    double __x, __y;
public:
    Point() {
        __x = __y = 0;
    }

    Point(double x, double y) {
        __x = x; __y = y;
    }

    Point(const Point &another) {
        __x = another.__x * 2;
        __y = another.__y * 3;
    }

    ~Point() {
        cout << "Destructor of point " << *this << " called" << endl;
    }

    void setX(double x) { __x = x; }
    void setY(double y) { __y = y; }

    double getX() { return __x; }
    double getY() { return __y; }

    friend std::ostream &operator<<(std::ostream &os, const Point &p) {
        // (x, y)
        os << '(' << p.__x << ", " << p.__y << ')';

        return os;
    }
};
```


