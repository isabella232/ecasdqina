---
layout: lib
title: 3次元幾何基本ライブラリ
permalink: geometory/geometory3D

---


[幾何基本ライブラリ]({{ "geometory/geometory" | absolute_url }}) が必要

`valarray`で三次元ベクトルを表現


```cpp
// require Geomeotry Library
/// --- Geometory 3D Library {{"{{"}}{ ///
#include <algorithm>
#include <valarray>

inline Scalar dot(const valarray< Scalar > &a, const valarray< Scalar > &b) {
  return (a * b).sum();
}
inline valarray< Scalar > cross(const valarray< Scalar > &a,
                                const valarray< Scalar > &b) {
  return a.cshift(1) * b.cshift(-1) - a.cshift(-1) * b.cshift(1);
}

template < class T >
inline T L1norm(const valarray< T > &a) {
  return abs(a).sum();
}
// L2 norm
template < class T >
inline Float norm(const valarray< T > &a) {
  return sqrt((a * a).sum());
}
template < class T >
inline T LINFnorm(const valarray< T > &a) {
  return abs(a).max();
}
// [0, pi]
Float arg(const valarray< Scalar > &a, const valarray< Scalar > &b) {
  Float val = (Float) dot(a, b) / norm(a) / norm(b);
  return acos(max(min(val, Float(1)), Float(-1)));
}
template < class T >
inline valarray< Float > to_float(const valarray< T > &a) {
  valarray< Float > a2(a.size());
  for(size_t i = 0; i < a.size(); i++) a2[i] = a[i];
  return a2;
}
inline valarray< Float > normalize(const valarray< Float > &a) {
  return a / norm(a);
}

/// }}}--- ///
```


# いろいろ

## ある多角形を三次元空間上である角度から見たときの面積

多角形の面積と同じサイズの，法線ベクトル $S$ と，
見ている角度を表す**単位**ベクトル $V$ の内積 $S \cdot V$ の絶対値

小さな長方形に区切って積分することを考えると納得できます ([参考](https://mathtrain.jp/yonheiho){:target="_blank"}<!--_-->)

## ２つの面が重ってできる直線

2つの面の法線の外積  
平行な場合は0となる

# 練習問題

* [L - 凸包が映し出される平面 (KUPC2018) - AtCoder](https://beta.atcoder.jp/contests/kupc2018/tasks/kupc2018_l){:target="_blank"}<!--_-->

# 参考

* [不遇の標準ライブラリ - valarray - SlideShare](https://www.slideshare.net/Ryosuke839/kbkz-tech8-lt){:target="_blank"}<!--_-->
* [四平方の定理（図形の面積と正射影） - 高校数学の美しい物語](https://mathtrain.jp/yonheiho){:target="_blank"}<!--_-->

