---
layout: lib
title: クエリ平方分割 (Mo's Algorithm)
permalink: algorithm/Mo

---


Moのアルゴリズム（クエリの平方分割）

オフラインでstaticなデータに対するクエリに答えられます．

配列の長さが $N$ のとき，  
区間について答えるクエリ $Q$ 個に，  
$O(N \sqrt{Q} f(N))$ で答えることができる．

ただし，$f(N)$ とは 区間 $[l, r)$ に対する答えから，  
$[l + 1, r), [l - 1, r), [l, r + 1), [l, r - 1)$ を答える(伸縮する，と呼ぶことにする)のにかかる時間である．
<!--]-->

伸縮にかかる時間は $O(1)$ や $O(\log N)$ だが， $O(\log N)$ はTLEになるかもしれない．

"出現した種類数"，"k個以上出現するもの" とかが基本的なキーワード．(あくまでも基本なのでね)

# 参考

* [Mo's algorithm - ei1333の日記](https://ei1333.hateblo.jp/entry/2017/09/11/211011){:target="_blank"}<!--_-->
* [Mo's algorithm の上位互換の話 - あなたは嘘つきですかと聞かれたら「YES」と答えるブログ](http://snuke.hatenablog.com/entry/2016/07/01/000000){:target="_blank"}<!--_-->
  * Mo自体についてもガッツリ書かれています
  * 計算量の証明はここを参照
  * このライブラリは別のところ( [Rollback可能なデータ構造とMo]({{ "algorithm/MoEx" | absolute_url }}) )においてあります

更新ができるMoは [時空間Mo]({{ "algorithm/Mo3D" | absolute_url }})

# Moの高速化

以下で議論されていた内容など．

* [Query regarding Mo's Algorithm - CodeChef Discussion](https://discuss.codechef.com/questions/119615/query-regarding-mos-algorithm){:target="_blank"}<!--_-->

1. <span style="color:red">ソートの比較関数を変える</span>
  * ブロックのパリティによって，右端をleft to rightにするかその逆にするかを交互にすれば，右端の移動回数が1/2になります．
  * コードもほんの少し変更するだけでよく，目に見えて速くなるのでおすすめです．
  * 不適切な評価関数を使わないように気をつけてください ([詳細]({{ "trap/improper-comparison" | absolute_url }}))
2. ブロックのサイズを変える
  * これは必ずしも速くなるとは言えず，  
  試してみるしかない．係数を2,3などにする．
3. add, remのクロック数を減らす
  * これはかなり有効ですが，同じ $O(1)$ であれば，  
  想定回であれば大抵大丈夫で，余計に考察することになるかもしれません．
4. ベタ書きする
  * 問題に合わせて最適な実装にします
  * うーん，コンテスト終わった後のお楽しみ程度の認識．

ただ，これらを組み合わせると3,4倍にも高速になります．  
(なりますというか，これができるのはだいぶ変態)

# 木上のMo

詳しくはうしさんの記事を見てください．

オイラーツアーすると，部分木に対するクエリが 区間 $[in_v, out_v)$ に対するクエリに変わります．
<!--]-->

パスに対するクエリは，頂点にデータがあるときは，亜種オイラーツアーみたいなのをして区間クエリと一点（LCA）の更新に置き換えます

# 実装

区間はすべて $[l, r)$ (半開区間) です

全部に `inline` がついてて気持ち悪いという気持ちになるが，ほんとそう


```cpp
// Mo(N, JUST Q, double k)
// favored : k = 2
// 1: insert
// 2: build
/// --- Mo Library {{"{{"}}{ ///

struct Mo {
  const int width;
  int q = 0;
  vector< int > le, ri, order;
  int nl = 0, nr = 0;
  Mo(int n, int q, double k = 1)
      : width(int(k* n / sqrt(q) + 1.0)), le(q), ri(q), order(q) {}
  inline void insert(int l, int r) {
    le[q] = l;
    ri[q] = r;
    order[q] = q;
    q++;
  }
  inline void build() {
    sort(begin(order), begin(order) + q, [&](int a, int b) {
      const int ab = le[a] / width, bb = le[b] / width;
      return ab != bb ? ab < bb : ab & 1 ? ri[a] < ri[b] : ri[b] < ri[a];
    });
    nl = nr = le[order[0]];
    for(int i = 0; i < q; i++) {
      const int id = order[i];
      while(nl > le[id]) add(--nl);
      while(nl < le[id]) rem(nl++);
      while(nr < ri[id]) add(nr++);
      while(nr > ri[id]) rem(--nr);
      next(id);
    }
  }
  inline void next(int i);
  inline void add(int i);
  inline void rem(int i);
};

/// }}}--- ///

inline void Mo::next(int i) {}
inline void Mo::add(int i) {}
inline void Mo::rem(int i) {}
```


## 木上の頂点にデータがあるパスクエリMo

うしさんの解説を見てください


```cpp
// MoTreeVertex(N, JUST Q, double k)
// favored : k = 2
// 1: addEdge
// 2: prebuild
// 3: insert
// 4: build
/// --- MoTreeVertex Library {{"{{"}}{ ///

struct MoTreeVertex {
  const int n, logn, m;
  const int width;
  int q = 0;
  vector< vector< int > > par;
  vector< int > dep;
  vector< int > in, vs;
  vector< vector< int > > g;
  vector< int8_t > flag;
  vector< int > le, ri, lcas, order;
  int nl = 0, nr = 0;
  int log(int x) {
    int h = 1;
    while((1 << h) < x) h++;
    return h;
  }
  MoTreeVertex(int n, int q, double k = 1)
      : n(n),
        logn(log(n)),
        m(2 * n - 1),
        width(int(k* m / sqrt(q) + 1.0)),
        q(q),
        par(logn, vector< int >(n, -1)),
        dep(n),
        in(n),
        g(n),
        flag(n),
        le(q),
        ri(q),
        lcas(q),
        order(q) {
    vs.reserve(m);
  }
  inline void addEdge(int u, int v) {
    g[u].emplace_back(v);
    g[v].emplace_back(u);
  }
  inline void prebuild() {
    dfs(0, -1, 0);
    for(int k = 1; k < logn; k++)
      for(int i = 0; i < n; i++) {
        int p = par[k - 1][i];
        if(p == -1) continue;
        par[k][i] = par[k - 1][p];
      }
  }
  void dfs(int i, int p, int d) {
    dep[i] = d;
    par[0][i] = p;
    in[i] = vs.size();
    vs.emplace_back(i);
    for(int j : g[i])
      if(j != p) {
        dfs(j, i, d + 1);
        vs.emplace_back(j);
      }
  }
  inline int lca(int u, int v) {
    if(dep[u] > dep[v]) swap(u, v);
    for(int k = logn - 1; k >= 0; k--) {
      int nv = par[k][v];
      if(nv != -1 && dep[nv] >= dep[u]) v = nv;
    }
    if(u == v) return u;
    for(int k = logn - 1; k >= 0; k--) {
      int nu = par[k][u], nv = par[k][v];
      if(nu != nv) u = nu, v = nv;
    }
    return par[0][u];
  }
  inline void insert(int u, int v) {
    if(in[u] > in[v]) swap(u, v);
    le[q] = in[u] + 1;
    ri[q] = in[v] + 1;
    lcas[q] = lca(u, v);
    order[q] = q;
    q++;
  }
  inline void build() {
    sort(begin(order), begin(order) + q, [&](int a, int b) {
      const int ab = le[a] / width, bb = le[b] / width;
      return ab != bb ? ab < bb : ab & 1 ? ri[a] < ri[b] : ri[b] < ri[a];
    });
    nl = nr = le[order[0]];
    for(int i = 0; i < q; i++) {
      if(i > 0) rem(lcas[order[i - 1]]);
      const int id = order[i];
      while(nl > le[id]) flip(vs[--nl]);
      while(nr < ri[id]) flip(vs[nr++]);
      while(nl < le[id]) flip(vs[nl++]);
      while(nr > ri[id]) flip(vs[--nr]);
      add(lcas[id]);
      next(id);
    }
  }
  inline void flip(int i) {
    if(flag[i] ^= 1)
      add(i);
    else
      rem(i);
  }
  inline void next(int id);
  inline void add(int i);
  inline void rem(int i);
};

/// }}}--- ///

inline void MoTreeVertex::next(int id) {}
inline void MoTreeVertex::add(int i) {}
inline void MoTreeVertex::rem(int i) {}
```


# 注意点

比較関数が `operator<` 相当ではなく `operator<=` 相当にならないように気をつけてください．(実装を参照) ([詳細]({{ "trap/improper-comparison" | absolute_url }}))

# 検証

* [D - Powerful array - CF (4334ms)](https://codeforces.com/contest/86/submission/42720275){:target="_blank"}<!--_-->
  * [meooowさんの高速化された1058ms解](https://codeforces.com/contest/86/submission/33239378){:target="_blank"}<!--_-->
* 木上のMo - [D - Tree and Queries - CF](https://codeforces.com/contest/375/submission/42721054){:target="_blank"}<!--_-->
  * $N \leq 10^5$ はだいぶ安心してMoを投げられます
* 木上の頂点にデータがあるパスクエリMo - [COT2 - Count on a tree II - SPOJ](https://www.spoj.com/files/src/22296870/){:target="_blank"}<!--_-->
* 同上 - [The L-th Number - AOJ](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/2270/judge/3127075/C++14){:target="_blank"}<!--_-->

# 練習

* [D - Powerful array - CF](https://codeforces.com/contest/86/problem/D){:target="_blank"}<!--_-->
* [D - Tree and Queries - CF](https://codeforces.com/contest/375/problem/D){:target="_blank"}<!--_-->
* [COT2 - Count on a tree II - SPOJ](https://www.spoj.com/problems/COT2/){:target="_blank"}<!--_-->
* [The L-th Number - AOJ](https://onlinejudge.u-aizu.ac.jp/problems/2270){:target="_blank"}<!--_-->


