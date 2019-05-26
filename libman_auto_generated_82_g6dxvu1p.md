---
layout: lib
title: '凸包 (Graham Scan, Andrew Scan)'
permalink: geometory/ConvexHull

---


二次元平面上の $N$ この点が与えられたときの，凸包 (を成す頂点) を $O(N \log N)$ で求めます

凸法とは，与えられた点に釘をうって，輪ゴムをかけたような図形のことです

(x, y)辞書順最小を原点とし，角度 = 傾き でソートして，ccwで調整

キャリパー法は上記リンク先にあります

格子点にのみ点がある時，座標の幅を $M$ とすると凸包の頂点数は $O(\sqrt{M})$ 個 (TODO : なにかしらの証明のリンク

あと，Graham ScanとAndrew Scanの違いはというと，角度ソートか座標ソートかの違いのようです [Graham scan - Wikipedia(en)](https://en.wikipedia.org/wiki/Graham_scan){:target="_blank"}<!--_-->

凸包が右・左から見えるかどうかによって「右側チェイン」「左側チェイン」と言ったりします (上下で分けることもできます)

[三次元凸包はここ]({{ "geometory/ConvexHull3D" | absolute_url }})

### 凸包構築の一般的なアルゴリズムついて

任意のケースに対する最良実行時間は$\Omega(N \log N)$で，漸近的に最速なアルゴリズムは $O(N \log H)$，$H$ は結果の凸包の頂点数

# 実装

[幾何基本ライブラリ]({{ "geometory/geometory" | absolute_url }})が必要

$O(N \log N)$

同一直線上にある場合は結構適当になっているので，そのような場合はAndrewScanのほうがいいかもしれない


```cpp
// require Geometory Library
/// --- Graham Scan Library {{"{{"}}{ ///

// make ConvexHull

// if there are 3 points on line,
// just guaranteed that point on hull is scanned exactly;
// if on Line of Hull, sometimes scanned as on Hull.
struct GrahamScan {
  Polygon poly;
  vector< size_t > ids;
  vector< size_t > hull;

  void add(Point p) { poly.emplace_back(p); }

  int operator[](int i) { return hull[(i % hull.size() + hull.size()) % hull.size()]; }

  void scan() {
    ids.resize(poly.size());
    iota(begin(ids), end(ids), 0);

    size_t startID = 0;
    for(size_t i = 1; i < poly.size(); i++) {
      if(make_pair(poly[startID].real(), poly[startID].imag()) >
         make_pair(poly[i].real(), poly[i].imag()))
        startID = i;
    }
    swap(ids[0], ids[startID]);

    sort(begin(ids) + 1, end(ids), [&](int a, int b) {
      Point p1 = poly[a] - poly[ids[0]];
      Point p2 = poly[b] - poly[ids[0]];
      // p1.y / p1.x < p2.y / p2.x
      double ev = p1.imag() * p2.real() - p2.imag() * p1.real();
      return abs(ev) < EPS ? norm(p1) < norm(p2) : ev < 0;
    });

    hull.emplace_back(ids[0]);
    for(size_t i = 1; i <= poly.size(); i++) {
      size_t ii = i % poly.size();
      while(hull.size() >= 2 &&
            ccw(poly[hull[hull.size() - 2]], poly[hull[hull.size() - 1]],
                poly[ids[ii]]) == -1)
        hull.pop_back();
      hull.emplace_back(ids[ii]);
    }

    hull.pop_back();
  }
};

/// }}}--- ///
```


## $O(N^2)$で構築する方法

全頂点について，他の頂点を偏角でソートして，隣り合う２つの点への角度が$\pi$を超えるかどうかで，「凸包上の点かどうか」が判定できます

# 検証

* [村の道路計画 (PCK) - AOJ](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/12/0342/judge/2473617/C++){:target="_blank"}<!--_-->
  * まるで検証しているようで，その場で書いたコードなので検証になっていない
* [B - Holes (600) - AtCoder](https://beta.atcoder.jp/contests/agc021/submissions/2145093){:target="_blank"}
  * この問題は $O(N^2)$ でもよい

# 参考

* 蟻本
* アルゴリズム・イントロダクション

