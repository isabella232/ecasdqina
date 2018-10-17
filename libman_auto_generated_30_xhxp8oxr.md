---
layout: lib
title: 最大フロー (Dinic)
permalink: graph/flow/Dinic

---


[FordFulkerson]({{ "graph/flow/FordFulkerson" | absolute_url }})と基本的なところは同じ．

流すことのできる辺のみをつかったBFSをして，sourceからの距離を求めて，  
距離が増える方へのみフローを流す，というのを繰り返す．

$ O(EV^2) $

* コスト1のみのとき， $O(\min\{V^{2/3}, E^{1/2}\}E)$
* 二部マッチングで使うときは $O(\sqrt{V}E)$

TODO : LCTで高速化できるらしく，やってみたく思っている．

動的なもの（あとで辺の重みや辺自体が減ったり増えたり）ができなかった．  
[FordFulkerson]({{ "graph/flow/FordFulkerson" | absolute_url }})ならできる．  
なんでかとかはわかってない．  
ちゃんと考えるとできるのかもしれない．

なんとなく動的を考慮した実装になっている．


```cpp
/// --- Dinic Libary {{"{{"}}{ ///
// solve max flow

struct Dinic {
  struct Edge {
    int from, to;
    ll cap, rev;
    int To(int i) { return from == i ? to : from; }
    ll& Cap(int i) { return from == i ? cap : rev; }
    ll& Rev(int i) { return from == i ? rev : cap; }
  };

  int n;
  vector< Edge > edges;
  vector< vector< int > > g;
  ll inf;
  Dinic(int n, ll inf = 1e18) : n(n), g(n), inf(inf) {}

  void addEdge(int a, int b, ll cap, int i = -1, bool undirected = false) {
    if(i == -1) i = edges.size();
    edges.resize(max(i + 1, (int) edges.size()));
    edges[i] = (Edge){a, b, cap, undirected ? cap : 0};
    g[a].emplace_back(i);
    g[b].emplace_back(i);
  }

  ll solve(int s, int t) {
    vector< int > level(n);
    ll flow = 0;
    while(bfs(s, level), level[t] > 0) {
      ll newflow = dfs(s, t, inf, level);
      if(newflow == 0) break;
      flow += newflow;
      if(flow >= inf) return inf;
    }
    return flow;
  }

private:
  void bfs(int s, vector< int >& level) {
    fill(begin(level), end(level), -1);
    queue< int > q;
    q.emplace(s);
    level[s] = 0;
    while(q.size()) {
      int i = q.front();
      q.pop();
      for(int idx : g[i]) {
        Edge edge = edges[idx];
        if(level[edge.To(i)] == -1 && edge.Cap(i) > 0) {
          level[edge.To(i)] = level[i] + 1;
          q.emplace(edge.To(i));
        }
      }
    }
  }

  ll dfs(int i, int t, ll flow, vector< int > const& level) {
    if(i == t) return flow;
    for(int idx : g[i]) {
      Edge& edge = edges[idx];
      if(edge.Cap(i) > 0 && level[edge.To(i)] > level[i]) {
        ll newflow = dfs(edge.To(i), t, min(flow, edge.Cap(i)), level);
        if(newflow == 0) continue;
        edge.Cap(i) -= newflow;
        edge.Rev(i) += newflow;
        return newflow;
      }
    }
    return 0;
  }
};

/// }}}--- ///
```


# 検証

* [F - Lotus Leaves - AtCoder](https://beta.atcoder.jp/contests/arc074/submissions/2141547){:target="_blank"}<!--_-->


# 参考

* [Dinic's algorithm - wikipedia(en)](https://en.wikipedia.org/wiki/Dinic%27s_algorithm){:target="_blank"}
* [Dinic法 - Algoogle](http://algoogle.hadrori.jp/algorithm/dinic.html){:target="_blank"}

