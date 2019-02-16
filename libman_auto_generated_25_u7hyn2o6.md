---
layout: lib
title: 二重連結成分分解
permalink: graph/BiconnectedComponent

---

関節点(Articulatioin)で**辺の集合に**分解．

lowlinkを使わずに直接構築するのが実装が結構楽．


```cpp
// NOTE : 二重連結成分分解
/// --- Biconnected Component Decomposition {{"{{"}}{ ///

struct Biconnected {
  int n;
  vector< vector< int > > g;
  vector< int > low, ord, used;
  vector< vector< pair< int, int > > > comps;
  int id = 0;
  vector< pair< int, int > > tmp;
  Biconnected(vector< vector< int > > g) : n(g.size()), g(g), low(n), ord(n), used(n) {
    for(int i = 0; i < n; i++)
      if(!used[i]) dfs(i);
  }

private:
  void dfs(int i, int p = -1) {
    used[i] = 1;
    ord[i] = low[i] = id++;
    for(int j : g[i])
      if(j != p) {
        pair< int, int > e(min(i, j), max(i, j));
        if(used[j]) {
          if(ord[i] > ord[j]) tmp.emplace_back(e);
          low[i] = min(low[i], ord[j]);
        } else {
          tmp.emplace_back(e);
          dfs(j, i);
          low[i] = min(low[i], low[j]);
          if(low[j] >= ord[i]) {
            comps.push_back({});
            while(1) {
              pair< int, int > ne = tmp.back();
              comps.back().emplace_back(ne);
              tmp.pop_back();
              if(e == ne) break;
            }
          }
        }
      }
  }
};

/// }}}--- ///
```


# 検証

* [F - AtCoDeerくんとグラフ色塗り / Painting Graphs with AtCoDeer - AtCoder](https://beta.atcoder.jp/contests/arc062/submissions/2529990){:target="_blank"}
