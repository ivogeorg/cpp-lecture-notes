## Overview

Module 1 consists of 2 weeks and 4 lectures. It does a review of the C++ and programming material which is considered prerequisite for this course. It examines C++ data types, primitive and aggregate, the basics of computer architecture and organization, the basics of the build process, and the basics of programmatic memory management.

### Computing (1)

What is computing? The manipulation and processing of data.

**//TODO**

### Data (1)

How is data stored on the computer? Data comes in two major varieties: primitive data types and aggregate data types.

Primitive types represent the most basic types of information the computer can represent and store directly in its smallest memory locations. These are integers, floating-point reals, double precision floating-point reals, characters, and booleans (bits). Thus, primitive data types can directly represent numbers, text, and logical values.

Aggregate data types are composed of primitive data types. There are two varieties: arrays and structures/classes. Arrays are sequential collections of a single primitive or aggregate data type, stored contiguously in memory. Structures/classes are arbitrary collections of primitive and aggregate data types, along with functions that can be applied on the structure/class. The data and functions are called members of the structure/class.

### Data (2)

Integers.

Integers are represented directly within the range of the data type.

Integers come in several different sizes: char (the C programming language treats characters as 8-bit integers), short integer, regular (default) integer, long integer. Since these sizes can vary between architectures, it is best to use the sizeof operator to determine their sizes:

```c++
#include <iostream>

using namespace std;

int main() {
    cout << "Hello, World!" << endl;

    cout << "The size in bytes of 'bool' is " << sizeof(bool) << endl;
    cout << "The size in bytes of 'char' is " << sizeof(char) << endl;
    cout << "The size in bytes of 'short' is " << sizeof(short) << endl;
    cout << "The size in bytes of 'int' is " << sizeof(int) << endl;
    cout << endl;
    cout << "The size in bytes of 'unsigned (int)' is " << sizeof(unsigned) << endl;
    cout << endl;
    cout << "The size in bytes of 'long' is " << sizeof(long) << endl;
    cout << "The size in bytes of 'float' is " << sizeof(float) << endl;
    cout << "The size in bytes of 'double' is " << sizeof(double) << endl;
    cout << endl;
    cout << "The size in bytes of 'int *' (pointer to int) is " << sizeof(int*) << endl;
    cout << "The size in bytes of 'double *' (pointer to double) is " << sizeof(double*) << endl;

    return 0;
}
```

On a modern Macbook the output is as follows:

```
/Users/ivogeorg/Library/Caches/CLion12/cmake/generated/ef94e2be/ef94e2be/Debug/sizetest
Hello, World!
The size in bytes of 'bool' is 1
The size in bytes of 'char' is 1
The size in bytes of 'short' is 2
The size in bytes of 'int' is 4

The size in bytes of 'unsigned (int)' is 4

The size in bytes of 'long' is 8
The size in bytes of 'float' is 4
The size in bytes of 'double' is 8

The size in bytes of 'int*' (pointer to int) is 8
The size in bytes of 'double*' (pointer to int) is 8

Process finished with exit code 0
```
### Data (3)

Signed vs unsigned.

Integers come in two varieties: signed (default) and unsigned. The same size variables (that is, char, short, int, long) is used for both signed and unsigned, so any bit pattern of 1-s and 0-s has a valid interpretation as a signed and unsigned integer. Since signed integers are the default, unsigned integers have to be qualified with the modifier keyword unsigned:

```
char c;
short s;
int i;
long l;
unsigned char uc;
unsigned short us;
unsigned int ui1;
unsigned ui2;  // notice that 'unsigned' by itself defaults to 'unsigned int'
unsigned long ul;
```

### Data (4)

Integer arithmetic.

Integer arithmetic is exact unless there is overflow or underflow, that is, the datum grows too large in absolute value for the corresponding memory unit to represent. Also, integer arithmetic discards fractional parts. For example, 5/3 == 1, not 1.(6).

### Data (5)

Real numbers.

Reals are approximated. Their precision is no only finite, but can also vary between numbers. Reals are represented by floating point data types. There are several representations of real numbers: fixed-point, floating-point, binary-coded-decimal, etc. They differ in format, precision, and speed of operations. We focus on floating-point numbers.

Their format (specifically the IEEE 754 single-precision) looks as follows:

