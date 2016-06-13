### IO (4)

The standard way to manipulate IO streams is through the two overloaded operators, extraction (`operator>>`) and insertion (`operator<<`). These operators are binary infix operators. On one side there is always a stream object (input, output, or bidirectional) and on the other any object that has overloaded the operator. Notice that the naming of the IO operators is from the perspective of the non-stream object: it's extracted from an input stream, and it is inserted into an output stream. Also notice that the stream is always **on the left-hand side**.

## Operator overloading (5)

To be able to use `operator>>` and `operator<<`, a class has to overload them. These operators modify the left-hand side, in other words, the calling object. There is a rule in operator overloading which says that operators which modify the calling object should be implemented as members in the class of the calling object. However, the left-hand side of these operators is a stream object, defined in one of the IO stream library files of the Standard Library. Since those cannot be modified (or, at the very least, is extremely bad and dangerous practice), the canonical way to implement them is as non-member functions, usually declared friend in the class of the right-hand object (that is, the one that has to be extracted from a stream or inserted into a stream). 

```c++
friend std::ostream &operator<<(std::ostream &os, const Point &p);

friend std::istream &operator>>(std::istream &is, Point &p);
```

### Templates (4)

Operator overloading in templates is straightforward for operators overloaded as members. Again, those are operator=, operator[], operator->, and those others that modify the calling object of the class where they are overloaded (e.g. the compound assignment operators).

A quick forward-looking note: Since templates require class and method definitions to be in one file, be careful not to confuse inlining with implementing in the same file. Inlining substitutes the method's machine instructions in-line for every call of the function in the code. As a general rule, only lightweight methods should be inlined.

Operator overloading for operators overloaded as non-member functions adds a wrinkle in templates that is worth understanding, and has an interesting side effect that has to do with inlining. Remember the code for the first Point template:

```c++
template <typename T>
class Point {
    // ..
public:
    // ...

    template <typename S>
    friend std::ostream &operator<<(std::ostream &os, const Point<S> &p);
};

template <typename W>
std::ostream &operator<<(std::ostream &os, const Point<W> &p) {
    os << p.getX() << ", " << p.getY();

    return os;
}
```

Why is the friend function using `template <typename S>` rather than using the `T` type parameter of the `Point` class? The `Point` class template and the `operator>>` function template are two different independent templates, one a class template and one a function template. From the point of view of the class template there is some function template outside its scope and it is granting it `friend` privileges. It cannot be templatized with the same parameter `T` of the class because they have different parameters (that is, `T` is not its parameter). To illustrate that, the template parameter name is set to `W`, though there is no further significance of the letter. Syntactically, it's just a name; functionally, it's just a placeholder.

However, there is a dangerous side effect of this implementation. The templatized friend declaration inside the class says more than originally intended: it declares any instantiation of the non-member operator<< to have friend privileges. This means that operator<< for double, int, and any other instantiation will have private access to all Point classes, effectively breaking their encapsulation. For this reason, this implementation is called **extrovert**.

To avoid this promiscuity, `operator<<` has to be _inlined_ as done in our `Point` class:

```c++
template <typename T, int dim>
class Point {
    // ...

public:
    // ...

    friend std::ostream &operator<<(std::ostream &os, const Point &p) {
        int i = 0;
        for ( ; i < dim - 1; i ++) {
            os << p.values[i] << ", ";
        }
        os << p.values[i] << std::endl;

        return os;
    }
};
```

Notice that the `operator<<` is not templated but a non-templated function for every instantiation of the `Point` template. This implementation is equivalent to manually adding an `operator<<` for `Point<int>` and another for `Point<double>`. The `double` instantiation has access only to `Point<double>` and not the `Point<int>`, and vice versa. For this reason, it is called **introvert**.

There is a introvert way to overload without inlining with slightly awkward syntax:

```c++
// forward declare the two templates
template <typename T, int dim> class Point;
template <typename T, int dim> std::ostream &operator<<(std::ostream &os, const Point<T, dim> &p);

// declare the templates
template <typename T, int dim>
class Point {
    // ...

public:
    // ...

    friend std::ostream &operator<< <T>(std::ostream &os, const Point<T, dim> &p);
};

// implement operator<<
template <T, dim>
std::ostream &operator<<(std::ostream &os, const Point<T, dim> &p) {
    int i = 0;
    for ( ; i < dim - 1; i ++) {
        os << p.values[i] << ", ";
    }
    os << p.values[i] << std::endl;

    return os;
}
```

Notice the `<T>` after the `friend` declaration of `operator<<`.  (**Note:** There is a _space_ between `<<` and `<T>`.) This is called a `specialization` of the template for type T. It relies on argument deduction in declarations. Now the operator only has private access to the template class with the same type `T`. See [C++ reference](http://en.cppreference.com/w/cpp/language/friend).

### Templates (5)

Templates are very powerful but they have their cost and limits. Our Point class template already imposes a lot of constraints on the type parameter T:

  1. T must support initialization to 0
  2. T must support copy-construction
  3. T must support assignment

Now let's look at what it additional constraints come from the implementation of the simple method `distanceTo()` for our templatized `Point`:

```c++
#include <cmath>

template <typename T>
class Point {
    T x, y;

public:
    // ...
    
    T distanceTo(const Point<T> &p) const;
};

template <typename K>
K Point<K>::distanceTo(const Point &p) const {
    K d = 0;
    d = (x - p.x) * (x - p.x) + (y - p.y) * (y - p.y);
    return std::sqrt((double) d); // note: sqrt requires a float or a double
}
```

Now, in addition to the above constraints, `T` must support:

  1. Addition, subtraction, and multiplication.
  2. Casting from `T` to `double` and vice versa.

Now, the types we can use for `T` are quite limited. Our templatized `Point` is versatile but it's also limiting. The overloaded operators that need to work with the intended parameter types implicitly limit the use of `Point<>` to only types for which these operators make sense.

Currently, there is no way to specify such template class usage constraints explicitly in the language. They need to go into the documentation. **Note:** There are [experimental syntactic features](http://en.cppreference.com/w/cpp/language/constraints) considered for the upcoming C++17 standard to deal with this shortcoming.

### String class (1)

The Standard Library string class is a powerful tool for manipulation of textual data. But it's also a nice showcase of several major C++ features that we have covered:

  1. The `std::string` class is an instantiation of the class template `std::basic_string`.
  2. `std::string` objects were used as temporary storage for stream chunks. They can also be converted to streams to be used in term by downstream extraction operators. Since the stream IO abstraction is defined as a stream of characters, string objects become naturally indispensable in implementing IO operators. Strings are used as temporary storage of chunks from the stream. 
  3. `operator+` is overloaded for string objects to mean appending. 

```c++
// (1) template
template < class charT,
           class traits = char_traits<charT>,    // basic_string::traits_type
           class Alloc = allocator<charT>        // basic_string::allocator_type
           > class basic_string;

typedef basic_string<char> string;

// (2) IO
std::string s("mystring"), t;
std::cout << s << std::endl; // prints "mystring"
stringstream strm(s);
s >> t;

// (3) 
std:: cout << s + t << std::endl; // prints "mystringmystring"
```

Note that the string is a fully-fledged type and, though compatible and mutually convertible with C-style strings (null-terminated char arrays), it is distinct from them. The Standard Library's section <string> provides many facilities for manipulation of string objects. The section <cstring> provides additional functionality applied to C-style strings, which, by virtue of mutual conversion, can be used along with string objects. In general, you should minimize the mixing of methods from the two libraries and prefer <string>.

### STL (1)

A sizable part of the C++ Standard Library, called the Standard Template Library (STL), supports _generic programming_. Generic programming is a paradigm that seeks to abstract to the maximum possible extent the particulars of direct memory management, switching data representation and structures, and implementing basic algorithms to manipulate data. STL provides:

  1. Class templates to container classes that can hold any of your user-defined types.
  2. Template-class compatible iterators to loop through container structures.
  3. Generic base-type-agnostic algorithms.

We can get an immediate taste of the advantages that STL confers by seeing how our `Point` and `Cluster` classes don't have to implement their own "raw" double array and Point pointer linked-list. For example:

```c++
//
// Created by Ivo Georgiev on 10/24/15.
//

#ifndef TEMTEST_POINTV_H
#define TEMTEST_POINTV_H

#include <vector>

template <typename T>
class PointV {
    unsigned int __dimensions;
    std::vector<T> __values;

public:
    PointV(unsigned int d) :
            __dimensions(d),
            __values(__dimensions)
    {}
    T &operator[](unsigned int i) { return __values[i]; }
    friend std::ostream &operator<<(std::ostream &os, const PointV<T> &p) {
        int i = 0;
        for ( ; i < p.__dimensions - 1; i ++) {
            os << p.__values[i] << ", ";
        }
        os << p.__values[i] << std::endl;

        return os;
    }
    void setValue(unsigned int i, T value) { __values[i] = value; }
    T getValue(unsigned int i) const { return __values[i]; }
};


#endif //TEMTEST_POINTV_H

// main.cpp
#include <iostream>
#include <vector>
#include "PointV.h"

using std::cout;
using std::endl;

int main() {
    const unsigned int dims = 10;
    PointV<double> p(dims);
    for (int i = 0; i < dims; i ++) p[i] = 10 * i + 4.13;

    cout << p << endl;

    return 0;
}
```

### Pointers (9)

Notice that though we don't instantiate the vector and forward_list classes with pointers to the base types, they are allocated on the heap. The STL containers take care of dynamic allocation and deletion of the objects we use store in them.

No pointer manipulation with generic programming using STL!

### Class design (7)

Notice that, templates or not, we don't have to change the interface of a class to reimplement it STL. The method names remain the same.

People who used our previous version don't have to change their code!

