---
layout: lib
title: 時空間Mo (三次元Mo，Mo with Update)
permalink: algorithm/Mo3D

---


時空間Moって名前が好き

三次元っていうのは，Left, Right, Timeの三軸．

[通常のMo]({{ "algorithm/Mo" | absolute_url }})に加え，更新をする．

計算量は $O(N^{5/3})$ なので，いつでも使えるというわけではないが，強い．

バケットのサイズを $B$ とおくと，

* 左端Leftが動く回数は $O(QB)$，
* 右端Rightが動く回数は $O(QB + \frac{N^2}{B})$
* Timeが動く回数は $O(\frac{N^3}{B^2})$

$N = Q$ と仮定すると， $B=N^{2/3}$ とすることで上記の $O(N^{5/3})$ が得られます．

詳細・図解が[うしさんの解説記事](https://ei1333.hateblo.jp/entry/2017/09/11/211011){:target="_blank"}<!--_-->に詳しく書いてあるので見てみてください…

Moに加えて，変数 $time$ を，$[0, time]$ を処理済みとして保持します．

# 実装


```cpp
// Mo3D(N, JUST Q, double k)
// favored : k = 6
// 1: insert
// 2: build
/// --- Mo3D Library {{"{{"}}{ ///

struct Mo3D {
  const int width;
  int q = 0;
  vector< int > le, ri, idx, order;
  int nl = 0, nr = 0, time = -1;
  Mo3D(int n, int q, double k = 1)
      : width(int(k* pow(max(n, q), 2.0 / 3.0) + 1.0)),
        le(q),
        ri(q),
        idx(q),
        order(q) {}
  inline void insert(int t, int l, int r) {
    idx[q] = t;
    le[q] = l;
    ri[q] = r;
    order[q] = q;
    q++;
  }
  inline void build() {
    sort(begin(order), begin(order) + q, [&](int a, int b) {
      const int al = le[a] / width, bl = le[b] / width;
      const int ar = ri[a] / width, br = ri[b] / width;
      return al != bl ? al < bl : ar != br ? ar < br : idx[a] < idx[b];
    });
    nl = nr = le[order[0]];
    for(int i = 0; i < q; i++) {
      const int id = order[i];
      while(time < idx[id]) addQuery(++time);
      while(time > idx[id]) remQuery(time--);
      while(nl > le[id]) add(--nl);
      while(nr < ri[id]) add(nr++);
      while(nl < le[id]) rem(nl++);
      while(nr > ri[id]) rem(--nr);
      next(id);
    }
  }
  inline void next(int i);
  inline void addQuery(int i);
  inline void remQuery(int i);
  inline void add(int i);
  inline void rem(int i);
};

/// }}}--- ///

// i is sequential
// idx[i] is absolute
inline void Mo3D::next(int i) {}
inline void Mo3D::addQuery(int i) {}
inline void Mo3D::remQuery(int i) {}
inline void Mo3D::add(int i) {}
inline void Mo3D::rem(int i) {}
```


# 検証

* [F - Machine Learning - CF](https://codeforces.com/contest/940/submission/42752956){:target="_blank"}<!--_-->

# 練習

* [F - Machine Learning - CF](https://codeforces.com/contest/940/problem/F){:target="_blank"}<!--_-->

# 参考

* [Mo's algorithm - ei1333の日記](https://ei1333.hateblo.jp/entry/2017/09/11/211011){:target="_blank"}<!--_-->
