---
layout: lib
title: '凸法(Graham Scan, Andrew Scan)'
permalink: geometory/grahamscan-polar

---


蟻本は左右に行って帰って，と走査しますが，  
ぼくはなんとなく角度でソートします．

一番左を捉えて原点とし，角度 = 傾き でソートして，ccwをたよりに調整します．

[幾何基本ライブラリ]({{ "geometory/geometory" | absolute_url }})が必要です．

なぜかキャリパー法が基本に入っててよくわかりません．（んぇ

蟻本には，座標値の幅が $M$ であるおうな凸多角形の頂点数は $O(\sqrt{M})$ 個しかない，とあるので押さえておいて損はないでしょう．（証明探さなきゃな…

あと，Graham ScanとAndrew Scanの違いはというと，角度ソートかx座標ソートかの違いのようです． [Graham scan \| Wikipedia(en)](https://en.wikipedia.org/wiki/Graham_scan){:target="_blank"}


```cpp
// require Geometory Library!
/// --- Graham Scan - Polar Sort - Library {{"{{"}}{ ///

// make ConvexHull

// if there are 3 points on line,
// just guaranteed that point on hull is scanned exactly;
// if on Line of Hull, sometimes scanned as on Hull.
struct GrahamScan {
  Polygon poly;
  vector< size_t > ids;
  vector< size_t > hull;

  void add(Point p) { poly.emplace_back(p); }

  int operator[](int i) {
    return hull[(i % hull.size() + hull.size()) % hull.size()];
  }

  void scan() {
    ids.resize(poly.size());
    iota(begin(ids), end(ids), 0);

    size_t startID = 0;
    for(size_t i = 1; i < poly.size(); i++) {
      if(make_pair(poly[startID].X, poly[startID].Y) >
         make_pair(poly[i].X, poly[i].Y))
        startID = i;
    }
    swap(ids[0], ids[startID]);

    sort(begin(ids) + 1, end(ids), [&](int a, int b) {
      Point p1 = poly[a] - poly[ids[0]];
      Point p2 = poly[b] - poly[ids[0]];
      // p1.y / p1.x < p2.y / p2.x
      double ev = p1.Y * p2.X - p2.Y * p1.X;
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


# 検証

* [村の道路計画 \| PCK \| AOJ](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/12/0342/judge/2473617/C++){:target="_blank"}
  * まるで検証しているようで，その場で書いたコードなので検証になっていない
* [B - Holes \| AC](https://beta.atcoder.jp/contests/agc021/submissions/2145093){:target="_blank"}
  * この問題はもっと低速な方法でいい


