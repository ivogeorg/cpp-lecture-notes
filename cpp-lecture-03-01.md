## Overview
Module 3 consists of 3 weeks and 6 lectures. It plunges into function and class templates. It looks at three Standard Library classes - vector, string, iostream - and shows how to make use of them in object-oriented programming. It reviews class design by using them as examples for operator overloading. It introduces exception handling.

### Input & output (IO) (1)

Today, data dominates computing. The paradigm has shifted from algorithm-centric to data-centric. The most important operations in the new data-driven processing paradigm is input (getting data into objects) and output (getting data out of objects). For us, this means being able to load our points into Point objects and Cluster objects from, say, a comma-separated values (CSV) file. Here is a first glimpse of how to we might do that: If the file holding the point data looks like this:

```
00002.3,5.6,0,5.6,7.9
1.3, 4.3, 0, 5.6, 7.9
2.4  ,  5.6   ,   0,  6.6  ,  7.1
4.1,5.6,5,1.6,7.9
```

We can use this code to read it in and load it into Point objects:

```c++
#include <iostream>
#include <fstream>
#include <sstream>

#include "Point.h"

// iostream
using std::cout;
using std::endl;

// fstream
using std::ifstream;

// sstream
using std::stringstream;
using std::string;

using Clustering::Point;

int main() {
    ifstream csv("points.csv");
    std::string line;
 
    if (csv.is_open()) {

        while (getline(csv,line)) {

            cout << "Line: " << line << endl;

            stringstream lineStream(line);
            string value;
            double d;
            Point p(5);

            int i = 1;
            while (getline(lineStream, value, ',')) {
                d = stod(value);
    
                cout << "Value: " << d << endl;

                p.setValue(i++, d);
            }
            cout << "Point: " << p << endl;
        }
    }
    csv.close();
    
    return 0;
}
```

The output will look as follows:

```
Line: 00002.3,5.6,0,5.6,7.9
Value: 2.3
Value: 5.6
Value: 0
Value: 5.6
Value: 7.9
Point: 2.3, 5.6, 0, 5.6, 7.9
Line: 1.3, 4.3, 0, 5.6, 7.9
Value: 1.3
Value: 4.3
Value: 0
Value: 5.6
Value: 7.9
Point: 1.3, 4.3, 0, 5.6, 7.9
Line: 2.4  ,  5.6   ,   0,  6.6  ,  7.1
Value: 2.4
Value: 5.6
Value: 0
Value: 6.6
Value: 7.1
Point: 2.4, 5.6, 0, 6.6, 7.1
Line: 4.1,5.6,5,1.6,7.9
Value: 4.1
Value: 5.6
Value: 5
Value: 1.6
Value: 7.9
Point: 4.1, 5.6, 5, 1.6, 7.9
```

We initialize a file input stream by passing the C-string (char *) name of a local text file to the ifstream constructor. Then we proceed to read from the stream, convert the data points we find there, and use the data to set the values of Point objects. Notice that the code is robust enough to handle minor variations in the input (e.g. leading zeros in double values, irregular white space between values, etc).

### IO (2)
The C++ Standard Library offers streams as an input-output abstraction. All data sources and sinks (i.e. data consumers) are treated as (potentially infinite) "streams" of characters. Thus, input-output mechanisms conforming to this abstraction can work with any data source and data consumer. See the overview in the [C++ Reference](http://www.cplusplus.com/reference/iolibrary/) to learn how the abstraction is defined and implemented.

The stream library is organized as a hierarchy of classes, starting with abstract (partially implemented or completely unimplemented classes) at the root and finishing with concrete implementations for different types of input-output data at the leaves (left-to-right in the Reference diagram). There are three standard implementations:

  1. For console IO: `std::cout`, `std::cin`, `std::cerr`.
  2. For C++ string IO.
  3. For file IO.

We have seen all of them. We routinely use the console streams to read in user input and print out program results and messages. We just used the file stream to read a text file with a particular format and write a text file with a particular (different) format. If you look closely, we also use a string stream (literally, `std::stringstream`) to convert a portion of the input stream read into a string variable back into an input stream for further processing.

The only special part of the stream syntax are the extraction (`operator>>`) and insertion (`operator<<`) operators, which are overloaded bitshift operators. These are binary operators, taking a stream on one side and an object of some other type on the other. The type of the object has to have the two operators overloaded.

### IO (3)

The most natural way to read from a stream is character-by-character. Standard library and user-defined functions can read a stream chunk-by-chunk. One such function is `std::getline`. Chunks are broken off upon "delimiter" character in the stream. The default delimiter is the `\n` (newline) character. A non-default delimiter can be specified as the third argument for `getline`.

Now we can take the example code from above and dividing the input work among the `Cluster` and `Point` extraction operators (`operator>>`). Notice the code sections that go into each function implementation; they form a multi-level out-in-out hierarchy:

```c++
#include <iostream>
#include <fstream>
#include <sstream>

#include "Point.h"

// iostream
using std::cout;
using std::endl;

// fstream
using std::ifstream;

// sstream
using std::stringstream;
using std::string;

using Clustering::Point;

int main() {
    ifstream csv("points.csv");
    std::string line;

    if (csv.is_open()) {

// << above here is main
// >> below here is Cluster::operator>>

        while (getline(csv,line)) {

            cout << "Line: " << line << endl;

            stringstream lineStream(line);
            string value;
            double d;
            Point p(5); // lineStream >> p;

// >> above here is Cluster::operator>>
// >> below here is Point::operator>>
            int i = 0;
            while (getline(lineStream, value, ',')) {
                d = stod(value);

                cout << "Value: " << d << endl;

                p.setValue(i++, d);
            }
// >> above here is Point::operator>>
// >> below here is Cluster::operator>>
            cout << "Point: " << p << endl;
        }
    }

// >> above here is Cluster::operator>>
// << below here is main

    csv.close();

    return 0;
}
```

The two extraction operators are shown below along with a minimal test function to show the highest level of the nested input-output hierarchy:

```c++
// test.cpp
void test_cluster_load_file(void) {
    cout << endl << "test_cluster_load_file: Testing loading cluster points from file" << endl << endl;

    std::ifstream csv("points.csv");
    if (csv.is_open()) {
        const bool RELEASE_POINTS = true;
        Cluster c(5, RELEASE_POINTS);

        // call to Cluster::operator>>
        csv >> c;
        
        std::cout << "c: " << std::endl << c << std::endl;
        csv.close();
    }

    cout << endl << "test_cluster_load_file: End" << endl << endl;
}

// Cluster.cpp
std::istream &operator>>(std::istream &istream, Cluster &cluster) {
    std::string line;
    while (getline(istream,line)) {
        int d = (int) std::count(line.begin(),
                                 line.end(),
                                 Point::POINT_VALUE_DELIM);
        PointPtr ptr = new Point(d + 1);
        std::stringstream lineStream(line);

        // call to Point::operator>>
        lineStream >> *ptr;

        cluster.add(ptr);
    }

    return istream;
}

// Point.cpp
std::ostream &operator<<(std::ostream &ostream, const Point &point) {
    int i = 0;
    for ( ; i < point.dim - 1; i ++)
        ostream << point.values[i]
        << Point::POINT_VALUE_DELIM << ' ';
    ostream << point.values[i];

    return ostream;
}
```
