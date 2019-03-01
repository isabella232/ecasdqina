---
layout: lib
title: Link/Cut Tree
permalink: data-structure/misc/LinkCutTree

---


**森** (もしくは **根付き有向木 (Arborescence) の集合**) への更新とクエリを順不同で高速に

根の変更クエリ(evert)には乗せるモノイドに対する要求があり，その条件は平衡二分探索木がreverseで要求するものと同じ([モノイド]({{ "math/Monoid" | absolute_url }})を参照)

IOIで常勝できることはあまりにも有名

参考になるものを準備しています

TODO : 区間に対する更新に対し，leftを求めるの，ちょっとどうすればいいかわからないけど，やっている人がいるのでそのうち見てみようと思います


```cpp
// do make(index, Monoid::T value)
// link(p, c) : c is root
// cut(c) : c is not root
// same(a, b)
// node->id
// lc[index] to access nodes
/// --- LinkCutTree {{"{{"}}{ ///
#include <cstdlib>
#include <vector>
template < class M_act >
struct LinkCutTree {
  using Monoid = typename M_act::Monoid;
  using X = typename Monoid::T;
  using M = typename M_act::M;

  // Splay sequence {{"{{"}}{
  struct Splay {
    int id;
    Splay *ch[2] = {nullptr, nullptr}, *p = nullptr;
    X val, accum;
    M lazy = M_act::identity(); ///////
    // size of BST // not of real subtree
    int sz = 1;
    bool isRoot() { return !p || (p->ch[0] != this && p->ch[1] != this); }
    bool rev = false;
    // call before use
    void eval() {
      if(lazy != M_act::identity()) {
        val = M_act::actInto(lazy, 1, val);
        accum = M_act::actInto(lazy, sz, accum);
        if(ch[0]) ch[0]->lazy = M_act::op(lazy, ch[0]->lazy);
        if(ch[1]) ch[1]->lazy = M_act::op(lazy, ch[1]->lazy);
        lazy = M_act::identity();
      }
      if(rev) {
        swap(ch[0], ch[1]);
        if(ch[0]) ch[0]->rev ^= 1;
        if(ch[1]) ch[1]->rev ^= 1;
        // accum = reverse(accum, sz)
        rev = false;
      }
    }
    void evalDown() {
      vector< Splay * > b2t;
      Splay *t = this;
      for(; !t->isRoot(); t = t->p) b2t.emplace_back(t);
      t->eval();
      while(b2t.size()) b2t.back()->eval(), b2t.pop_back();
    }
    X accumulated(Splay *a) { return !a ? Monoid::identity() : a->accum; }
    int size(Splay *a) { return !a ? 0 : a->sz; }
    // call after touch
    void prop() {
      if(ch[0]) ch[0]->eval();
      if(ch[1]) ch[1]->eval();
      sz = size(ch[0]) + 1 + size(ch[1]);
      accum = Monoid::op(Monoid::op(accumulated(ch[0]), val), accumulated(ch[1]));
    }
    Splay(const X &val, int id) : id(id), val(val), accum(val) {}
    Splay *rotate(bool R) {
      Splay *t = ch[!R];
      if((ch[!R] = t->ch[R])) ch[!R]->p = this;
      t->ch[R] = this;
      if((t->p = p)) {
        if(t->p->ch[0] == this) t->p->ch[0] = t;
        if(t->p->ch[1] == this) t->p->ch[1] = t;
      }
      p = t;
      prop(), t->prop();
      return t;
    }
    // bottom-up
    void splay() {
      evalDown();
      while(!isRoot()) {
        Splay *q = p;
        if(q->isRoot()) {
          q->rotate(q->ch[0] == this);
        } else {
          Splay *r = q->p;
          bool V = r->ch[0] == q;
          if(q->ch[!V] == this)
            r->rotate(V);
          else
            q->rotate(!V);
          p->rotate(V);
        }
      }
    }
  };
  // }}}

  Splay *pool;
  LinkCutTree(int n) { pool = (Splay *) malloc(sizeof(Splay) * n); }
  ~LinkCutTree() { free(pool); }
  Splay *operator[](int i) const { return pool + i; }
  Splay *make(int i, const X &x = Monoid::identity()) {
    return new(pool + i) Splay(x, i);
  }
  const X &get(Splay *x) {
    x->splay();
    return x->val;
  }
  void set(Splay *x, const X &val) {
    x->splay();
    x->val = val;
    x->prop();
  }
  Splay *expose(Splay *x) {
    Splay *prv = nullptr, *now = x;
    for(; now; prv = now, now = now->p) {
      now->splay();
      now->ch[1] = prv;
      now->prop();
    }
    x->splay();
    return prv;
  }
  void cut(Splay *c) {
    expose(c);
#ifdef DEBUG
    static const struct CannotCutRoot {} ex;
    if(!c->ch[0]) throw ex;
#endif
    Splay *s = c->ch[0];
    c->ch[0] = nullptr;
    c->prop();
    s->p = nullptr;
  }
  void link(Splay *parent, Splay *child) {
#ifdef DEBUG
    static const struct CannotLinkSameNode {} ex;
    if(same(parent, child)) throw ex;
#endif
    expose(parent), expose(child);
    child->p = parent;
    parent->ch[1] = child;
    parent->prop();
  }
  void evert(Splay *x) {
    expose(x);
    x->rev = true;
  }
  bool same(Splay *a, Splay *b) {
    if(a == b) return true;
    expose(a), expose(b);
    return a->p != nullptr;
  }
  Splay *lca(Splay *a, Splay *b) {
#ifdef DEBUG
    static const struct CannotLCADifferentNode {} ex;
    if(!same(a, b)) throw ex;
#endif
    expose(a), a = expose(b);
    return !a ? b : a;
  }
  void act(Splay *a, const M &m) { expose(a), a->lazy = m; }
  X query(Splay *a) {
    expose(a);
    return a->accum;
  }
  // root of subtree
  Splay *getRoot(Splay *a) {
    expose(a);
    Splay *t = a;
    while(t->ch[0]) t = t->ch[0];
    t->splay();
    return t;
  }
};

/// }}}--- ///

/// --- Monoid examples {{"{{"}}{ ///
constexpr long long inf_monoid = 1e18 + 100;
#include <algorithm>
struct Nothing {
  using T = char;
  using Monoid = Nothing;
  using M = T;
  static constexpr T op(const T &, const T &) { return T(); }
  static constexpr T identity() { return T(); }
  template < class X >
  static constexpr X actInto(const M &, ll, ll, const X &x) {
    return x;
  }
};

template < class U = long long >
struct RangeMin {
  using T = U;
  static T op(const T &a, const T &b) { return min(a, b); }
  static constexpr T identity() { return T(inf_monoid); }
};

template < class U = long long >
struct RangeMax {
  using T = U;
  static T op(const T &a, const T &b) { return max(a, b); }
  static constexpr T identity() { return -T(inf_monoid); }
};

template < class U = long long >
struct RangeSum {
  using T = U;
  static T op(const T &a, const T &b) { return a + b; }
  static constexpr T identity() { return T(0); }
};

template < class U >
struct RangeProd {
  using T = U;
  static T op(const T &a, const T &b) { return a * b; }
  static constexpr T identity() { return T(1); }
};

template < class U = long long >
struct RangeOr {
  using T = U;
  static T op(const T &a, const T &b) { return a | b; }
  static constexpr T identity() { return T(0); }
};

#include <bitset>

template < class U = long long >
struct RangeAnd {
  using T = U;
  static T op(const T &a, const T &b) { return a & b; }
  static constexpr T identity() { return T(-1); }
};

template < size_t N >
struct RangeAnd< bitset< N > > {
  using T = bitset< N >;
  static T op(const T &a, const T &b) { return a & b; }
  static constexpr T identity() { return bitset< N >().set(); }
};

/// }}}--- ///

/// --- M_act examples {{"{{"}}{ ///
template < class U = long long, class V = U >
struct RangeMinAdd {
  using X = U;
  using M = V;
  using Monoid = RangeMin< U >;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, ll, const X &x) { return m + x; }
};

template < class U = long long, class V = U >
struct RangeMaxAdd {
  using X = U;
  using M = V;
  using Monoid = RangeMax< U >;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, ll, const X &x) { return m + x; }
};

template < class U = long long, class V = U >
struct RangeMinSet {
  using M = U;
  using Monoid = RangeMin< U >;
  using X = typename Monoid::T;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return -M(inf_monoid); }
  static X actInto(const M &m, ll, const X &) { return m; }
};

template < class U = long long, class V = U >
struct RangeMaxSet {
  using M = U;
  using Monoid = RangeMax< U >;
  using X = typename Monoid::T;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return -M(inf_monoid); }
  static X actInto(const M &m, ll, const X &) { return m; }
};

template < class U = long long, class V = U >
struct RangeSumAdd {
  using X = U;
  using M = V;
  using Monoid = RangeSum< U >;
  static M op(const M &a, const M &b) { return a + b; }
  static constexpr M identity() { return 0; }
  static X actInto(const M &m, ll n, const X &x) { return m * n + x; }
};

template < class U = long long, class V = U >
struct RangeSumSet {
  using X = U;
  using M = V;
  using Monoid = RangeSum< U >;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return -M(inf_monoid); }
  static X actInto(const M &m, ll n, const X &) { return m * n; }
};

template < class U, class V = U >
struct RangeProdMul {
  using X = U;
  using M = V;
  using Monoid = RangeProd< U >;
  static M mpow(M a, ll b) {
    X r(1);
    while(b) {
      if(b & 1) r = r * a;
      a = a * a;
      b >>= 1;
    }
    return r;
  }
  static M op(const M &a, const M &b) { return a * b; }
  static constexpr M identity() { return M(1); }
  static X actInto(const M &m, ll n, const X &x) { return x * mpow(m, n); }
};

template < class U, class V = U >
struct RangeProdSet {
  using X = U;
  using M = V;
  using Monoid = RangeProd< U >;
  static M op(const M &a, const M &) { return a; }
  static constexpr M identity() { return V::unused; }
  static X actInto(const M &m, ll n, const X &) {
    return RangeProdMul< U, V >::mpow(m, n);
  }
};

template < class U = long long, class V = U >
struct RangeOr2 {
  using X = U;
  using M = V;
  using Monoid = RangeOr< U >;
  static M op(const M &a, const M &b) { return a | b; }
  static constexpr M identity() { return M(0); }
  static X actInto(const M &m, ll, const X &x) { return m | x; }
};

template < class U = long long, class V = U >
struct RangeAnd2 {
  using X = U;
  using M = V;
  using Monoid = RangeAnd< U >;
  static M op(const M &a, const M &b) { return a & b; }
  static constexpr M identity() { return M(-1); }
  static X actInto(const M &m, ll, const X &x) { return m & x; }
};

template < class U, size_t N >
struct RangeAnd2< U, bitset< N > > {
  using X = U;
  using M = bitset< N >;
  using Monoid = RangeAnd< U >;
  static M op(const M &a, const M &b) { return a & b; }
  static constexpr M identity() { return bitset< N >().set(); }
  static X actInto(const M &m, ll, const X &x) { return m & x; }
};
/// }}}--- ///

using LCT = LinkCutTree< RangeSumAdd<> >;

LCT lc(N);
```


# 検証

* RMQとRUQ - [AOJのなんか](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/DSL_2_F/judge/3092002/C++14){:target="_blank"}<!--_-->
  * 遅延セグ木でできることがevert付きでできる
* LCA - [AOJのなんか](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/GRL_5_C/judge/3092319/C++14){:target="_blank"}<!--_-->
* HL-Decomp(TLE) [E. The Number Games - CF](https://codeforces.com/contest/980/submission/41594330){:target="_blank"}<!--_-->
* HL-Decomp [PCKの問題 - AOJ](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/0367/judge/3093506/C++14){:target="_blank"}<!--_-->
* 部分木サイズ，link/cut，部分木root [E - Black Cats Deployment - AtCoder](https://beta.atcoder.jp/contests/cf17-tournament-round3-open/submissions/3128272){:target="_blank"}<!--_-->
  * なんかいい感じのLCT向け問題を見つけてしまった (想定解ではない)
  * cutができて，サイズが分かるUnionFind的な扱いをする
  * evertがないとき，部分木のサイズを計算できる
  * (cutの後にlinkがあるとかは，部分木のサイズは対応が難しいかも)
  * 部分木のroot取得もある

# 練習

* [JOI春13-day4 - 3 - 宇宙船 (Spaceships) - AtCoder](https://beta.atcoder.jp/contests/joisc2013-day4/tasks/joisc2013_spaceships){:target="_blank"}<!--_-->
* [E - Black Cats Deployment - AtCoder](https://beta.atcoder.jp/contests/cf17-tournament-round3-open/tasks/asaporo2_e){:target="_blank"}<!--_-->

# 参照

* [プログラミングコンテストでのデータ構造 2 ～動的木編～ - SlideShare](https://www.slideshare.net/iwiwi/2-12188845){:target="_blank"}<!--_-->
* [Spaceships 解説 - SlideShare](https://www.slideshare.net/qnighy/2013-spaceships2){:target="_blank"}<!--_-->
  * SplayやLCTのポテンシャルを用いた計算量の評価と証明が書かれています

