---
layout: lib
title: 重心分解 (Centroid Decomposition)
permalink: graph/CentroidDecomposition

---


木を重心分解する

# キーワード

基本的にパスに関する質問

* ある点を始点とするパスのなかで 最適なものを答えよ / 条件をみたすものを数え上げよ
* ある条件を満たすパスを数え上げよ
* ある点を通る条件を満たすパスを数え上げよ

# メモ

始点指定なしのものはすべての点を始点とするクエリに置き換えることで **始点指定あり** に帰着できる (この帰着は必ずしも必要とは限らない)

重心を選択，排除を繰り返すことにより同じサイズの (有向) 木 (**重心木** と呼ぶことにする) が出来上がる

重心木は高さが $O(\log N)$ となる

また，各頂点$v$についてその頂点を重心とするような部分木を $v$**が支配する部分木** と呼ぶことにする

すべての頂点について，その頂点が支配する部分木を探索すると $O(N \log N)$ となる

以下，重心木を top-down に処理するか bottom-up に処理するかで何が変わるかのメモ

### top-downに処理する

DFSすればよい

各頂点，子 ([実装](#実装) の `.child[v]`) を探索する前にその頂点をdeadにしてから潜ると良い

分割統治や，予めそこを始点とするクエリを集めておく (オフラインクエリ) ことによって求められる

### bottom-upに処理する

各頂点について親 ([実装](#実装) の `.par[v]`) へとたどっていけばいい

以前いた頂点を含む部分木について省かなかればいけない場合は気をつける (数え上げなど; [検証](#検証) の "木の問題" を参照)

ダブリングなどを用いて二点間の距離を計算すると，オンラインでクエリを処理できる

# 実装


```cpp
// CentroidDecomposition( <tree> [, process ] )
// .setProcess(func)
// === .build() : returns root id ===
// info about reformed graph (depth is O(log N))
// .par[i] : int
// .child[i] : vector<int>
// .sz[i] : int : subtree size
/// --- Centroid Decomposition {{"{{"}}{ ///
#include <cassert>
#include <functional>
#include <vector>
struct CentroidDecomposition {
  using graph_type = vector< vector< int > >;
  using process_type = function< void(int centroid, const vector< bool > &) >;
  size_t n;
  graph_type tree;
  process_type process;
  CentroidDecomposition() : n(0) {}
  CentroidDecomposition(size_t n, const process_type process =
                                      [&](int, const vector< bool > &) -> void {})
      : n(n), tree(n), process(process) {}
  CentroidDecomposition(const graph_type &tree) : CentroidDecomposition(tree.size()) {
    this->tree = tree;
  }
  void setProcess(const process_type &process) { this->process = process; }
  void addEdge(size_t a, size_t b) {
    assert(a < n && b < n);
    tree[a].push_back(b);
    tree[b].push_back(a);
  }

private:
  vector< bool > processing;
  vector< size_t > sub;
  bool built = 0;

public:
  int root = -1;
  vector< int > par;
  vector< int > sz;
  vector< vector< int > > child;
  int build() {
    assert(!built);
    built = 1;
    processing.resize(n);
    sub.resize(n);
    par.resize(n);
    sz.resize(n);
    child.resize(n);
    return root = decomposite(0, -1);
  }

private:
  int decomposite(int start, int p) {
    dfs(start, -1);
    int centroid = search_centroid(start, -1, sub[start] / 2);
    sz[centroid] = sub[start];
    par[centroid] = p;
    processing[centroid] = 1;
    for(auto &j : tree[centroid])
      if(!processing[j]) child[centroid].push_back(decomposite(j, centroid));
    processing[centroid] = 0;
    process(centroid, processing);
    return centroid;
  }
  void dfs(int i, int p) {
    sub[i] = 1;
    for(auto &j : tree[i])
      if(j != p && !processing[j]) {
        dfs(j, i);
        sub[i] += sub[j];
      }
  }
  int search_centroid(int i, int p, size_t mid) {
    for(auto &j : tree[i])
      if(j != p && !processing[j]) {
        if(sub[j] > mid) return search_centroid(j, i, mid);
      }
    return i;
  }
};
/// }}}--- ///

// do NOT go to vertex v when u[v] is true !!!!
// void process(int c, const vector< bool > &u) {}
```


# 検証

bottom-up で書いた

* [#199 div2 E - Xenia and Tree - codefroces](https://codeforces.com/contest/342/submission/50313110){:target="_blank"}<!--_-->
* [みんプロ2018 決勝 C - 木の問題 (1200) - AtCoder](https://atcoder.jp/contests/yahoo-procon2018-final-open/submissions/4349163){:target="_blank"}<!--_-->

top-down で書いた

* [Path Inversions - CSAcademy](https://csacademy.com/submission/2164172/){:target="_blank"}<!--_-->

inline のみですませた

* [#458 E - Palindromes in a Tree - codeforces](https://codeforces.com/contest/914/submission/50353783){:target="_blank"}<!--_-->

# 練習問題

* [#199 div2 E - Xenia and Tree - codefroces](https://codeforces.com/contest/342/problem/E){:target="_blank"}<!--_-->
* [#190 div1 C - Ciel the Commander - codeforces](https://codeforces.com/problemset/problem/321/C){:target="_blank"}<!--_-->
* [Path Inversions - CSAcademy](https://csacademy.com/contest/archive/task/path-inversions/statement/){:target="_blank"}<!--_-->
* [#458 E - Palindromes in a Tree - codeforces](https://codeforces.com/contest/914/problem/E){:target="_blank"}<!--_-->
* [NIKKEI 2019本選 G - Greatest Journey (1200) - AtCoder](https://atcoder.jp/contests/nikkei2019-final/tasks/nikkei2019_final_g){:target="_blank"}<!--_-->
* [みんプロ2018 決勝 C - 木の問題 (1200) - AtCoder](https://atcoder.jp/contests/yahoo-procon2018-final-open/tasks/yahoo_procon2018_final_c){:target="_blank"}<!--_-->

# 参考

* [木の重心列挙アルゴリズム - Learning Algorithms](http://www.learning-algorithms.com/entry/2018/01/03/215559){:target="_blank"}<!--_-->
* [重心分解による分割統治法の一般形について - Learning Algorithms](http://www.learning-algorithms.com/entry/2018/01/20/031005){:target="_blank"}<!--_-->
* [ツリーの重心分解 (木の重心分解) の図解 - Qiita (@drken)](https://qiita.com/drken/items/4b4c3f1824339b090202){:target="_blank"}<!--_-->
* [木の重心分解(Centroid-Decomposition) - ei1333.github.io/luzhiled](https://ei1333.github.io/luzhiled/snippets/tree/centroid-decomposition.html){:target="_blank"}<!--_-->

