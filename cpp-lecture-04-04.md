### Inheritance (11)

Multiple inheritance

### Inheritance (12)

Virtual inheritance

### Templates (7) 

Template specializations.

### Templates (8)

Templates and inheritance: orthogonal & complementary

No `virtual` functions in templates.

Type erasure?

### Class design (7)

Our internal clustering criterion BetaCV is very computationally expensive: at every iteration we have to recompute the distances between every two points to calculate it. We can reducing this computational expense by caching the distances. To do we will precompute them and store them in a hash table. A hash table (aka associative array, or map) stores pairs of the form _{key, value}_ where key and value can be of primitive or complex types, either them same or different from each other.

The hash table's primary advantage is that it has constant O(1) operations: insertion, search, retrieval. It achieves that by using a hash function which maps the key to a natural number (i.e. a positive integer). This number is used as an address of one (of many) containers (called buckets) where the pair will be stored. If more than one pair's key evaluates to the same number, more than one pair will be stored in the same bucket. This is called a collision. Buckets are usually implemented as linked lists of the colliding pairs. So, the more collisions, the longer the bucket lists, the more the performance of the hash table is degraded.

We want our key to contain two points and use their unique id-s to calculate the hash function. For this, we can use a pairing function of the form

![pairing-function](https://cloud.githubusercontent.com/assets/6043344/16057932/080559a8-3231-11e6-8f8c-d3be60c82441.png)

An easy and frequently used function is the _Cantor function_ 

![cantor-function](https://cloud.githubusercontent.com/assets/6043344/16058061/8115cd0a-3231-11e6-909f-6d85ad98c6d7.png)

It maps any two positive integers to a unique positive integer.

Notice that to avoid storing every distance twice, our function has to meet the transitivity condition

![transitivity](https://cloud.githubusercontent.com/assets/6043344/16058120/b5524918-3231-11e6-9c55-e8771c66ac2b.png)

The Cantor function does not have this property, but we can achieve this by:

  1. sorting the two id's in ascending order before we apply the Cantor function function, and
  2. extending the key equality to meet the transitivity condition, and
  3. making sure we calculate only one of the distances.

### STL (4)

We will use the STL class `std::unordered_map` as the hash table to store our distances. For that, we need to define three types:

   1. A structure that represents the key type (here, `DPKey`);
   2. A functor that implements the hashing for the new key type (here, `DPKeyHash`);
   3. A functor that implements the equality for the new key type (here, `DPKeyEqual`). 

`Point.h`:

```c++
//
// Created by Ivo Georgiev on 11/12/15.
//

#ifndef HASHTEST_POINT_H
#define HASHTEST_POINT_H

#include <iostream>

namespace Clustering {

    class Point {
        int __dim;        // number of dimensions of the point
        double *values; // values of the point's dimensions

    public:
        static const char POINT_VALUE_DELIM;

        Point(int d) : Point(d, 0) { }
        Point(int d, double v);
//        Point(int, double []); // useful for testing but problematic

        // Big three: cpy ctor, overloaded operator=, dtor
        Point(const Point &);
        Point &operator=(const Point &);
        virtual ~Point();

        // Accessors & mutators
        int getDims() const { return __dim; }
        void setValue(int, double);
        double getValue(int) const;

        // Functions
        double distanceTo(const Point &) const;

        // Overloaded operators

        // Members
        Point &operator*=(double);
        Point &operator/=(double);
        const Point operator*(double) const; // prevent (p1 * 2) = p2;
        const Point operator/(double) const;

        double &operator[](int index) { return values[index]; } // TODO out-of-bds

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

        virtual void print(std::ostream &os) const;
//        void print(std::ostream &os) const;
    };


}


#endif //HASHTEST_POINT_H
```

`Point.cpp`:

```c++
//
// Created by Ivo Georgiev on 11/12/15.
//

#include <iostream>
#include <cmath>
#include "Point.h"

namespace Clustering {

    const char Point::POINT_VALUE_DELIM = ',';

    Point::Point(int dimensions, double value) {
        if (dimensions == 0)
            dimensions = 2;                 // TODO throw exception
        __dim = dimensions;
        values = new double[__dim];
        for (int i = 0; i < __dim; i ++)
            values[i] = value;
    }

    Point::Point(const Point &rhs) {        // TODO throw exception
        __dim = rhs.__dim;
        values = new double[__dim];
        for (int i = 0; i < __dim; i++)
            values[i] = rhs.values[i];
    }

    Point &Point::operator=(const Point &rhs) { // TODO throw exception
        if (this == &rhs) {
            return *this;
        } else {
            delete[] values;

            __dim = rhs.__dim;
            values = new double[__dim];
            for (int i = 0; i < __dim; i++)
                values[i] = rhs.values[i]; // HERE
        }
        return *this;
    }

    Point::~Point() {
        delete[] values;
        std::cout << "(~Point)" << std::endl;
    }

// NOTE dimensions naturally start at 1, not 0
    void Point::setValue(int dimension, double value) { // TODO throw exception
        if (dimension >= 0 && dimension < __dim)
            values[dimension] = value;
    }

    double Point::getValue(int dimension) const { // TODO throw exception
        if (dimension >= 0 && dimension < __dim)
            return values[dimension];
        return 0;
    }

    double Point::distanceTo(const Point &other) const { // TODO throw exception
        if (other.__dim == __dim) {
            double sum = 0;
            for (int i = 0; i < __dim; i++) {
                double diff = values[i] - other.values[i];
                sum += diff * diff;
            }
            return sqrt(sum);
        }
        return 0;
    }

    Point &Point::operator*=(double d) {
        for (int i = 0; i < __dim; i++)
            values[i] *= d;
        return *this;
    }

    Point &Point::operator/=(double d) { // TODO handle div-by-0 or let runtime
        for (int i = 0; i < __dim; i++)
            values[i] /= d;
        return *this;
    }

    const Point Point::operator*(double d) const { // NOTE the use of copy ctor
        return Point(*this) *= d;
    } // NOTE destructor for local object is called here

    const Point Point::operator/(double d) const {
        return Point(*this) /= d;
    }

    // p1 += p1 is allowed, but different dimensions throws an exception
    Point &operator+=(Point &lhs, const Point &rhs) { // TODO exception
        if (&lhs == &rhs) {
            return lhs *= 2;
        } else if (lhs.__dim == rhs.__dim) {
            for (int i = 0; i < lhs.__dim; i++)
                lhs.values[i] += rhs.values[i];
        }
        return lhs;
    }

    Point &operator-=(Point &lhs, const Point &rhs) { // TODO exception
        if (&lhs == &rhs) {
            return lhs *= 2;
        } else if (lhs.__dim == rhs.__dim) {
            for (int i = 0; i < lhs.__dim; i++)
                lhs.values[i] -= rhs.values[i];
        }
        return lhs;
    }

    const Point operator+(const Point &lhs, const Point &rhs) {
        Point p(lhs);
        return p += rhs;
    }

    // NOTE
    // returning Point(lhs) += rhs; doesn't work since Point(rhs) cannot
    // be an lvalue and operator+= expects an lvalue as its 1st argument


    const Point operator-(const Point &lhs, const Point &rhs) {
        Point p(lhs);
        return p -= rhs;
    }

    bool operator==(const Point &one, const Point &another) { // TODO exception
        if (one.__dim != another.__dim) return false;

        bool isEqual = true;
        for (int i = 0; i < one.__dim; i++) {
            if (one.values[i] != another.values[i]) {
                isEqual = false;
                break;
            }
        }
        return isEqual;
    }

    bool operator!=(const Point &one, const Point &another) {
        return !(one == another);
    }

    // lexicographic order from left to right - arbitrary, but good practice
    bool operator<(const Point &one, const Point &another) { // TODO exception
        bool isLess = false;
        for (int i = 0; i < one.__dim; i++) {
            if (one.values[i] < another.values[i]) {
                isLess = true;
                break;
            }
        }
        return isLess;
    }

    bool operator>=(const Point &one, const Point &another) { // TODO exception
        return !(one<another);
    }

    bool operator<=(const Point &one, const Point &another) { // TODO exception
        return (one<another || one==another);   // TODO expensive worst case
    }

    bool operator>(const Point &one, const Point &another) { // TODO exception
        return !(one<=another);
    }

    std::ostream &operator<<(std::ostream &ostream, const Point &point) {
        point.print(ostream);
        return ostream;
    }

    // TODO exception on dimensionality mismatch
    std::istream &operator>>(std::istream &istream, Point &point) {
        std::string value;

        int i = 1;
        while (getline(istream, value, Point::POINT_VALUE_DELIM)) {
            double d = stod(value);
            point.setValue(i++, d);
        }
        return istream;
    }

    void Point::print(std::ostream &os) const {
        int i = 0;
        for (; i < __dim - 1; i++)
            os << values[i]
            << Point::POINT_VALUE_DELIM << ' ';
        os << values[i];
    }
}
```

`DataPoint.h` (implements the three new structures):

```c++
//
// Created by Ivo Georgiev on 11/12/15.
//

#ifndef HASHTEST_DATAPOINT_H
#define HASHTEST_DATAPOINT_H


#include "Point.h"

namespace Clustering {
    class DataPoint : public Point {
        unsigned int __id;
        // __dim inheritted but inaccessible
        // __values inheritted but inaccessible
        static unsigned int __idGenerator;
    public:
        DataPoint(int d) :
                Point(d),
                __id(__idGenerator ++)
        {}

        ~DataPoint();

        unsigned int getId() const { return __id; }

        void print(std::ostream &os) const override;

        static void rewindIdGen() { __idGenerator --; }
    };


    // Types needed to supply to std::unordered_map to store distances between points

    struct DPKey { // key {p1, p2}
        DataPoint p1, p2;
        DPKey(const DataPoint &dp1, const DataPoint &dp2) :
                p1(dp1), p2(dp2)
        {}
    };

    struct DPKeyHash { // hash functor
        std::size_t operator()(const DPKey &key) const { // note the const
            unsigned int
                    u1 = key.p1.getId(),
                    u2 = key.p2.getId();

            if (u1 > u2) std::swap(u1, u2); // ascending order

            return std::hash<std::size_t>()((u1 + u2) * (u1 + u2 + 1) / 2 + u2);
        }
    };

    struct DPKeyEqual { // equality functor (implements transitivity)
        bool operator()(const DPKey &lhs, const DPKey &rhs) const { // note the const
            return (lhs.p1.getId() == rhs.p1.getId() && lhs.p2.getId() == rhs.p2.getId()) ||
                   (lhs.p1.getId() == rhs.p2.getId() && lhs.p2.getId() == rhs.p1.getId());
        }
    };
}


#endif //HASHTEST_DATAPOINT_H
```

`DataPoint.cpp`:

```c++
//
// Created by Ivo Georgiev on 11/12/15.
//

#include "DataPoint.h"

namespace Clustering {
    unsigned int DataPoint::__idGenerator = 0;

    void DataPoint::print(std::ostream &os) const {
        os << __id << " : " ;
        Point::print(os);
    }

    DataPoint::~DataPoint() {
        std::cout << "(~DataPoint)" << std::endl;
        // ~Point() called automatically before exit
    }
}
```

`main.cpp`:

```c++
#include <iostream>
#include <vector>
#include <unordered_map>

#include "DataPoint.h"

using namespace std;
using namespace Clustering;

int main() {
    const unsigned int DIM = 10;

    // Create a vector of data points
    cout << endl << "Create vector of points" << endl;
    std::vector<DataPoint> data;
    for (int i = 0; i < 10; i ++) {
        DataPoint dp = DataPoint(DIM);
        for (int j = 0; j < DIM; j ++) dp[j] = 13.5 * i + 5.14 * j;
        data.push_back(dp);
    }

    // Let's see them
    cout << endl <<  "Print out vector" << endl;
    for (auto it = data.begin(); it != data.end(); ++ it)
        cout << *it << endl;

    // Create an unordered map storing the distances between points
    cout << endl <<  "Create unordered map of distances" << endl;
    std::unordered_map<DPKey, double, DPKeyHash, DPKeyEqual> distances;
    for (auto ito = data.begin(); ito != data.end(); ++ ito) {
        auto iti = ito;
        ++ iti; // Start the inner iterator at the next value to avoid repeating points
        for (; iti != data.end(); ++ iti) {
//            cout << *ito << ", " << *iti << endl; // Uncomment to see the points...
//            cout << "DISTANCE: " << ito->distanceTo(*iti) << endl; // ...and the distance
            DPKey key(*ito, *iti);
            auto search = distances.find(key); // check if it is there already (won't be :))
            if (search == distances.end())
                distances[key] = ito->distanceTo(*iti);
        }
    }

    // Let's see the table
    cout << endl <<  "Print out map" << endl;
    for (auto it = distances.begin(); it != distances.end(); ++ it)
        cout
        << "{{"
        << it->first.p1.getId() // Note that the entries of the map are of type std::pair
        << ", "
        << it->first.p2.getId()
        << "}, "
        << it->second
        << "}"
        << endl;

    // Destructors called
    cout << endl <<  "Going out of scope" << endl;
    return 0;
}
```

Notice that what we really need are the two id-s and there is no need to store the two `Point` objects in the key. A faster and more lightweight version can use only the Point id-s for the key.
