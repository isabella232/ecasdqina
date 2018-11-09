---
layout: lib
title: 行列の積と累乗
permalink: math/linear-algebra/matrix

---


時短ライブラリ


```cpp
/// --- Matrix mult pow Library {{"{{"}}{ ///

template < class T >
vector< vector< T > > mult(vector< vector< T > > a, vector< vector< T > > b) {
  assert(a[0].size() == b.size());
  vector< vector< T > > res(a.size(), vector< T >(b[0].size(), 0));
  for(size_t i = 0; i < a.size(); i++) {
    for(size_t j = 0; j < b[0].size(); j++) {
      for(size_t k = 0; k < b.size(); k++) {
        res[i][j] += a[i][k] * b[k][j];
      }
    }
  }
  return res;
}

template < class T >
vector< vector< T > > makeE(int n) {
  vector< vector< T > > r(n, vector< T >(n, 0));
  for(int i = 0; i < n; i++) r[i][i] = T(1);
  return r;
}

template < class T >
vector< vector< T > > pow(vector< vector< T > > a, ll k) {
  assert(a.size() == a[0].size());
  vector< vector< T > > r = makeE< T >(a.size());
  while(k) {
    if(k & 1) r = mult(r, a);
    a = mult(a, a);
    k >>= 1;
  }
  return r;
}

/// }}}--- ///

using Vec = vector< ll >;
using Mat = vector< Vec >;
```

