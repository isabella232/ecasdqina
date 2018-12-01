---
layout: lib
title: 基本
permalink: graph/general

---


```cpp
/// --- Graph Template {{"{{"}}{ ///

template < class T >
struct Edge {
  int from, to;
  T cost;
  Edge(int to, T cost) : from(-1), to(to), cost(cost) {}
  Edge(int from, int to, T cost) : from(from), to(to), cost(cost) {}
};
template < class T >
using WeightedGraph = vector< vector< Edge< T > > >;
using UnWeightedGraph = vector< vector< int > >;

/// }}}--- ///
```

