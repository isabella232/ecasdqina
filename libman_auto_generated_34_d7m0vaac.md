---
layout: lib
title: 最大フロー (Dinic)
permalink: graph/flow/Dinic

---


[FordFulkerson]({{ "graph/flow/FordFulkerson" | absolute_url }})と基本的なところは同じ

流すことのできる辺のみをつかったBFSをして，sourceからの距離を求めて，距離が増える方へのみフローを流す，というのを繰り返す

$O(EV^2)$

* コスト1のみのとき， $O(\min\{V^{2/3}, E^{1/2}\}E)$
* 二部マッチングで使うときは $O(\sqrt{V}E)$

動的なもの（あとで辺の重みや辺自体が減ったり増えたり）ができなかった  
[FordFulkerson]({{ "graph/flow/FordFulkerson" | absolute_url }})ならできる  
なんでかとかはわかってない  
ちゃんと考えるとできるのかもしれない  

実用上はとても速いが最悪ケースが知られているため，任意のグラフが与えられるような問題は向いていない．**グラフに帰着できる問題**などは**意図的にグラフを操作しにくい**のであれば，Dinic法が有効，ということがある (参考 : [ここ](http://topcoder.g.hatena.ne.jp/Mi_Sawa/20140311){:target="_blank"}<!--_-->の「結論」)

# 実装


```cpp
// constructor(n, inf?) // be careful !
// addEdge(from, to, capacity, isDirected? = false)
//  - returns edgeID
// === build(s, t) - returns max flow (or inf) ===
// .edges[edgeID]
// === restoreMinCut(s) ===
// .isCut[edgeID]
// === --- ===
// inf * 2 < LL_MAX
/// --- Max Flow with Dinic {{"{{"}}{ ///
#include <algorithm>
#include <queue>
#include <vector>
struct Dinic {
  struct Edge {
    int from, to;
    ll cap, rev;
    int To(int i) const { return from == i ? to : from; }
    ll& Cap(int i) { return from == i ? cap : rev; }
    ll& Rev(int i) { return from == i ? rev : cap; }
  };

  int n;
  vector< Edge > edges;
  vector< vector< int > > g;
  ll inf;
  Dinic(int n, ll inf = 1e18) : n(n), g(n), inf(inf) {}
  int addEdge(int a, int b, ll cap, bool undirected = false) {
    edges.emplace_back((Edge){a, b, cap, undirected ? cap : 0});
    g[a].emplace_back(edges.size() - 1);
    g[b].emplace_back(edges.size() - 1);
    return edges.size() - 1;
  }
  vector< int > level;
  vector< size_t > itr;
  ll build(int s, int t) {
    level.resize(n);
    itr.resize(n);
    ll flow = 0, newflow;
    while(bfs(s), level[t] >= 0) {
      itr.assign(n, 0);
      while((newflow = dfs(s, t, inf)) > 0) {
        flow += newflow;
        if(flow >= inf) return inf;
      }
    }
    return flow;
  }
  vector< int > isCut;
  void restoreMinCut(int s) {
    isCut = vector< int >(edges.size());
    // bfs
    vector< int > used(n);
    queue< int > q;
    q.emplace(s);
    used[s] = 1;
    while(q.size()) {
      int i = q.front();
      q.pop();
      for(int idx : g[i]) {
        Edge& edge = edges[idx];
        if(!used[edge.To(i)] && edge.Cap(i) > 0) {
          q.emplace(edge.To(i));
          used[edge.To(i)] = 1;
        }
      }
    }
    for(size_t i = 0; i < edges.size(); i++) {
      if(used[edges[i].from] != used[edges[i].to]) isCut[i] = 1;
    }
  }

private:
  void bfs(int s) {
    fill(begin(level), end(level), -1);
    queue< int > q;
    q.emplace(s);
    level[s] = 0;
    while(q.size()) {
      int i = q.front();
      q.pop();
      for(int idx : g[i]) {
        Edge edge = edges[idx];
        if(level[edge.To(i)] < 0 && edge.Cap(i) > 0) {
          level[edge.To(i)] = level[i] + 1;
          q.emplace(edge.To(i));
        }
      }
    }
  }

  ll dfs(int v, int t, ll flow) {
    if(v == t) return flow;
    for(size_t& i = itr[v]; i < g[v].size(); ++i) {
      Edge& edge = edges[g[v][i]];
      if(edge.Cap(v) > 0 && level[edge.To(v)] > level[v]) {
        ll newflow = dfs(edge.To(v), t, min(flow, edge.Cap(v)));
        if(newflow == 0) continue;
        edge.Cap(v) -= newflow;
        edge.Rev(v) += newflow;
        return newflow;
      }
    }
    return 0;
  }
};
/// }}}--- ///

const int N = 2e6;
const ll inf = 1e18;
Dinic flow(N, inf);
```


# 検証

* [AGC029F - Construction of a tree - AtCoder](https://atcoder.jp/contests/agc029/submissions/3823160){:target="_blank"}<!--_-->

# 練習問題

* [ARC074F - Lotus Leaves (800) - AtCoder](https://atcoder.jp/contests/arc074/tasks/arc074_d){:target="_blank"}<!--_-->
* [AGC029F - Construction of a tree (2200) - AtCoder](https://atcoder.jp/contests/agc029/tasks/agc029_f){:target="_blank"}<!--_-->

# 参考

* [Dinic's algorithm - wikipedia(en)](https://en.wikipedia.org/wiki/Dinic%27s_algorithm){:target="_blank"}
* [Dinic法 - Algoogle](http://algoogle.hadrori.jp/algorithm/dinic.html){:target="_blank"}
* [最大流問題について. - れんしゅうちょう。 - TopCoder部](http://topcoder.g.hatena.ne.jp/Mi_Sawa/20140311){:target="_blank"}<!--_-->
  * Dinic法とその計算量，最悪ケースや最悪ケースについて書かれています
* [最大流問題について その3 - れんしゅうちょう。 - TopCoder部](http://topcoder.g.hatena.ne.jp/Mi_Sawa/20140320){:target="_blank"}<!--_-->
  * LCTでの高速化について書かれています

