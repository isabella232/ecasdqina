---
layout: lib
title: HL分解 (Heavy-Light Decomposition)
permalink: graph/HL-Decomposition

---


木を**パスを頂点とする木**に分解します

新しい木は高さが $O(\log N)$ となります

頂点にデータを乗せて，パスクエリと部分木クエリをこなすことができます

パスクエリだけであれば順序 (親から子への**方向**) を保つことができます (すなわち [モノイド]({{ "math/Monoid" | absolute_url }}) を処理できるということ．遅延セグメントツリーを用いてパス更新も可能．[#練習問題](#練習問題) の [ネットワークの課金システム](https://onlinejudge.u-aizu.ac.jp/problems/0367){:target="_blank"}<!--_--> などがこの性質を使います)

副産物としてLCAなどを求めることができます

---

もし根付きでなければ適当に根を決める．全ての頂点からその子へと伸びる辺について，**一番部分木のサイズが大きい頂点へとつながる辺** を Heavy Edge, それ以外を Light Edge と呼びます

そうすると，根から順にLight Edgeをたどることは $O(\log N)$ 回しかできません．なぜなら，Light Edgeを選ぶと部分木のサイズが $1/2$ 以下になるからです

Heavy Edgeによるパスを考えます．これを一つの頂点と捉え，Light Edgeが新たな辺として木を成していると捉えることができます

Heavy Edgeの処理は Euler Tour と同様にして [セグメントツリー]({{ "data-structure/SegmentTree/SegmentTree" | absolute_url }}) などを用いればよいです

# ダブリングと比べて

||HLD with 遅延セグ木|[ダブリング]({{ "graph/DoublingTree" | absolute_url }})|
|---|---|---|---|
|クエリあたり時間計算量|$O(\log ^ 2 N)$|$O(\log N)$|
|メモリ|$O(N)$|$O(N \log N)$|
|区間更新|**できる**|できない|
|点更新|**できる**|できない|

# 実装


```cpp
// query(hi, lo, func, inclusive?)
// hld[i] : index on sequence
// WARN : build after adding edges!
/// --- HL-Decomposition Library {{"{{"}}{ ///

struct HLD {
  int n;
  vector< int > head;
  vector< int > sz;
  vector< int > dep;
  vector< int > par;
  vector< int > vid;
  int id = 0;
  vector< vector< int > > g; // tree
  HLD(int n) : n(n), head(n), sz(n), dep(n), par(n), vid(n), g(n) {}
  HLD(vector< vector< int > > g, int root = 0) : HLD(g.size()) {
    this->g = g;
    build(root);
  }
  int operator[](int i) { return vid[i]; }
  void addEdge(int a, int b) {
    g[a].emplace_back(b);
    g[b].emplace_back(a);
  }
  void build(int root = 0) {
    head[root] = root;
    dfs0(root, -1, 0);
    dfs1(root, -1);
  }
  int lca(int a, int b) {
    while(1) {
      if(vid[a] > vid[b]) swap(a, b);
      if(head[a] == head[b]) return a;
      b = par[head[b]];
    }
  }
  void query(int hi, int lo, function< void(int, int) > f, bool inclusive = true) {
    while(lo != -1 && dep[lo] >= dep[hi]) {
      int nex = max(vid[head[lo]], vid[hi]);
      f(nex + (nex == vid[hi] && !inclusive), vid[lo] + 1);
      lo = par[head[lo]];
    }
  }

private:
  void dfs0(int i, int p, int d) {
    par[i] = p;
    sz[i] = 1;
    dep[i] = d;
    for(int &j : g[i])
      if(j != p) {
        dfs0(j, i, d + 1);
        sz[i] += sz[j];
        if(sz[j] > sz[g[i][0]]) {
          swap(g[i][0], j);
        }
      }
  }
  void dfs1(int i, int p) {
    vid[i] = id++;
    for(int j : g[i])
      if(j != p) {
        head[j] = j == g[i][0] ? head[i] : j;
        dfs1(j, i);
      }
  }
};

/// }}}--- ///
```


# 検証

* [PCK 2017 pre 12 ネットワークの課金システム - AOJ](https://onlinejudge.u-aizu.ac.jp/solutions/problem/0367/review/3114389/luma/C++14){:target="_blank"}<!--_-->
* [No.235 めぐるはめぐる (5) - yukicoder](https://yukicoder.me/submissions/278941){:target="_blank"}<!--_-->

# 練習問題

* [PCK 2017 pre 12 ネットワークの課金システム - AOJ](https://onlinejudge.u-aizu.ac.jp/problems/0367){:target="_blank"}<!--_-->
* [No.235 めぐるはめぐる (5) - yukicoder](https://yukicoder.me/problems/no/235){:target="_blank"}<!--_-->

