---
layout: lib
title: 最小全域有向木問題 (Chu-Liu/Edmonds' algorithm)
permalink: graph/MinimumSpanningArborescence

---


Minimum Spanning Arborscense

詳細は [#参考](#参考) を参照してください

関連 - [最小全域木]({{ "graph/MST" | absolute_url }})

# 実装

$O(NM)$ , 構築もする場合は $O(NM \log N)$ ぐらいだとおもうんですがもっと速いかもしれません

[SCC]({{ "graph/StronglyConnectedComponent" | absolute_url }}) が必要


```cpp
// require SCC
// MSA<T>( size )
// MSA<T>( <weighted-arborescence> )
// .add_edge(from, to, weight [, id])
// === .build() or .build_with_construct() ===
//    - O(NM), O(NM log N), resp.
//    - returns the sum value
// .get_using_edges() : returns ids
/// --- Minimum Spanning Arborescence {{"{{"}}{ ///
#include <set>
#include <tuple>
#include <vector>
template < class T >
struct MinimumSpanningArborescence {
public:
  using size_type = StronglyConnectedComponent::size_type;
  using graph_type = StronglyConnectedComponent::graph_type;
  using id_type = int;
  using weighted_graph_type =
      std::vector< std::vector< std::tuple< size_type, T, id_type, id_type > > >;
  using weighted_graph_without_id_type =
      std::vector< std::vector< std::tuple< size_type, T > > >;
  size_type n;
  weighted_graph_type graph;
  MinimumSpanningArborescence(size_type n) : n(n), graph(n) {}
  MinimumSpanningArborescence(const weighted_graph_without_id_type &input_graph)
      : n(input_graph.size()) {
    for(size_type from = 0; from < n; from++) {
      for(auto e : input_graph[from]) {
        auto to = std::get< 0 >(e);
        auto w = std::get< 1 >(e);
        add_edge(from, to, w);
      }
    }
  }

private:
  bool built = 0, built_with_construct = 0;
  std::vector< id_type > using_edges;

public:
  void add_edge(size_type from, size_type to, T w, size_type id = 0) {
    assert(!built);
    assert(from < n && to < n);
    graph[from].emplace_back(to, w, id, 0);
  }

public:
  T build(size_type root) {
    assert(!built);
    built = 1;
    std::multiset< id_type > _;
    return calc(graph, root, 0, 1, 0, _);
  }
  T build_with_construct(size_type root) {
    assert(!built);
    built = built_with_construct = 1;
    std::multiset< id_type > using_edges_ms;
    auto res = calc(graph, root, static_cast< T >(0), 1, 1, using_edges_ms);
    using_edges = std::vector< id_type >(using_edges_ms.begin(), using_edges_ms.end());
    return res;
  }
  std::vector< id_type > get_using_edges() {
    assert(built_with_construct);
    return using_edges;
  }

private:
  T calc(const weighted_graph_type &now, size_type start, T sum, bool first_time,
         bool do_construct, std::multiset< id_type > &using_edges) {
    size_type sz = now.size();
    std::vector< size_type > rev(sz, sz);
    std::vector< id_type > using_id(sz), remove_id(sz);
    std::vector< T > weight(sz);
    for(size_type i = 0; i < sz; i++) {
      for(auto e : now[i]) {
        size_type to;
        id_type id, rid;
        T w;
        std::tie(to, w, id, rid) = e;
        if(to == start) continue;
        if(rev[to] == sz || w < weight[to]) {
          weight[to] = w;
          rev[to] = i;
          using_id[to] = id;
          remove_id[to] = rid;
        }
      }
    }

    StronglyConnectedComponent scc(sz);
    for(size_type i = 0; i < sz; i++) {
      if(i == start) continue;
      scc.add_edge(rev[i], i);
      sum += weight[i];
      if(do_construct) {
        using_edges.insert(using_id[i]);
        if(!first_time) {
          using_edges.erase(using_edges.find(remove_id[i]));
        }
      }
    }

    scc.build();

    if(scc.component_size() == sz) return sum;
    weighted_graph_type arranged(scc.component_size());
    for(size_type i = 0; i < sz; i++) {
      for(auto e : now[i]) {
        size_type to, id;
        T w;
        std::tie(to, w, id, std::ignore) = e;
        if(scc[i] == scc[to]) continue;
        arranged[scc[i]].emplace_back(scc[to], w - weight[to], id, using_id[to]);
      }
    }
    return calc(arranged, scc[start], sum, 0, do_construct, using_edges);
  }
};
/// }}}--- ///
template < class T = long long >
using MSA = MinimumSpanningArborescence< T >;
```


# 検証

* [Spanning Tree - Minimum-Cost Arborescence - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/review.jsp?rid=3419423#1){:target="_blank"}<!--_-->

# 参考

* [最小全域有向木 - Chu-Liu/Edmondsのアルゴリズム](https://www.creativ.xyz/chu-liu-edmonds-522){:target="_blank"}<!--_-->
* [最小有向全域木を求める - Chu-Liu/Edmonds' algorithm - Ark's Blog](https://ark4rk.hatenablog.com/entry/2017/09/15/011937){:target="_blank"}<!--_-->
* [最小全域有向木（m log n） - ｼﾞｮｲｼﾞｮｲｼﾞｮｲ](http://joisino.hatenablog.com/entry/2017/01/11/230141){:target="_blank"}<!--_-->

