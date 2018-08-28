---
layout: lib
title: Binary Indexed Tree (Fenwick Tree)
permalink: data-structure/misc/BIT

---

getは自分で配列を用意することで $O(log N)$ を $O(1)$ にできる．


```cpp
// NOTE : there's get and sum method.
// NOTE : access i < n only
/// --- BIT Library {{"{{"}}{ ///

template < class T = ll >
struct BIT {
  vector< T > data;
  size_t n;
  T identity;
  function< T(const T &, const T &) > merge;
  BIT() : n(0) {}
  BIT(size_t n, T identity = T(),
      function< T(const T &, const T &) > merge =
          [](T const &a, T const &b) { return a + b; })
      : n(n), identity(identity), merge(merge) {
    data.resize(n, identity);
  }
  void add(int i, T x) {
    i++;
    while(i <= n) {
      data[i - 1] = merge(data[i - 1], x);
      i += i & -i;
    }
  }
  T sum(int i) {
    if(i < 0) return 0;
    i++;
    T s = identity;
    while(i > 0) {
      s = merge(s, data[i - 1]);
      i -= i & -i;
    }
    return s;
  }
  T get(int i, function< T(const T &) > const &inverse = [](T const &a) {
    return -a;
  }) {
    return merge(sum(i), inv(sum(i - 1)));
  }
  T range(int a, int b,
          function< T(const T &) > const &inverse = [](T const &a) {
            return -a;
          }) {
    return merge(sum(b), inv(sum(a - 1)));
  }
};

/// }}}--- ///

// BIT<> bit(N);
```


