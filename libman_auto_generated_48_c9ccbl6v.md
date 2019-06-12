---
layout: lib
title: QI性を利用したDP高速化
permalink: dynamic-programming/speedup/QI-Speedup

---


文献 [^1] が全てです．そのうち詳しい解説を書くかもしれません

Convex/Concave QI Speedupという名前は適当につけました


$$
D(i) = g\left(\min_{0 \leq j \lt i}\{D(j) + w(j, i)\}\right)
$$

$D(0)$は分かっているとし，$D(N)$が求めたい (引数は全て整数です)

というDPを考えます．$g$は任意の関数で，何でもいいです (時間がかからなければ)
<!--_-->

愚直にやると$O(N^2)$ですが，$w$が [Concave Quadrangle Inequality]({{ "math/Monge#monge" | absolute_url }}) か [Convex Quadrangle Inequality]({{ "math/Monge#convex-qi" | absolute_url }}) を満たす時，これは$O(N \log N)$で求められます

$\min$ではなく$\max$という場合でも，$$w'(j, i)=-w(j, i)$$を考えることにより ($w$がConvex QIなら$$w'$$はConcave QIといった具合に) 計算することができます

# メモ

そのうち書くかもしれない解説のためのメモ

$w$がConvex QIとする

$C(j, i) = f(j) + w(j, i)$ は 「$i$ から $j$ を採用したときの損」と考える．損を最小化するのが目的

$l \lt k$ について $f(r) = C(l, r) - C(k, r)$ とは 「$k$ではなく$l$を選んだときの損」であり，これはConvex QIから (広義) 単調減少であることが分かる

つまり，「大きい方 ( = $k$) を採用する得はだんだん小さくなる」といえる

ここで，「大きい方はいつから採用することで得するか」に二分探索で答えられる

あとはstackやdequeを用いることで，これらが「いい性質」を保持し続けるように管理すればよい


# 実装


```cpp
// D(0) is given
// D(i) = f( min(0 <= j < i,  D(j) + w(j, i)) , i)
// w must satisfy Convex QI
// Convex QI is w[===] + w[-=-] <= w[==-] + w[-==]
// by default, f is identity function
// O(n log n)
// NOTE : w(j, i) is 1-indexed
// Convex QI Speedup {{"{{"}}{
#include <cassert>
#include <functional>
#include <stack>
#include <type_traits>
#include <vector>
template < class T, class W, class F = function< T(const T &) > >
auto ConvexQISpeedup(T d0, size_t n, const W &w,
                     const F &f = [](const T &t) { return t; }) {
#ifdef DEBUG
  static_assert(is_same< T, decltype(w(0, 0)) >::value, "T must equal to typeof w(...)");
  static_assert(is_same< T, decltype(f(d0)) >::value, "T must equal to typeof f(...)");
#endif

  vector< T > D(n + 1);
  D[0] = d0;

  // C(j, i) = D(j) + w(j, i)
  auto C = [&](size_t j, size_t i) { return D[j] + w(j, i); };

  // search the first time to choose l rather than k
  // if w satisfies Closest Zero Property, you can speedup h
  auto h = [&](size_t l, size_t k) {
    assert(l < k && k < n + 1);
    size_t ok = n + 1, ng = k;
    while(ok - ng > 1) {
      size_t mid = (ok + ng) >> 1;
      if(C(l, mid) <= C(k, mid))
        ok = mid;
      else
        ng = mid;
    }
    return ok;
  };

  // (k, h) where k is index and h is the time to die
  stack< pair< size_t, size_t > > S;
  S.emplace(0, n + 1);
  for(size_t i = 1; i <= n; i++) {
    size_t j = S.top().first;
    if(C(i - 1, i) >= C(j, i))
      D[i] = f(C(j, i));
    else {
      D[i] = f(C(i - 1, i));
      while(S.size() &&
            C(i - 1, S.top().second - 1) < C(S.top().first, S.top().second - 1))
        S.pop();
      if(S.empty())
        S.emplace(i - 1, n + 1);
      else
        S.emplace(j, h(S.top().first, i - 1));
    }
    if(S.top().second == i + 1) S.pop();
  }
  return D;
}
// }}}
```


// @ Concave QI ( = Monge ) Speedup

# 検証

* [No.705 ゴミ拾い Hard - yukicoder(submission)](https://yukicoder.me/submissions/312784){:target="_blank"}<!--_-->


# 練習問題

* [No.705 ゴミ拾い Hard - yukicoder](https://yukicoder.me/problems/no/705){:target="_blank"}<!--_-->

# 参考

[^1]: [Speeding up dynamic programming with applications to molecular biology](https://www.sciencedirect.com/science/article/pii/0304397589901011){:target="_blank"}<!--_-->

