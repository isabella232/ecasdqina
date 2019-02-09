---
layout: lib
title: Monotone Minima
permalink: dynamic-programming/speedup/Monotone-Minima

---


執筆中です

TODO : 図となにか

# 移動回数は準線形

$y, x$ が1ずつしか移動できないとします ([Mo]({{ "algorithm/Mo" | absolute_url }}) のようなイメージ)

$y$ は $O(H \log H)$ 回，$x$ は $O(W \log H)$ 回となります

解析は再帰の深さが $O(\log H)$ で各深さでそれぞれ $O(H)$, $O(W)$ だけ動けばよいからです (この証明はnoshiさんに教えてもらいました)

2冪に丸めて完全二分木上のDFSの経路の長さを解析することでも同様の結果を得られます

この性質が使える問題を [Divide and Conquer Optimization]({{ "dynamic-programming/Divide-and-Conquer-Optimization" | absolute_url }}) で紹介しています

# 実装


```cpp
// A[y] := min argmin ( 0 <= x < m, w(y, x) )
// monotone is A[y] <= A[y+1]
// y moves O(H log H) times
// x moves O(W log H) times
// O(m log n)
/// --- Monotone Minima {{"{{"}}{ ///
#include <algorithm>
#include <functional>
#include <type_traits>
#include <utility>
#include <vector>
enum MonotoneMinimaType {
  ALL,
  UPPER,
  UPPER_EXCLUSIVE, // exclude diagonal
  LOWER,
  LOWER_EXCLUSIVE, // exclude diagonal
};
template < class T, class F, class Compare = less< T > >
auto MonotoneMinima(int n, int m, const F &w, T identity,
                    const MonotoneMinimaType type = ALL,
                    const Compare &comp = Compare()) {
#ifdef DEBUG
  static_assert(is_same< T, decltype(w(0, 0)) >::value, "T must equal to type of w");
#endif
  vector< pair< int, T > > argmin(n); // min argmin
  function< void(int, int, int, int) > f = [&](int yl, int yr, int xl, int xr) {
    if(yl > yr) return;
    int y = (yl + yr) / 2;
    int idx = -1;
    T mval = identity;
    for(int x = (type == UPPER || type == UPPER_EXCLUSIVE)
                    ? max(y + (type == UPPER_EXCLUSIVE), xl)
                    : xl;
        x <= (type == LOWER || type == LOWER_EXCLUSIVE
                  ? min(y - (type == LOWER_EXCLUSIVE), xr)
                  : xr);
        x++) {
      auto val = w(y, x);
      // dump(val);
      if(idx == -1 || comp(val, mval)) {
        idx = x;
        mval = val;
      }
    }
    argmin[y] = make_pair(idx, mval);
    f(yl, y - 1, xl, idx);
    f(y + 1, yr, idx, xr);
  };
  f(0, n - 1, 0, m - 1);
  return argmin;
}
/// }}}--- ///
```


# 参考

* [Totally Monotone Matrix Searching (SMAWK algorithm) - 週刊 spaghetti_source](https://topcoder.g.hatena.ne.jp/spaghetti_source/20120923/1348327542){:target="_blank"}<!--_-->
* [DP高速化 - ferinの競プロ帳](https://ferin-tech.hatenablog.com/entry/2018/02/23/071343){:target="_blank"}<!--_-->

