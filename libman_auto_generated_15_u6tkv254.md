---
layout: lib
permalink: data-structure/BST/RBST
title: RBST

---

Randomized BST
reference : https://www.slideshare.net/iwiwi/2-12188757/46
treapのほうがはやいのでいらない子

verify : https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/DSL_2_G/judge/3087089/C++14

// @ RBSTMultiset Sequence Library

verify : https://beta.atcoder.jp/contests/arc033/submissions/2978736

// @snippet rbst_multiset

```cpp
/// --- RBST Multiset Library {{"{{"}}{ ///

template < class Key >
struct RBSTMultiset {
public:
  const Key key;

private:
  using u32 = uint32_t;
  using PNN = std::pair< RBSTMultiset*, RBSTMultiset* >;
  RBSTMultiset *l = nullptr, *r = nullptr;
  int sz = 1;
  friend RBSTMultiset* prop(RBSTMultiset* a) {
    a->sz = size(a->l) + 1 + size(a->r);
    return a;
  }
  // XorShift128 [0, 2^32) {{"{{"}}{
  struct XorShift128 {
    using u32 = uint32_t;
    u32 x = 123456789, y = 362436069, z = 521288629, w = 88675123;
    XorShift128(u32 seed = 0) { z ^= seed; }
    u32 operator()() {
      u32 t = x ^ (x << 11);
      x = y;
      y = z;
      z = w;
      return w = (w ^ (w >> 19)) ^ (t ^ (t >> 8));
    }
  };
  // }}}
public:
  RBSTMultiset(Key key) : key(key) {}
  friend RBSTMultiset* merge(RBSTMultiset* a, RBSTMultiset* b) {
    static std::random_device rnd;
    static XorShift128 xs(rnd());
    if(a == nullptr) return b;
    if(b == nullptr) return a;
    if(xs() % (size(a) + size(b)) < (u32) size(a)) {
      a->r = merge(a->r, b);
      return prop(a);
    } else {
      b->l = merge(a, b->l);
      return prop(b);
    }
  }
  // lower (-inf, key), [key, inf)
  // upper (-inf, key], (key, inf)
  friend PNN split(RBSTMultiset* a, Key key, bool upper) {
    if(a == nullptr) return PNN(nullptr, nullptr);
    RBSTMultiset *sl, *sr;
    if(upper ? key < a->key : !(a->key < key)) {
      std::tie(sl, sr) = split(a->l, key, upper);
      a->l = sr;
      return PNN(sl, prop(a));
    } else {
      std::tie(sl, sr) = split(a->r, key, upper);
      a->r = sl;
      return PNN(prop(a), sr);
    }
  }
  friend PNN lower_split(RBSTMultiset* a, const Key& key) {
    return split(a, key, false);
  }
  friend PNN upper_split(RBSTMultiset* a, const Key& key) {
    return split(a, key, true);
  }
  friend int size(RBSTMultiset* a) { return a == nullptr ? 0 : a->sz; }
  friend void insert(RBSTMultiset*& a, Key key) {
    RBSTMultiset *sl, *sr;
    std::tie(sl, sr) = lower_split(a, key);
    a = merge(sl, merge(new RBSTMultiset(key), sr));
  }
  friend void erase(RBSTMultiset*& a, Key key) {
    RBSTMultiset *sl, *sr, *tl, *tr;
    std::tie(sl, sr) = upper_split(a, key);
    std::tie(tl, tr) = lower_split(sl, key);
    a = merge(tl, sr);
  }
  friend void erase(RBSTMultiset*& a, Key keyL, Key keyR,
                    bool inclusive = false) {
    RBSTMultiset *sl, *sr, *tl, *tr;
    std::tie(sl, sr) = split(a, keyR, inclusive);
    std::tie(tl, tr) = lower_split(sl, keyL);
    a = merge(tl, sr);
  }
  friend void erase1(RBSTMultiset*& a, Key key) {
    if(a == nullptr) return;
    if(key < a->key) {
      erase1(a->l, key);
    } else {
      if(!(a->key < key)) {
        a = merge(a->l, a->r);
        return;
      }
      erase1(a->r, key);
    }
    prop(a);
  }
  friend Key getKth(RBSTMultiset*& a, int k) {
    static const struct CannotGetKthOfNullptr {
    } ex;
    if(a == nullptr) throw ex;
    if(k <= size(a->l)) {
      if(k == size(a->l)) return a->key;
      return getKth(a->l, k);
    } else {
      return getKth(a->r, k - size(a->l) - 1);
    }
  }
  friend int count(RBSTMultiset*& a, Key key) {
    RBSTMultiset *sl, *sr, *tl, *tr;
    std::tie(sl, sr) = upper_split(a, key);
    std::tie(tl, tr) = lower_split(sl, key);
    int cnt = size(tr);
    a = merge(merge(tl, tr), sr);
    return cnt;
  }
  friend int count(RBSTMultiset*& a, Key keyL, Key keyR,
                   bool inclusive = false) {
    RBSTMultiset *sl, *sr, *tl, *tr;
    std::tie(sl, sr) = split(a, keyR, inclusive);
    std::tie(tl, tr) = lower_split(sl, keyL);
    int cnt = size(tr);
    a = merge(merge(tl, tr), sr);
    return cnt;
  }
};

/// }}}--- ///

RBSTMultiset< ll >* ms = nullptr;
```


