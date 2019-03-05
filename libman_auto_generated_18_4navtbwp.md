---
layout: lib
title: 赤黒木 (Red-Black Tree; RBT)
permalink: data-structure/BBST/Red-Black-Tree

---


[BBST]({{ "data-structure/BBST" | absolute_url }}) のひとつ

# Sequenceタイプ

遅延セグ木にできることが基本できて，reverseも条件付きでできる

永続化はコメントを切り替えるだけ (デフォルトで永続化)


```cpp
constexpr int FREE_TYPE = 0; // 0 : nothing, 1 : free, 2 : destroy
constexpr int POOL_SIZE = 1.1e7;

// <BBST Basic Funcs> ( node [, ...] )
// build ( node, first, last )
// Node::rebuildCheck ( <nodes>, threshold = 1e4 )
// mfree( node )
// [Persistent] clone ( node [, <range> ] )
/// --- Red-Black Tree Sequence {{"{{"}}{ ///
#include <cassert>
#include <cstdint>
#include <numeric>
#include <tuple>
#include <utility>
#include <vector>

namespace RBT {

// MemoryPool {{"{{"}}{

template < int POOL_SIZE >
struct MemoryPool {
  template < class T >
  struct Core {
  private:
    vector< T * > ptr;
    T *start;
    uint_fast32_t idx = 0;

  public:
    Core() : ptr(POOL_SIZE) { expand(); }
    void expand() {
      start = (T *) malloc(sizeof(T) * POOL_SIZE);
      iota(ptr.begin(), ptr.end(), start);
    }
    void clear() {
      idx = 0;
      if(FREE_TYPE) iota(ptr.begin(), ptr.end(), start);
    }
    T *alloc() { return ptr[idx++]; }
    void free(T *p) {
      assert(FREE_TYPE);
      if(p) ptr[--idx] = p;
    }
    void destroy(T *p) {
      assert(FREE_TYPE);
      if(p) {
        p->~T();
        free(p);
      }
    }
    const T *next() { return ptr[idx]; }
    int capacity() const { return ptr.size(); }
    int used() const { return idx; }
    int rest() const { return ptr.size() - idx; }
  };
};

// }}}

// RBTSeq Base {{"{{"}}{

template < class M_act, bool isPersistent, template < class > class Allocator >
struct RedBlackTreeSequenceBase {
private:
  using Node = RedBlackTreeSequenceBase;

public:
  static Allocator< Node > alc;
  using Monoid = typename M_act::Monoid;
  using X = typename Monoid::T;
  using M = typename M_act::M;

private:
  Node *c[2];
  X accum = Monoid::identity();
  M lazy = M_act::identity();
  uint_fast8_t rev = false;
  int sz;
  enum Color { BLACK, RED };
  uint_fast8_t color;
  uint_fast8_t level;

private:
  // leaf node
  RedBlackTreeSequenceBase(X val = Monoid::identity())
      : c{0, 0}, accum(val), sz(1), color(BLACK), level(0) {}
  // internal node
  RedBlackTreeSequenceBase(Node *l, Node *r, Color color) : c{l, r}, color(color) {
    prop(this);
  }
  static Node *make(Node *l, Node *r, Color color) {
    // assert(l && r && l != alc.next() && r != alc.next()); // explicit
    return new(alc.alloc()) Node(l, r, color);
  }
  static Node *make(const X &x) { return new(alc.alloc()) Node(x); }

  // prop and eval {{"{{"}}{

private:
  // [RBT] a is internal node
  // a is evaled
  // a->c[0], c[1] is proped, evaled
  friend Node *prop(Node *a) {
    a->sz = a->c[0]->sz + a->c[1]->sz;
    a->accum = Monoid::op(a->c[0]->accum, a->c[1]->accum);
    // assert(a->c[0]->level + (a->c[0]->color == BLACK) ==
    //        a->c[1]->level + (a->c[1]->color == BLACK)); // explicit
    a->level = a->c[0]->level + (a->c[0]->color == BLACK);
    return a;
  }
  // call before use val, accum
  friend Node *eval(Node *a) {
    if(a->lazy != M_act::identity()) {
      a->accum = M_act::actInto(a->lazy, a->sz, a->accum);
      for(int i = 0; i < 2; i++)
        if(a->c[i]) {
          a->c[i] = copy(a->c[i]);
          a->c[i]->lazy = M_act::op(a->lazy, a->c[i]->lazy);
        }
      a->lazy = M_act::identity();
    }
    if(a->rev) {
      swap(a->c[0], a->c[1]);
      for(int i = 0; i < 2; i++)
        if(a->c[i]) {
          a->c[i] = copy(a->c[i]);
          a->c[i]->rev ^= 1;
        }
      a->rev = false;
    }
    return a;
  }

  // }}}

  // BBST {{"{{"}}{
public:
  friend void mfree(Node *a) {
    if(FREE_TYPE == 1)
      alc.free(a);
    else if(FREE_TYPE == 2)
      alc.destroy(a);
  }
  friend void push_front(Node *&a, const X &x) { a = merge(make(x), a); }
  friend void push_back(Node *&a, const X &x) { a = merge(a, make(x)); }
  friend void insert(Node *&a, int k, const X &x) { insert(a, k, make(x)); }
  friend void insert(Node *&a, int k, Node *b) {
    Node *sl, *sr;
    tie(sl, sr) = split(a, k);
    a = merge(sl, merge(b, sr));
  }
  friend X pop_front(Node *&a) {
    Node *sl, *sr;
    tie(sl, sr) = split(a, 1);
    a = sr;
    X x = getValue(sl);
    mfree(sl);
    return x;
  }
  friend X pop_back(Node *&a) {
    Node *sl, *sr;
    tie(sl, sr) = split(a, size(a) - 1);
    a = sl;
    X x = getValue(sr);
    mfree(sr);
    return x;
  }
  friend X erase(Node *&a, int k) {
    Node *x, *y, *z;
    tie(x, y, z) = split(a, k, k + 1);
    X v = getValue(y);
    mfree(y);
    a = merge(x, z);
    return v;
  }
  friend void erase(Node *&a, int l, int r) {
    Node *x, *y, *z;
    tie(x, y, z) = split(a, l, r);
    mfree(y);
    a = merge(x, z);
  }
  friend tuple< Node *, Node *, Node * > split(Node *a, int l, int r) {
    Node *sl, *sr, *tl, *tr;
    tie(sl, sr) = split(a, r);
    tie(tl, tr) = split(sl, l);
    return make_tuple(tl, tr, sr);
  }
  friend void mset(Node *&a, int k, const X &v) {
    if(!a) return;
    if(k < 0 || a->sz <= k) return;
    a = eval(a);
    if(isLeaf(a))
      a->accum = v;
    else if(k < a->c[0]->sz)
      mset(a->c[0], k, v);
    else
      mset(a->c[1], k - a->c[0]->sz, v);
    a = prop(a);
  }
  friend X getValue(const Node *a) {
    assert(isLeaf(a));
    return a ? a->lazy != M_act::identity() ? M_act::actInto(a->accum, 1, a->lazy)
                                            : a->accum
             : Monoid::identity();
  }
  friend vector< X > getAll(const Node *a) {
    vector< X > v(size(a));
    auto ite = v.begin();
    getdfs(a, M_act::identity(), ite);
    return v;
  }

  friend X mget(const Node *a, int k) {
    if(!a) return Monoid::identity();
    if(isLeaf(a)) return k == 0 ? getValue(a) : Monoid::identity();
    return k < a->c[0]->sz ? mget(a->c[0], k) : mget(a->c[1], k - a->c[0]->sz);
  }
  friend void act(Node *&a, int l, int r, const M &m) {
    Node *x, *y, *z;
    tie(x, y, z) = split(a, l, r);
    if(y) y->lazy = M_act::op(m, y->lazy);
    a = merge(merge(x, y), z);
  }
  friend X fold(Node *a) { return a ? (eval(a), a->accum) : Monoid::identity(); }
  friend X fold(Node *&a, int l, int r) {
    Node *x, *y, *z;
    tie(x, y, z) = split(a, l, r);
    X res = fold(y);
    a = merge(merge(x, y), z);
    return res;
  }
  friend void reverse(Node *a) {
    if(a) a->rev ^= 1;
  }
  friend void reverse(Node *&a, int l, int r) {
    Node *x, *y, *z;
    tie(x, y, z) = split(a, l, r);
    if(y) y->rev ^= 1;
    a = merge(merge(x, y), z);
  }
  friend int size(const Node *a) { return a ? a->sz : 0; }
  /// }}}--- ///

private:
  static Node *copy(Node *a) {
    // assert(a); // explicit
    if(isPersistent) {
      return new(alc.alloc()) Node(*a);
    } else {
      return a;
    }
  }

  // [RBT] rotate, submerge {{"{{"}}{
private:
  // [RBT]
  // a is evaled, NOT proped
  // a->c[_] is unique, evaled;
  // a->c[!R] is internal node
  // a->c[!R]->c[!R] is unique, evaled
  friend Node *rotate(Node *a, bool R) {
    Node *l = a->c[!R];
    a->c[!R] = eval(copy(l->c[R]));
    l->c[R] = prop(a);
    return prop(l);
  }
  // [RBT]
  // a is internal node, evaled, NOT proped
  // a->[_] is unique, evaled
  // return proped, evaled
  friend Node *check(Node *a, bool R) {
    if(a->color == BLACK && a->c[!R]->color == RED && a->c[!R]->c[!R]->color == RED) {
      // assert(a->c[!R]->c[R]->color == BLACK); // explicit
      a->color = RED;
      a->c[!R]->color = BLACK;
      if(a->c[R]->color == BLACK) return rotate(a, R);
      a->c[R]->color = BLACK;
    }
    return prop(a);
  }
  // return proped, evaled
  friend Node *submerge(Node *a, Node *b) {
    if(a->level < b->level) {
      // b is NOT leaf node
      // assert(b->c[0] && b->c[1]); // explicit
      b = eval(b);
      b->c[0] = submerge(a, copy(b->c[0]));
      b->c[1] = eval(copy(b->c[1]));
      return check(b, 1);
    } else if(a->level > b->level) {
      // a is NOT leaf node
      // assert(a->c[0] && a->c[1]); // explicit
      a = eval(a);
      a->c[1] = submerge(copy(a->c[1]), b);
      a->c[0] = eval(copy(a->c[0]));
      return check(a, 0);
    } else {
      // assert(a->color == BLACK && b->color == BLACK); // explicit
      return make(eval(a), eval(b), RED);
    }
  }
  // }}}

  // [RBT] {{"{{"}}{
public:
  // return evaled
  friend Node *merge(Node *a, Node *b) {
    if(!a) return b;
    if(!b) return a;
    Node *c = submerge(a, b);
    c->color = BLACK;
    return c;
  }

  // [0, k), [k, n)
  friend pair< Node *, Node * > split(Node *a, int k) {
    if(!a) return {0, 0};
    if(k == 0) return {0, a};
    if(k >= a->sz) return {a, 0};
    a = eval(a);
    // a is internal
    // assert(a->c[0] && a->c[1]); // explicit

    Node *sl, *sr;
    Node *l = copy(a->c[0]), *r = copy(a->c[1]);
    mfree(a);
    if(k < l->sz) {
      tie(sl, sr) = split(l, k);
      return {sl, merge(sr, r)};
    }
    if(k > l->sz) {
      tie(sl, sr) = split(r, k - l->sz);
      return {merge(l, sl), sr};
    }
    l->color = r->color = BLACK;
    return {l, r};
  }
  friend Node *clone(Node *a) {
    assert(isPersistent);
    if(!a) return 0;
    return new(alc.alloc()) Node(*a);
  }
  friend Node *clone(Node *&a, int l, int r) {
    assert(isPersistent);
    if(!a) return 0;
    Node *x, *y, *z;
    tie(x, y, z) = split(a, l, r);
    Node *res = clone(y);
    a = merge(merge(x, y), z);
    return res;
  }

  // it must be guaranteed that there's enough memory in pool
  template < class InputIter, class = typename iterator_traits< InputIter >::value_type >
  friend Node *build(Node *&a, InputIter first, InputIter last) {
    mfree(a);
    int n = distance(first, last);
    if(n == 0) return 0;
    vector< Node * > v(n), tmp(n);
    InputIter ite = first;
    for(int i = 0; i < n; ++i, ++ite) v[i] = make(*ite);
    while(v.size() != 1) {
      tmp.resize(v.size() >> 1);
      for(size_t i = 0; i < tmp.size(); i++) tmp[i] = merge(v[i << 1], v[(i << 1) | 1]);
      if(v.size() & 1) tmp.emplace_back(v.back());
      swap(v, tmp);
    }
    return a = v[0];
  }

  friend inline bool isLeaf(const Node *a) { return !bool(a->c[0]); }

private:
  static void getdfs(const Node *a, M m, typename vector< X >::iterator &ite) {
    if(a->lazy != M_act::identity()) m = M_act::op(m, a->lazy);
    if(isLeaf(a)) {
      *ite++ = m != M_act::identity() ? M_act::actInto(a->accum, 1, m) : a->accum;
    } else {
      getdfs(a->c[0], m, ite);
      getdfs(a->c[1], m, ite);
    }
  }

public:
  static void rebuild(vector< Node * > &roots) {
    vector< vector< X > > vals(roots.size());
    for(size_t i = 0; i < roots.size(); i++) {
      vals[i] = (getAll(roots[i]));
    }
    alc.clear();
    for(size_t i = 0; i < roots.size(); i++) {
      build(roots[i] = 0, vals[i].begin(), vals[i].end());
    }
  }

  static inline void rebuildCheck(Node *&a, int threshold = 10000) {
    if(Node::alc.rest() < threshold) {
      vector< Node * > v{a};
      Node::rebuild(v);
      a = v[0];
    }
  }

  static inline void rebuildCheck(vector< Node * > &v, int threshold = 10000) {
    if(Node::alc.rest() < threshold) Node::rebuild(v);
  }

  friend string to_string(const Node *a) {
    if(!a) return "";
    return "(" + to_string(a->c[0]) +          //
           ", accum: " + to_string(a->accum) + //
           // " isBlack: " + to_string(a->color == BLACK) + //
           " sz: " + to_string(a->sz) + //
           ", " + to_string(a->c[1]) + ")";
  }

  // }}}
};

template < class M_act, bool isPersistent, template < class > class Allocator >
Allocator< RedBlackTreeSequenceBase< M_act, isPersistent, Allocator > >
    RedBlackTreeSequenceBase< M_act, isPersistent, Allocator >::alc;

// }}}

template < class M_act >
using RBTSeq = RedBlackTreeSequenceBase< M_act, false, MemoryPool< POOL_SIZE >::Core >;

template < class M_act >
using PersistentRBTSeq =
    RedBlackTreeSequenceBase< M_act, true, MemoryPool< POOL_SIZE >::Core >;

} // namespace RBT

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
  static constexpr X actInto(const M &, long long, const X &x) {
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
struct RangeAnd< std::bitset< N > > {
  using T = std::bitset< N >;
  static T op(const T &a, const T &b) { return a & b; }
  static constexpr T identity() { return std::bitset< N >().set(); }
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
struct RangeAnd2< U, std::bitset< N > > {
  using X = U;
  using M = std::bitset< N >;
  using Monoid = RangeAnd< U >;
  static M op(const M &a, const M &b) { return a & b; }
  static constexpr M identity() { return std::bitset< N >().set(); }
  static X actInto(const M &m, ll, const X &x) { return m & x; }
};
/// }}}--- ///

using RBT::PersistentRBTSeq;
using RBT::RBTSeq;

// using Node = RBTSeq< RangeSumAdd<> >; // FREE_TYPE = 1
using Node = PersistentRBTSeq< RangeSumAdd<> >; // FREE_TYPE = 0

Node *seq = 0;
```


## 使い方

`操作(obj, ...)` の順序．BBST共通の関数は [BBST]({{ "data-structure/BBST" | absolute_url }}) を参照

`build( Node*&, first, last)` で線形時間で構築

`Node::rebuildCheck( <node>, threshold = 1e4)` でプールの残りがしきい値を下回ったら `rebuild` してくれる．`<node>`は `Node*&` か `vector<Node*>&`

`mfree( Node* )` でノードのメモリを解法 (メモリプールに返却)

永続時は `clone( Node*& )` でノードを複製できる

## メモ

赤黒木そのものについては参考に貼ったスライドがわかりやすいのでそちらを参照してほしい

Persistentとそうでないものを一緒にした

生ポインタは基本的に `unique_ptr` と同等で，所有権ごと渡していると考えていい

関数を抜けた後は全てのノードは `proped` が保証されるが `evaled` は保証されない (用語については[BBST]({{ "data-structure/BBST" | absolute_url }}))

(永続化の場合) 子どもに対するポインタは基本 `shared_ptr` であると考え， `copy` したら `unique` であると考える．永続化でなければずっと `unique`

`merge` の計算量は2つの `rank` (実装内では `level` ) 差に対し線形時間となる．他の関数もだいたい同じで， `split` も自明ではないものの同様のことが言える．`copy` の回数は `rank` 差を $L$ とおくと， $50L$ ぐらいが目安だと感じた (数命令1セットと考えて実装では大きめに `1e4` をデフォルト値にしている)

どのような実装でも，メモリプールがないと速度面で競プロで使いにくいと思う

### shared_ptrをつかう版

メモリプールは使うとして，`shared_ptr` を使うか使わないか選ぶ余地がある

[my_shared_ptrを使った D - グラフではない への提出](https://beta.atcoder.jp/contests/arc030/submissions/3628654){:target="_blank"}<!--_--> (約4s)

使う場合は `free` されるノードを `destroy` することでメモリを必要な分だけ賄える

これは採用せず，`rebuild` 型のみにした (ちなみに**ぶっ壊し型**と呼んでいる)

### 永続RBSTについて

全体の提出を見てみると，最速帯は [RBST]({{ "data-structure/BBST/RBST" | absolute_url }}) というBBSTを永続化した**ぶっ壊し型**のものが多い

だが，[RBSTはコピー可能は嘘 - よすぽの日記](http://yosupo.hatenablog.com/entry/2015/10/30/115910){:target="_blank"}<!--_--> や [永続 RBST を撃墜するケース - mitaki28.info blog](http://blog.mitaki28.info/1446205599273/){:target="_blank"}<!--_--> などで紹介されているように，実装と入力が相まって遅くなる可能性がある

それでもぶっ壊し型は強いようだし，ライブラリ禁止下ではRBSTを書きたくなるかもしれないが，hack, systesのあるコンテストなどでは控えたほうがいいかもしれない

多くのコンテストが，想定が永続BBSTならRBTでも大丈夫なTLになっているはずだ

## 検証

* BBST - [E - Hash Swapping (800) - AtCoder](https://beta.atcoder.jp/contests/soundhound2018-summer-final-open/submissions/3632794){:target="_blank"}<!--_-->
* 永続 - [D - グラフではない (ARC) - AtCoder](https://beta.atcoder.jp/contests/arc030/submissions/3632475){:target="_blank"}<!--_-->
  * 約1400ms
* 永続 - [JOI春 - コピー＆ペースト - AtCoder](https://beta.atcoder.jp/contests/joisc2012/submissions/3632712){:target="_blank"}<!--_-->

# Multisetタイプ

書いてないよ

# 参考

* [PDF: アルゴリズムとデータ構造 2-3-4木](http://web-ext.u-aizu.ac.jp/~qf-zhao/TEACHING/Alg-1/lec11-1.pdf){:target="_blank"}<!--_-->
  * とてもわかり易いです
* [PDF: 赤黒木](https://www.ioi-jp.org/camp/2012/2012-sp-tasks/2012-sp-day4-copypaste-slides.pdf){:target="_blank"}<!--_-->
  * JOI春 - コピー＆ペースト の解説スライド
* [平衡二分探索木(Red-Black-Tree) - Luzhiled's memo](https://ei1333.github.io/luzhiled/snippets/structure/red-black-tree.html){:target="_blank"}<!--_-->
* [Red–black tree - Wikipedia (en)](https://en.wikipedia.org/wiki/Red–black_tree){:target="_blank"}<!--_-->

