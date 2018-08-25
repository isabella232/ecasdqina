---
layout: lib
permalink: algorithm/FastZetaTransform
title: 高速ゼータ変換

---


工事中


```cpp
// a'[i] = sum(j は i を含む, a[j])
template < class T >
void zeta(vector< T > &a,
          function< T(T, T) > const &op = [](T a, T b) { return a + b; }) {
  int n = a.size();
  for(int i = 0; i < n; i++)
    for(int b = 0; b < (1 << n); b++)
      if(!(b & (1 << i))) a[b] = op(a[b], a[b | (1 << i)]);
}
```



```cpp
// zetaの逆操作
template < class T >
void moebius(vector< T > &a,
             function< T(T, T) > const &op = [](T a, T b) { return a - b; }) {
  int n = a.size();
  for(int i = 0; i < n; i++)
    for(int b = 0; b < (1 << n); b++)
      if(b & (1 << i)) a[b] = op(a[b], a[b | (1 << i)]);
}
```


