---
layout: lib
title: 根付き木に対する部分木クエリ (DSU on Tree)
permalink: graph/DSU-on-Tree

---


根付き木に対し部分木クエリを行います

# キーワード

**部分木クエリ**をします

* 条件をみたすものを数えよ

# 説明

[\[Tutorial\] Sack (dsu on tree)](https://codeforces.com/blog/entry/44351){:target="_blank"}<!--_--> がわかりやすいです

"v, cが与えられ，vの部分木内の色cの頂点の数を答えろ" みたいなのに対応できます

より正確には，これは実装を見てから理解したほうがいいと思うが，

* ある頂点を 有効化/無効化 する
* 各頂点vについて部分木が有効になったタイミングでクエリに答える

といったことができればいい

これは $O(N \log N)$ でできる．色々な書き方が紹介されているが，どれかひとつ把握すればいいと思う．HLD Styleを紹介する

# 計算量について

[HL分解]({{ "graph/HL-Decomposition" | absolute_url }}) による木の変換を考える．HLD Styleは Light Edge を登ったタイミングで全ての部分木内の頂点を走査している

ここで各頂点についていつ走査されるか考える．これは根へと向かう途中にある Light Edge の本数だが，これは $O(\log N)$ 本となる．よって全体で $O(N \log N)$ となる

# 実装


```cpp
// DSU_on_tree( <tree> , root, add, rem, proceed)
// add, rem, proceed is (vertex) => void
// O(N log N)
// DSU on Tree {{"{{"}}{
#include <cassert>
#include <functional>
#include <stack>
#include <vector>
// HLD style
class DSU_on_tree {
public:
  using size_type = size_t;
  using graph_type = vector< vector< int > >;
  using function_type = function< void(size_type) >;
  size_type n;
  graph_type tree;
  size_type root;
  function_type add, rem, proceed;

  DSU_on_tree() : n(0) {}
  DSU_on_tree(const graph_type &tree, size_type root, const function_type &add,
              const function_type &rem, const function_type &proceed)
      : n(tree.size()), tree(tree), root(root), add(add), rem(rem), proceed(proceed) {
    assert(root < n);
  }

private:
  bool initiated = 0;
  vector< int > heavy;
  vector< size_type > sub;

public:
  int getHeavy(size_type i) {
    assert(initiated);
    assert(i < n);
    return heavy[i];
  }

public:
  void init() {
    assert(!initiated);
    sub.resize(n);
    heavy.resize(n, -1);
    dfs_size();
    initiated = 1;
    dfs();
  }

private:
  void dfs_size() {
    vector< int > used(n);
    stack< pair< size_type, int > > stk;
    stk.emplace(root, -1);
    while(stk.size()) {
      size_type i;
      int p;
      tie(i, p) = stk.top();
      if(!used[i]) {
        used[i] = 1;
        for(auto &j : tree[i])
          if(j != p) {
            stk.emplace(j, i);
          }
      } else {
        stk.pop();
        sub[i] = 1;
        size_type max_size = 0;
        for(auto &j : tree[i])
          if(j != p) {
            sub[i] += sub[j];
            if(heavy[i] == -1 || max_size < sub[j]) max_size = sub[j], heavy[i] = j;
          }
      }
    }
  }
  void dfs() {
    vector< int > used(n);
    stack< tuple< size_type, int, bool > > stk;
    stk.emplace(root, -1, 1);
    while(stk.size()) {
      size_type i;
      int p;
      bool keep;
      tie(i, p, keep) = stk.top();
      if(!used[i]) {
        used[i] = 1;
        if(heavy[i] != -1) stk.emplace(heavy[i], i, 1);
        for(auto &j : tree[i])
          if(j != p && j != heavy[i]) stk.emplace(j, i, 0);
      } else {
        stk.pop();
        dfs_func(i, p, heavy[i], add);
        proceed(i);
        if(!keep) dfs_func(i, p, -1, rem);
      }
    }
  }
  void dfs_func(size_type i, int p, int dontGo, const function_type &func) {
    stack< pair< size_type, int > > stk;
    stk.emplace(i, p);
    while(stk.size()) {
      size_type i;
      int p;
      tie(i, p) = stk.top();
      stk.pop();
      func(i);
      for(auto &j : tree[i])
        if(j != p && j != dontGo) {
          stk.emplace(j, i);
        }
    }
  }
};
// }}}
```


# 検証

* [ECR47 F - Dominant Indices - codeforces](https://codeforces.com/contest/1009/submission/50643706){:target="_blank"}<!--_-->
  * 少し応用．`rem` が行われるタイミングで完全にリセットして，`add` が行われたらその頂点を候補とする，`ans[heavy[i]]` との比較をする，といったことをしている

# 練習問題

* [ECR47 F - Dominant Indices - codeforces](https://codeforces.com/contest/1009/problem/F){:target="_blank"}<!--_-->

# 参考

* [\[Tutorial\] Sack (dsu on tree)](https://codeforces.com/blog/entry/44351){:target="_blank"}<!--_-->

