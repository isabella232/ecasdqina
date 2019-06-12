---
layout: lib
title: HL分解 (Heavy-Light Decomposition)
permalink: graph/HL-Decomposition

---


木を**パスを頂点とする木**に分解します

新しい木は高さが $O(\log N)$ となります

頂点にデータを乗せて，パスクエリをこなすことができます

さらに，EulerTourの要領で部分木クエリもできます

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
// HLD( <tree> , root )
// === .build() ===
// .fold(hi, lo, func, inclusive)
//   where func(l, r) proceeds with [l, r)
// === O(1) ===
// .in(a) : in-time of Euler Tour : alias = .[a]
// .out(a) : out-time of Euler Tour
// .rev(a) : rev[in[a]] = a
// .head(a) : ascend all light edges
// .tail(a) : descend all heavy edges
// ---
// .subtree_size(a)
// .depth(a) : 0-indexed
// .parent(a) : -1 if [a] is root
// .heavy(a) : [a] cannot be a leaf. return the node opposite of the heavy edge
// === O(log n) ===
// .climb(a)
// .descendTo(from, to, steps)
// .steps(a, b)
// === --- ===
// for subtree : [ .in(a)           , .out(a) )
// (exclusive) : [ .in_exclusive(a) , .out(a) )
// HL-Decomposition {{"{{"}}{
#include <cassert>
#include <functional>
#include <vector>
// based on Euler Tour
struct HLD {
public:
  using size_type = std::size_t;
  using graph_type = std::vector< std::vector< int > >;

private:
  size_type n;
  std::vector< size_type > hd, tl;
  std::vector< size_type > sub;
  std::vector< size_type > dep;
  std::vector< int > par;
  std::vector< size_type > vid;
  size_type root;
  graph_type tree;

public:
  HLD() : n(0) {}
  HLD(size_type n, size_type root = 0)
      : n(n), hd(n), tl(n), sub(n), dep(n), par(n), vid(n), tree(n) {
    setRoot(root);
  }
  HLD(const graph_type &tree, size_type root) : HLD(tree.size(), root) {
    this->tree = tree;
  }

  void setRoot(size_type root) {
    assert(root < n);
    this->root = root;
  }

private:
  bool built = 0;
  std::vector< size_type > vid_rev;

public:
  void build() {
    assert(!built && n);
    built = 1;

    vid_rev.resize(n);

    hd[root] = root;
    dfs0();
    dfs1();
    for(size_type i = 0; i < n; i++) vid_rev[vid[i]] = i;
  }

private:
  void dfs0() {
    std::vector< int > used(n);
    std::vector< std::tuple< size_type, int, size_type > > stk;
    stk.reserve(n);
    stk.emplace_back(root, -1, 0);
    while(stk.size()) {
      size_type i, d;
      int p;
      std::tie(i, p, d) = stk.back();
      if(!used[i]) {
        used[i] = 1;
        par[i] = p;
        dep[i] = d;
        for(auto &j : tree[i])
          if(j != p) {
            stk.emplace_back(j, i, d + 1);
          }
      } else {
        stk.pop_back();
        sub[i] = 1;
        for(auto &j : tree[i])
          if(j != p) {
            if(sub[j] > sub[tree[i].back()]) {
              std::swap(tree[i].back(), j);
            }
            sub[i] += sub[j];
          }
        if(tree[i].back() != p) {
          tl[i] = tl[tree[i].back()];
        } else {
          tl[i] = i;
        }
      }
    }
  }
  void dfs1() {
    std::vector< int > used(n);
    std::vector< std::tuple< size_type, int > > stk;
    stk.reserve(n);
    stk.emplace_back(root, -1);
    size_type id = 0;
    while(stk.size()) {
      size_type i;
      int p;
      std::tie(i, p) = stk.back(), stk.pop_back();
      vid[i] = id++;
      for(auto j : tree[i])
        if(j != p) {
          hd[j] = j == tree[i].back() ? hd[i] : j;
          stk.emplace_back(j, i);
        }
    }
  }

public:
  size_type operator[](size_type i) const { return in(i); }
  size_type in(size_type i) const {
    assert(built);
    assert(i < n);
    return vid[i];
  }
  size_type in_exclusive(size_type i) const { return in(i) + 1; }
  size_type out(size_type i) const {
    assert(built);
    assert(i < n);
    return vid[i] + sub[i];
  }
  size_type out_exclusive(size_type i) const { return out(i) - 1; }
  size_type head(size_type i) const {
    assert(built);
    return hd.at(i);
  }
  size_type tail(size_type i) const {
    assert(built);
    return tl.at(i);
  }
  size_type rev(size_type i) const {
    assert(built);
    return vid_rev.at(i);
  }
  size_type subtree_size(size_type i) const {
    assert(built);
    return sub.at(i);
  }
  size_type depth(size_type i) const {
    assert(built);
    return dep.at(i);
  }
  int parent(size_type i) const {
    assert(built);
    return par.at(i);
  }
  size_type steps(size_type a, size_type b) const {
    assert(built);
    assert(a < n && b < n);
    return dep[a] + dep[b] - 2 * dep[lca(a, b)];
  }
  size_type climb(size_type a, long long t) const {
    assert(built);
    assert(a < n && t >= 0);
    while(t) {
      long long c = std::min< long long >(vid[a] - vid[hd[a]], t);
      t -= c;
      a = vid_rev[vid[a] - c];
      if(t && a != root) {
        t--;
        a = par[a];
      }
      if(a == root) break;
    }
    return a;
  }
  size_type descendTo(size_type from, size_type to, long long steps) const {
    assert(built);
    assert(steps >= 0);
    assert(from < n && to < n);
    return climb(to, dep[to] - dep[from] - steps);
  }
  void add_edge(size_type a, size_type b) {
    assert(built);
    assert(a < n && b < n);
    tree[a].emplace_back(b);
    tree[b].emplace_back(a);
  }
  size_type lca(size_type a, size_type b) const {
    assert(built);
    assert(a < n && b < n);
    while(1) {
      if(vid[a] > vid[b]) std::swap(a, b);
      if(hd[a] == hd[b]) return a;
      b = par[hd[b]];
    }
  }
  size_type heavy(size_type a) const {
    assert(built);
    assert(a < n);
    assert(tree[a].back() != par[a]);
    return tree[a].back();
  }
  void fold(size_type hi, int lo, std::function< void(int, int) > f,
            bool inclusive) const {
    assert(built);
    assert(hi < n && 0 <= lo && lo < (int) n);
    while(lo != -1 && dep[lo] >= dep[hi]) {
      size_type nex = std::max(vid[hd[lo]], vid[hi]);
      f(nex + (nex == vid[hi] && !inclusive), vid[lo] + 1);
      lo = par[hd[lo]];
    }
  }
  size_type size() const { return n; }
};
// }}}
```


# 検証

* [PCK 2017 pre 12 ネットワークの課金システム - AOJ](https://onlinejudge.u-aizu.ac.jp/solutions/problem/0367/review/3114389/luma/C++14){:target="_blank"}<!--_-->
* [No.235 めぐるはめぐる (5) - yukicoder](https://yukicoder.me/submissions/278941){:target="_blank"}<!--_-->
* 部分木クエリ [RUPC 2015 - Tree - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/review.jsp?rid=3405804#1){:target="_blank"}<!--_-->

# 練習問題

* [PCK 2017 pre 12 ネットワークの課金システム - AOJ](https://onlinejudge.u-aizu.ac.jp/problems/0367){:target="_blank"}<!--_-->
* [No.235 めぐるはめぐる (5) - yukicoder](https://yukicoder.me/problems/no/235){:target="_blank"}<!--_-->
* [RUPC 2015 - Tree - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2667){:target="_blank"}<!--_-->

# 参考

* [競技プログラミングにおけるHL分解まとめ - はまやんはまやんはまやん](https://www.hamayanhamayan.com/entry/2017/04/10/172636){:target="_blank"}<!--_-->
* [Heavy-Light Decomposition - beet's soil](http://beet-aizu.hatenablog.com/entry/2017/12/12/235950){:target="_blank"}<!--_-->

