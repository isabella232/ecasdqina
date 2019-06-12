---
layout: lib
title: 最小全域木 (borůvka法)
permalink: graph/boruvka

---


ブルーフカ法

全頂点を一頂点からなる連結成分であるとする．各連結成分から他の連結成分へ繋がる一番コストが小さい辺を選ぶ，ということを繰り返す

各ステップで**連結成分の数**の減少を少なく抑えようとしても必ず半分以下になる

よって $O(\log N)$ 回のステップ，各ステップで [UnionFind]({{ "data-structure/misc/UnionFind" | absolute_url }}) による連結性の計算，最小コスト辺の計算の分だけの時間がかかる

実装中の関数 `f` は所属を受け取り，各所属ごとの最小コスト辺と行き先を返す関数とする

計算量は $O(\sum_n f(n) + N \alpha (N) \log N)$ で，$\alpha (N)$ は [UnionFind]({{ "data-structure/misc/UnionFind" | absolute_url }}) の操作にかかる時間，$n$ は $N, \lfloor N/2 \rfloor, \lfloor \lfloor N/2 \rfloor /2 \rfloor, \dots, 1$ を回す

例えば，通常のMSTであれば $f(n) = M$ となる (全ての辺を調べれば良いので，$n$に依存しない)

# 実装


```cpp
// require UnionFind
// F(component-size, belongs) -> vector<(cost, to)>
// boruvka {{"{{"}}{
#include <vector>
template < class T, class F >
T boruvka(int n, const F &f) {
  UF uf(n);
  T res(0);
  bool update = 1;
  vector< int > belongs(n), rev(n);
  while(update) {
    update = 0;
    int ptr = 0;
    for(int i = 0; i < n; i++)
      if(i == uf.find(i)) rev[ptr] = i, belongs[i] = ptr++;
    for(int i = 0; i < n; i++) belongs[i] = belongs[uf.find(i)];
    auto v = f(ptr, belongs);
    for(int i = 0; i < ptr; i++)
      if(v[i].second >= 0 && uf.unite(rev[i], rev[v[i].second]))
        res += v[i].first, update = 1;
    if(!update) break;
  }
  return res;
}
// }}}
```


# 検証

* [KEYENCE2019 E - Connecting Cities (600) - AtCoder](https://atcoder.jp/contests/keyence2019/submissions/4012572){:target="_blank"}<!--_-->

# 参考

* [ブルーフカ法 - Wikipedia](https://ja.wikipedia.org/wiki/ブルーフカ法){:target="_blank"}<!--_-->

