### Build (2)

The build process is initiated when you press **Run-->Build** in CLion. It prepares your program for execution. It has three stages: 

  1. Preprocessor
  2. Compiler
  3. Linker
  
The preprocessor and compiler work on one file at a time. The linker puts together all files into a single executable program file.

The preprocessor takes a source code file and:

  * prepares the file for the compiler
  * handles all [preprocessor directives](http://www.cplusplus.com/doc/tutorial/preprocessor/) (e.g. `#include`, `#define`) and removes them
    * the #include directive results in the inclusion of the header file into the current file
    * expands in place all macros that have been defined with #define
    * substitutes in place all constants defined with #define
  * removes all comments
  * generates a translation unit, which is the input the compiler actually compiles

The compiler takes a translation unit and:

  * translates the C++ code into machine code (using the instruction format of the target [processor architecture](http://www.intel.com/content/dam/www/public/us/en/documents/manuals/64-ia-32-architectures-software-developer-instruction-set-reference-manual-325383.pdf))
  * produces an object file (**NOTE:** Not to be confused with C++ class _objects_.)
  * includes information about all declared functions and all defined functions in the object file

The linker takes all the files and:

  * resolves all function declarations and definitions across all files, making sure there is one and only one definition per function
  * assembles all object files into a single executable program

### Class design (1)

Inline functions are functions which:

  * are simple and fast (e.g. a single assignment)
  * do not need to incur the overhead of having a stack frame when called
  * are defined either within the class definition body or outside with a prefix inline
  * get their code substituted by the compiler every place they are called (so, should not be overused) by the rest of the code

Here is an example with the Point class:

```c++
class Point {
    double x, y;
public:
    double getX() const { return x; } 
    double getY() const { return y; }
    void setX(const double);
    void setY(const double);
};

inline void Point::setX(const double newX) {
    x = newX;
}

inline void Point::setY(const double newY) {
    y = newY;
}

int main() {
    Point a, b(3, 4.5), c(3, 2.5);
    
    // in all these four calls, the machine code for the inline
    // functions is substituted inline (that is, in place) within 
    // the machine code body of the main function
    a.setX(2.1);
    a.setY(0.7);
    
    cout << fixed << setprecision(2) << b.getX() << ' ' << c.getY();
}
```

Constructor initialization lists are a way to initialize class members before the constructor body is executed. This is important in some cases we will see in subsequent modules. Here is an example with the Point class:

```c++
class Point {
    double x, y;
public:
    Point();
    Point(const double, const double);
};

Point::Point() : x(0), y(0) {}

Point::Point(const double initX, const double initY) :
        x(initX), y(initY) {}
```

### Pointers (2)

To review, pointers are variables that point to blocks of memory allocated on the heap. The size of the allocated block depends on the base type of the pointer (e.g. int, double, Point, etc). Indeed, in modern C++ pointers have types and the compiler enforces proper type-aware usage. (In contrast, in C pointers were "raw" and were simply integers which happened to be memory addresses.) 

Pointers can be assigned in three ways:

  * with a newly allocated memory block on the heap (therefore, through the new operator)
  * to each other, when they are the same type (that is, they point to the same type of dynamically allocated objects)
  * with the "address of" operator **&** 

Dereferencing a pointer means accessing the dynamically allocated object it points to. Pointers are dereferenced with the __*__ operator.

```c++
#include "Point.h"

int main() {
    // 1. Examples of pointer assignment:
    
    Point p1, *pt = new Point(5, 6);
    // p1 is a Point on the stack
    // pt points to a Point on the heap

    Point *px = pt, *py = &p1;
    // px points to what pt points to
    // py points to p1, which is on the stack
    
    // 2. Examples of dereferencing
    
    pt->setY(6.5); // is equivalent to
    (*pt).setY(6.5); 
    // this is a dereference, followed by the dot operator
    // the parenteses are needed because . has higher
    // precedence than * (book pp.925-926)
    
    *py = *pt; // now we are assigning one Point to another
    // Here, *py is p1 and *pt is the Point pt points to
    // the result is that p1 becomes (5.0, 6.0)
    
    py = new Point[100]; // dynamic array
    for (int i=0; i<100; i++) {
        py[i].setY(i);
        (*(py + i)).setX(i+2);
        // remember that an array variable is a pointer
        // to the first element (element 0) of an array
        // so py[i] is equivalent to *(py + i), and thus
        // the [i] operator dereferences the i-th
        // element (counting from zero, of course)
    }
    
    return 0:
}
```
The last example of dereferencing is also an example of pointer arithmetic. Pointer arithmetic is a leftover feature from the time in the evolution of the C/C++ languages when pointers were simply integers. For backward compatibility, modern C++ cannot forbid pointer arithmetic. Essentially, it is a technique of calculating addresses that relies on explicit arithmetic operations on pointers. In this case, this is the _*(pointer + integer)_ statement, which accesses the integer-th element of the pointer array. This is equivalent to the much cleaner expression pointer[integer]. You should prefer the latter over the former, but it's good to understand this aspect of pointers.

Note that, since in C++ pointers have types (e.g. _pointer-to-int_, _pointer-to-Point_, etc.), pointer arithmetic works with the correct **step** in the array. This means that _*(pointer-to-int + 5)– will jump over _5 * sizeof(int)_ memory positions to find the correct element, while _*(pointer-to-Point + 5)_ will jump over _5 * sizeof(Point)_ memory positions (which is at least 4 times farther for a _Point_ with _double x_ and _y_). Once again we see how types help with the correct interpretation of memory locations, which leads to correct operation with this memory.


