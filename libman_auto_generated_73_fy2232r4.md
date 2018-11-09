---
layout: lib
title: Convex Hull Trick; CHT
permalink: misc/ConvexHullTrick

---


直線の集合 $L = \\{f_i(x) = a_i x + b_i\\}$ に対し，以下のクエリを高速に行えます
* add(a, b) : 直線 $f(x) = ax + b$ を追加するクエリ
* query(x) : $x$ に対して，最小値を取るような直線に対するその最小値を返すクエリ
  * $\displaystyle\min\_{f\in L}{f(x)}$

DPにも使える (ページ下部の参考も参照してください)

直線の数を$N$，クエリの数を$Q$ とする

# 通常

使い方はコードの下．オンラインクエリ可能


```cpp
// CHT<T, x-increasing?, Comp>()
// - maximize : let Comp = greater<T>
// .add(a, b) : f(x) = ax + b
// - minimize : (a) desc
// - maximize : (a) asc
/// --- Convex Hull Trick Library {{"{{"}}{ ///

#include <cassert>
#include <functional>
#include <utility>
#include <vector>

template < class T = long long, bool xIncreasing = false,
           class Comp = less< T > >
struct CHT {
  static T EPS;
  static Comp comp;

private:
  using Line = pair< T, T >;

public:
  vector< Line > lines;
  // is l2 unnecessary ?
  bool check(Line l1, Line l2, Line l3) {
    if(l2.first == l3.first) return 1;
    // cp(l2, l3).x <= cp(l2, l1).x
    return (l2.first - l1.first) * (l3.second - l2.second) + EPS >=
           (l3.first - l2.first) * (l2.second - l1.second);
  }
  T f(int i, const T &x) { return lines[i].first * x + lines[i].second; }
  void add(const T &a, const T &b) {
    assert("add monotonic" && (lines.empty() || !comp(lines.back().first, a)));
    if(lines.size() && lines.back().first == a && !comp(b, lines.back().second))
      return;
    while((int) lines.size() >= 2 &&
          check(lines[lines.size() - 2], lines.back(), Line(a, b)))
      lines.pop_back();
    lines.emplace_back(a, b);
  }
  T query(const T &x) {
    Line p = get(x);
    return p.first * x + p.second;
  }
  pair< T, T > get(const T &x) {
    assert(lines.size());
    if(xIncreasing) {
      static size_t head = 0;
      if(head >= lines.size()) head = lines.size() - 1;
      while(head + 1 < lines.size() && comp(f(head + 1, x), f(head, x))) head++;
      return lines[head];
    } else {
      int ok = lines.size() - 1, ng = -1;
      while(ok - ng > 1) {
        int mid = (ok + ng) >> 1;
        if(comp(f(mid, x), f(mid + 1, x)))
          ok = mid;
        else
          ng = mid;
      }
      return lines[ok];
    }
  }
};

template < class T, bool xIncreasing, class Comp >
T CHT< T, xIncreasing, Comp >::EPS = 1e-19;

template < class T, bool xIncreasing, class Comp >
Comp CHT< T, xIncreasing, Comp >::comp;

/// }}}--- ///
```


* `CHT<T, bool xIncreasing, Comp>`
  * `xIncreasing` : クエリ (get, query) の `x` が広義単調増加かどうか
  * デフォルトで最小値．`Comp = greater<T>` とすれば最大値
* `add(a, b)` : ならし $O(1)$
  * `f(x) = ax + b` を追加する
  * 最小化 : `a` の広義降順，`b` は自由
  * 最大化 : `a` の広義昇順，`b` は自由
  * 違反すると `assert` に引っかかる
  * この順序を任意の順で行うこともできる ([#追加順序任意のCHT](#追加順序任意のcht))
* `get(x) : (a, b)`
  * 単調 (x-increasing) なら 全体 $O(N + Q)$．そうでないなら 1回 $O(\log N)$
  * `x` を代入したときの値が最小 (最大) となる直線を `(a, b)` の形で返す
* `query(x)` : getと同様の計算量
  * `x` での最小 (最大) 値

## 検証

* [C - スペースエクスプローラー高橋君 - AtCoder](https://beta.atcoder.jp/contests/colopl2018-final-open/submissions/2171456){:target="_blank"}<!--_-->
  * このときの実装は少々バグがあります
* [D - Computer Game - Codeforces](https://codeforces.com/contest/1067/submission/45446448){:target="_blank"}<!--_-->

# 追加順序任意のCHT

Dynamic CHT; DCHT と呼ぶことにしておく．こいつが動的CHTなのか削除可能なものがそうなのか，よくわかっていない

オンラインクエリ可能

[http://d.hatena.ne.jp/sune2/20140310/1394440369](http://d.hatena.ne.jp/sune2/20140310/1394440369){:target="_blank"}<!--_--> にあるものに手を加えたもの


```cpp
// DynamicCHT<T, Comp>()
// - maximize : let Comp = greater<T>
// === --- ===
// N = no. of f(x)
// .add(a, b) : f(x) = ax + b : amortized O(log N)
// .query(x) : returns f(x), when f minimize f(x) : O(log N)
// .get(x) : returns f that minimize f(x) as (a, b) : O(log N)
// === --- ===
// f can duplicate
/// --- Dynamic Convex Hull Trick Library {{"{{"}}{ ///

#include <functional>
#include <limits>
#include <set>
#include <utility>

// |ab| < LLONG_MAX/4 ???
template < class T = long long, class Comp = less< T > >
struct DynamicCHT {
  static T INF;
  static T EPS;
  static Comp comp;

private:
  struct Line { // ax + b
    T a, b;
    Line(const T &a, const T &b) : a(a), b(b) {}
    bool operator<(const Line &rhs) const { // (a, b)
      return a != rhs.a ? comp(rhs.a, a) : comp(b, rhs.b);
    }
  };
  struct CP {
    T numer, denom; // x-coordinate; denom is non-negative for comparison
    Line p;
    CP(const T &n) : numer(n), denom(1), p(0, 0) {}
    // p1 < p2
    CP(const Line &p1, const Line &p2) : p(p2) {
      if(p1.a == INF || p1.a == -INF)
        numer = -INF, denom = 1;
      else if(p2.a == INF || p2.a == -INF)
        numer = INF, denom = 1;
      else {
        numer = p1.b - p2.b, denom = p2.a - p1.a;
        if(denom < 0) numer = -numer, denom = -denom;
      }
    }
    bool operator<(const CP &rhs) const {
      if(numer == INF || rhs.numer == -INF) return 0;
      if(numer == -INF || rhs.numer == INF) return 1;
      return numer * rhs.denom < rhs.numer * denom;
    }
  };
  set< Line > lines;
  set< CP > cps;
  typedef typename set< Line >::iterator It;

public:
  DynamicCHT() {
    // sentinel
    lines.insert({Line(INF, 0), Line(-INF, 0)});
    cps.insert(CP(Line(INF, 0), Line(-INF, 0)));
  }
  void add(const T &a, const T &b) {
    const Line p(a, b);
    It pos = lines.insert(p).first;
    if(check(*prev(pos), p, *next(pos))) {
      // ax + b is unnecessary
      lines.erase(pos);
      return;
    }
    cps.erase(CP(*prev(pos), *next(pos)));
    {
      It it = prev(pos);
      while(it != lines.begin() && check(*prev(it), *it, p)) --it;
      // lines (it, pos) is unnecessary
      // [it, pos - 1] : [pos - 1, pos] is still not added
      eraseRange(it, prev(pos));
      // [it + 1, pos - 1]
      lines.erase(++it, pos);
      pos = lines.find(p);
    }
    {
      It it = next(pos);
      while(next(it) != lines.end() && check(p, *it, *next(it))) ++it;
      // lines (pos, it) is unnecessary
      // [pos + 1, it] : [pos, pos + 1] is still not added
      eraseRange(++pos, it);
      // [pos + 1, it - 1]
      lines.erase(pos, it);
      pos = lines.find(p);
    }
    cps.insert(CP(*prev(pos), *pos));
    cps.insert(CP(*pos, *next(pos)));
  }
  T query(const T &x) const {
    pair< T, T > p = get(x);
    return p.first * x + p.second;
  }
  pair< T, T > get(const T &x) const {
    const Line &p = (--cps.lower_bound(CP(x)))->p;
    return make_pair(p.a, p.b);
  }
  friend ostream &operator<<(ostream &os, const DynamicCHT &a) {
    os << "\n";
    os << "lines : " << a.lines.size() << "\n";
    for(auto &p : a.lines)
      os << "(" << p.a << ", " << p.b << ")"
         << "\n";
    os << "cross points : " << a.cps.size() << "\n";
    for(auto &p : a.cps)
      os << "(x = " << p.numer << "/" << p.denom << "; " << p.p.a << ", "
         << p.p.b << ")"
         << "\n";
    return os;
  }

private:
  // erase cp which can be made from lines [a, b]
  void eraseRange(It a, It b) {
    for(It it = a; it != b; ++it) cps.erase(CP(*it, *next(it)));
  }
  // p2 is unnecessary?
  bool check(const Line &p1, const Line &p2, const Line &p3) {
    if(p1.a == p2.a) return 1;
    if(p1.a == INF || p1.a == -INF || p3.a == INF || p3.a == -INF) return 0;
    //  cp(p2, p3).x <= cp(p2, p1).x
    return (p2.a - p1.a) * (p3.b - p2.b) + EPS >= (p2.b - p1.b) * (p3.a - p2.a);
  }
};

template < class T, class Comp >
T DynamicCHT< T, Comp >::INF = numeric_limits< T >::has_infinity
                                   ? numeric_limits< T >::infinity()
                                   : numeric_limits< T >::max();

template < class T, class Comp >
T DynamicCHT< T, Comp >::EPS = 1e-19;

template < class T, class Comp >
Comp DynamicCHT< T, Comp >::comp; // only for less and greater

/// }}}--- ///
```


インターフェースは通常と同じ

* `add` : ならし $O(\log N)$
* `get` : $O(\log N)$

## 検証

* [D - Computer Game - Codeforces](https://codeforces.com/contest/1067/submission/45442782){:target="_blank"}<!--_-->

# 削除可能なConvexHullTrick

TODO : 理解

* [yosupoさんの記事](http://yosupo.hatenablog.com/entry/2015/12/02/235855){:target="_blank"}<!--_-->

---

通常のCHTが既に動的なのでね，動的CHTってなんだろうね

sort + CHTよりDCHTのほうが漸近的に速いが，sort + CHTのほうがちょっと速いかなと言う感じ

# 自分用のメモ

直線が不必要な条件

$\displaystyle-\frac{b_3-b_2}{a_3-a_2} \leq -\frac{b_2-b_1}{a_2-a_1}$

について，最小化なら $a_3 \gt a_2 \gt a_1$ という条件が成り立っているので，
最小・最大化どちらでも $(a_3-a_2)(a_2-a_1) \gt 0$ となり，

$(b_3-b_2)(a_2-a_1) \geq (b_2-b_1)(a_3-a_2)$ へ一意に変形できる

# EPSについて

`(aの正の最小値)(bの正の最小値)` より小さな正の値を採用するといい

たとえばデフォルトでは `1e-19` になっているが，これは両方が小数9桁ずつを想定している

# 参考

* [Convex-Hull Trick - SATANIC++](http://satanic0258.hatenablog.com/entry/2016/08/16/181331){:target="_blank"}<!--_-->
* [Dynamic Programming Optimizations - Codeforcesの記事](https://codeforces.com/blog/entry/8219){:target="_blank"}<!--_-->

