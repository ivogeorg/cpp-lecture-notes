### Namespaces (1)

Namespaces are a scoping feature of the C++ language. Classes (types) can be declared in a special named block (scope). The syntax of namespaces is very straightforward:

```c++
// to declare
namespace [name_of_namespace] {

    class Point {};
    
    class Cluster {};
    
    class KMeans {};

}

// to use
using namespace [name_of_namespace];
// or
using [name_of_namespace]::Point;
using [name_of_namespace]::Cluster;
// or
int main() {
    [name_of_namespace]::KMeans kmeans(10, 5);
    
    return 0;
}
```

Namespace provide additional isolation of name sets of types and related methods, minimizing potential ambiguities in a build environment. For example, the following two Point types are distinct and can both be used in the same codebase:

```c++
namespace Clustering {
    
    class Point {};
    
}

namespace Demographics {
    
    class Point {};
    
}

int main() {
    Clustering::Point cpt;
    Demographics::Point dpt;
    
    // ...
    
    return 0;
}
```

If you have printed anything to the console in C++, you have already encountered namespaces:

```c++
#include <iostream>

using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    
    return 0;
}
```

The object **std::cout** is an output stream that usually outputs to the console. Library namespaces, where **std::endl** is an output-only I/O manipulator. Both are defined in the C++ Standard Library and so exist in the std namespace. We have used the namespace-wide directive using namespace std;  to be able to use these objects. This directive allows any types and objects from the std namespace to be used in the subsequent code, however this is poor style. The Standard Library is big and so is the namespace std. It is much more sparing and clear to use a per-class directive (`using std::cout;`) or even a name qualifier upon usage (`std::cout << p << std:: endl;`). 

### Namespaces (2)

The scope resolution operator **::** is used to specify a namespace in qualified class names (as in `Clutering::Point` or `std::string`). This is the same operator we use to specify implementations of class member methods (as in `Point::Point()` and `Cluster::operator=(const Cluster &)`).

When we want to implement methods of a class that is declared within a namespace, we should wrap the source code in a namespace block `{}`, as follows:

```c++
#include "Cluster.h"

namespace Clustering {
    
    // declaration and initialization of a static variable
    unsigned Cluster::__idGen = 0;
    
    Cluster::Cluster() {
        // implementation of default constructor
    }
    
    Cluster &Cluster::Cluster(const Cluster &clust) {
        // implementation of copy constructor
        return *this;
    }

    friend std::istream &operator>>(std::istream &is, Cluster &clust) {
        // implementation of the extraction operator
        
        // Notice: No Cluster:: qualifier since this is NOT A MEMBER function
        
        return is;
    }

    // other method implementations

}
```

The scope resolution operator allows us to gain visibility of the types and methods that are declared within a namespace. Otherwise, they are invisible to us outside unless our code is being written inside the namespace.

Namespaces can be nested, as follows:

```c++
namespace A {
    
    namespace B {
        
        namespace C {
            
            class Point {};
            
        }
        
    }
    
}
```

In this case, the scope resolution operator is applied multiple times in a row (as in `A::B::C::Point p(13);`).

### Namespaces (3)

There are two special namespaces, the anonymous namespace and the global namespace.

The anonymous namespace is declared simply without a name, and the global namespace is the default namespace for any object, class or method which has not been declared in another namespace:

```c++
namespace {
    
    unsigned id;
    
    class Helper {};

    void doService(Helpter &hlp) {}

}

// in global namespace
const unsigned GlobalMagicNumber = 1221;


int main() {
    // NOTICE:
    // Everyting declared in an anonymous namespace
    // is visible within the translation unit (file).
    // For variables and methods that are in the 
    // anonmous namespace but outside of class
    // declarations, this is equivalent to declaring
    // them 'static'.
    Helper hlp(std::cout, 
               "ivo.georgiev@ucdenver.edu",
               ::GlobalMagicNumber);
    // Also notice that to refer to objects in the
    // global namespace, we use a leading scope
    // resolution operator (::).
               
    return 0;
}
```

The anonymous namespace is useful to hide infrastructure code, helper functions, file-local global variables, etc.

### Class design (6)

We are using a linked list to hold the `Point` objects in a `Cluster`. All it takes to declare a linked list is a node structure and a single pointer to a node. The node structure holds an object of the base data type we want to collect with the linked list, and a link to the next node (or, links to both the next and previous nodes):

```c++
namespace Clustering {

    typedef struct LNode *LNodePtr;

    struct LNode {

        Point point;
        LNodePtr next;

        LNode(const Point &p, LNodePtr n);

    };

    class Cluster {
        
        int __size;
        LNodePtr __points;
        
    };
    
}
```

The linked list is one of the most popular data structures in programming. It is very flexible and prolific. The list consists of nodes that are linked to each other by pointers. The list is implicitly ordered but, compared to arrays/vectors allows easy (and cheap) insertions and deletions at any position in the list. Each node is allocated dynamically and separately so all the nodes are on the heap.

The list above is a singly-linked list and can only be traversed in the forward direction. A doubly linked list equivalent is shown here:

```c++
namespace Clustering {

    typedef struct LNode *LNodePtr;

    struct LNode {

        Point point;
        LNodePtr next, prev;

        LNode(const Point &p, LNodePtr n);

    };

    class Cluster {
        
        int __size;
        LNodePtr __points;

    };
    
}
```

The doubly linked list is easier to manipulate, but has twice the overhead of a singly linked list per data unit.

It is helpful to employ the following two techniques in the implementation of the singly linked-list manipulation methods:

  1. Handle separately and in this order the cases of:
    1. Empty list
    2. The first element of the list
    3. All the rest of the elements
  2. When traversing the list keep track of two variables
    1. A pointer to the current element (e.g. `curr`)
    2. A poitner to the previous element (e.g. `prev`)

### Overloading operators (4)

The signatures of overloaded operator declarations, when the standard/conventional C++ operator semantics are followed, give a lot of information about what and how should be implemented.

For example, the signature

```c++
friend Point &operator+=(Point &lhs, const Point &rhs); // p1 += p2;
```

tells us that the the left-hand side argument, which is the object appearing on the left of the binary operator, will be modified, and the right-hand side one will not. The return type is a reference to an object, which, combined with the knowledge of how the operator is used (i.g. its standard semantics), tells us that we will return a reference to the `lhs` object.

In contrast, the signature

```c++
friend const Point operator+(const Point &lhs, const Point &rhs); // p1 = p2 + p3;
```

tells us that, since neither of the arguments can be changed but a new Point object is returned by value, the this function will create a new object and return a copy of it (return by value).

### References (1)

References are aliases for objects.

Like pointers:

  * they stand for objects
  * they are assigned the addresses of already existing local objects
  * can allow the manipulation of large objects without the overhead of copying them

Unlike pointers:

  * they have static/local semantics (e.g. ref.dataMember vs ptr->dataMember)
  * they have to be initialized when created
  * cannot be reassigned (that is, they are implicitly const)

References are less powerful than pointers, but safer. They can be though of a safer equivalent of pointers with a simplified and more elegant syntax. As we have seen, they are preferred as function arguments and return values.

To learn more about C++ references, hear it [from the horse's mouth](https://isocpp.org/wiki/faq/references).
