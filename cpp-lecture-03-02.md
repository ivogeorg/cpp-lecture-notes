### Templates (1)

Templates are a C++ feature which allows an even further abstraction of functionality and universalization of code. There are two major types of templates: function templates and class templates.

The canonical way to templatize: from a class with members of fixed types to a template class with parameterized member types. The fixed types are substituted with type parameters. For example, our double-based Point can be transformed to a template class which takes a type (class) parameter for the dimension values of Point. The original Point looks like this:

```c++
#ifndef TEMTEST_POINT_H
#define TEMTEST_POINT_H

class Point {
    double x, y;

public:
    Point() : Point(0, 0) {}
    Point(double a, double b) : x(a), y(b) {}
    double getX() const { return x; }
    double getY() const { return y; }
    void setX(double a) { x = a; }
    void setY(double b) { y = b; }

    friend std::ostream &operator<<(std::ostream &os, const Point &p);

};

std::ostream &operator<<(std::ostream &os, const Point &p) {
    os << p.getX() << ", " << p.getY();

    return os;
}

#endif //TEMTEST_POINT_H
```

The template class looks like this:

```c++
#ifndef TEMTEST_POINT_H
#define TEMTEST_POINT_H

template <typename T>
class Point {
    T x, y;

public:
    Point() : Point(0, 0) {}
    Point(T a, T b) : x(a), y(b) {}
    T getX() const { return x; }
    T getY() const { return y; }
    void setX(T a) { x = a; }
    void setY(T b) { y = b; }

    template <typename S>
    friend std::ostream &operator<<(std::ostream &os, const Point<S> &p);

};

template <typename T>
std::ostream &operator<<(std::ostream &os, const Point<T> &p) {
    os << p.getX() << ", " << p.getY();

    return os;
}


#endif //TEMTEST_POINT_H
```

The type `double` is substituted with a parameter name `T`. **NOTE:** The fact that the class template type parameter name is `T` and the function template type parameter in the `friend operator>> `declaration is `S` is significant. The fact that the class template type parameter name is `T` and so is the function template type parameter in the `operator>>` definition is not significant. The latter can be anything, even `T`, since this is a function template that is different (read, separate and independent) from the `Point` class template. 

Note that all the code is in the header file. With templates, the declaration and definition should be **in one file** or, if in two, both files should be `#include`d. (See a following Template section for an explanation of why that is.)

This code can be used as follows:

```c++
#include <iostream>
#include "Point.h"

using std::cout;
using std::endl;

int main() {
    Point<double> p1;

    p1.setX(2.2);
    p1.setY(3.1);

    cout << p1 << endl;
    return 0;
}
```

Our templatized `Point` can now take different types: `double`, `float`, `int`, etc.

When the compiler encounters an template instantiation (that is, a declaration of an instantiated template type), it does the reverse of what we did above: it substitutes T with double and creates a new compilation unit for the resulting (non-template) code.

A note the terminology:

  1. A class template is a template for a class.
  2. A template class is in instantiation of a class template.
  3. A function template is a template for a function.
  4. A template function is an instantiation of a function template.

Now that we have the ability to use different base types for our `Point`, let's use the template mechanism further to parametrize the dimensionality of the `Point`:

```c++
#ifndef TEMTEST_POINTM_H
#define TEMTEST_POINTM_H

#include <iostream>

template <typename T, int dim>
class Point {
    T values[dim];

public:
    Point() {};
    Point(T *v) { for (int i = 0; i < dim; i ++) values[i] = v[i]; }
    T get(int pos) const { return values[pos]; }
    void set(int pos, T a) { values[pos] = a; }

    friend std::ostream &operator<<(std::ostream &os, const Point<T, dim> &p) {
        int i = 0;
        for ( ; i < dim - 1; i ++) {
            os << p.values[i] << ", ";
        }
        os << p.values[i] << std::endl;

        return os;
    }
};

#endif //TEMTEST_POINTM_H
```

Let's see it in action:

```c++
#include <iostream>
#include "PointM.h"

using std::cout;
using std::endl;

int main() {
    const int ddim = 10;
    Point<double, ddim> p1;

    for (int i = 0; i < ddim; i ++)
        p1.set(i, (i + 1) * (ddim + 1/(20.0) ) );

    cout << p1 << endl;

    const int idim = 5;
    Point<int, idim> p2;

    for (int i = 0; i < ddim; i ++)
        p2.set(i, (i + 1) * ddim );

    cout << p2 << endl;

    return 0;
}

// Output:
// 10.05, 20.1, 30.15, 40.2, 50.25, 60.3, 70.35, 80.4, 90.45, 100.5
//
// 10, 20, 30, 40, 50
```

### Templates (2)

We have seen the syntax for both a class template (`Point`) and a function template (`operator>>`).

```c++
// The following examples are taken from the standard
// class unique_ptr

// function template
template <class U>
void operator()(U* ptr) const;

// class template
template<
    class T,
    class Deleter = std::default_delete<T> // note: default
> class unique_ptr;

template <
    class T,
    class Deleter
> class unique_ptr<T[], Deleter>;  // note: array specialization
```

A template can have an unrestricted number of parameters, which form the parameter list. Each parameter can be of the following:

  1. A non-type template parameter. (Remember, _type_ is approximately equivalent to _class_ in C++ parlance.) For example, the `int dim` in our second `Point` template.
  2. A type template parameter. This is the prevalent case. For example, `typename T` above.
  3. A template template parameter. For example, `template <template <typename T> class S> class Cluster {};`.

### Templates (3)

Templates are treated by the compiler differently from regular (non-template) code. If there is no instantiation of a template (like Point<double> p;) in the rest of the program code, the compiler only checks the template code for basic syntax. It checks nothing else (e.g. existence of default constructor, support for operator+, accessible copy constructor for return-by-value, etc).

The compiler only performs a full check upon encountering an instantiation. For each instantiation it encounters, it performs a separate pass. Collectively, this is known as the second pass (aka second phase) of compilation.

Because the compiler effectively generates a new compilation unit for each template instantiation, it requires the full code to be in one file (generally the header). Remember that the compiler works on one compilation unit at a time. If the "implementation" of a class template methods were in a different file, the compiler would be unable to create a full instantiation and perform a check. The methods would remain unimplemented and the linker would generate a lot of (usually very cryptic) error messages.

 
