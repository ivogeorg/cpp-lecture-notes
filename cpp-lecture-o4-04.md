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

Our internal clustering criterion BetaCV is very computationally expensive: at every iteration we have to recompute the distances between every two points to calculate it. We can reducing this computational expense by caching the distances. To do we will precompute them and store them in a hash table. A hash table (aka associative array, or map) stores pairs of the form {key, value} where key and value can be of primitive or complex types, either them same or different from each other.

The hash table's primary advantage is that it has constant O(1) operations: insertion, search, retrieval. It achieves that by using a hash function which maps the key to a natural number (i.e. a positive integer). This number is used as an address of one (of many) containers (called buckets) where the pair will be stored. If more than one pair's key evaluates to the same number, more than one pair will be stored in the same bucket. This is called a collision. Buckets are usually implemented as linked lists of the colliding pairs. So, the more collisions, the longer the bucket lists, the more the performance of the hash table is degraded.

We want our key to contain two points and use their unique id-s to calculate the hash function. For this, we can use a pairing function of the form

![pairing-function](https://cloud.githubusercontent.com/assets/6043344/16057932/080559a8-3231-11e6-8f8c-d3be60c82441.png)
