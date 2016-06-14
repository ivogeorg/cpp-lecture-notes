### Inheritance (4)

An important concern when using inheritance is to get all the access rights straight. A derived class inherits all data members and most member methods of the base class. But it cannot refer to private members of the base class (even though it has inherited them) in the implementation of its own methods. Why?

**//TODO - CODE... (Point, Centroid)**

This would break encapsulation and the private access specifier for the members of a class would be trivially defeated by just subclassing from it. The derived class has to use the appropriate accessors and mutators to access its inherited private members.

Not inherited:  Note that this makes private functions essentially unusable to derived classes. Indeed, private methods are considered helper methods. If a method is not just a helper method, it should be made public.

Not inherited: Friends "are not inherited". Friends are, by definition, not members, so they can't be inherited. So, it's more accurate to say that friendship is not inherited.

Inheritance (5)
To give access to private members to derived classes, C++ provides another access specifier, `protected`. `protected` data members remain private to all but derived classes.

**//TODO - CODE... (Point, Centroid)**

### Inheritance (6)

how does a derived class "redefine" base class behavior?

polymorphism, late binding, delayed dispatch

`virtual` keyword

`virtual` functions `virtual int foo() const;`

automatically `virtual` in derived classes (i.e. `virtual` property is inherited); keyword can be omitted, but it's good to keep it

overloading vs overriding vs redefining (don't do the last)!

`override`, as in `keyword int foo() const override;`

preventing (further) overriding of a `virtual` function, as in `virtual int foo() const final;`

pure virtual functions, as in `virtual int foo() const = 0;`

abstract classes - classes that are not fully implemented

### STL (3)

uses of abstract classes 
