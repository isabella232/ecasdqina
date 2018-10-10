---
layout: lib
title: 併合可能heap (LeftistHeap)
permalink: data-structure/Heap/LeftistHeap

---


併合可能なヒープのことをMeldable Heapといいます


```cpp
/// --- LeftistHeap Library {{"{{"}}{ ///
template < class T >
struct LeftistHeap {
  LeftistHeap *l = nullptr, *r = nullptr;
  T val;
  int rnk;
  LeftistHeap(T val = T()) : val(val) {}
};

template < class T, class Compare = less< T > >
LeftistHeap< T > *meld(LeftistHeap< T > *a, LeftistHeap< T > *b,
                       const Compare &comp = Compare()) {
  if(a == nullptr) return b;
  if(b == nullptr) return a;
  if(!comp(a->val, b->val)) swap(a, b);
  a->r = meld(a->r, b, comp);
  if(a->l == nullptr || a->l->rnk < a->r->rnk) swap(a->l, a->r);
  a->rnk = ((a->r == nullptr) ? 0 : a->r->rnk) + 1;
  return a;
}

template < class T, class Compare = less< T > >
inline LeftistHeap< T > *push(LeftistHeap< T > *&a, T const &e,
                              const Compare &comp = Compare()) {
  LeftistHeap< T > *b = new LeftistHeap< T >(e);
  a = a == nullptr ? b : meld(a, b, comp);
  return b;
}

template < class T, class Compare = less< T > >
inline void pop(LeftistHeap< T > *&a, const Compare &comp = Compare()) {
  a = meld(a->l, a->r, comp);
}
template < class T, class Compare = less< T > >
LeftistHeap< T > *second(LeftistHeap< T > *a, const Compare &comp = Compare()) {
  return a->r == nullptr ? a->l : comp(a->l->val, a->r->val) ? a->l : a->r;
}
/// }}}--- ///
```


# 練習問題

* [C - 最適二分探索木 (ATC002) - AtCoder](https://beta.atcoder.jp/contests/atc002/tasks/atc002_c){:target="_blank"}<!--_-->
* [E - Range Minimum Queries (600) - AtCoder](https://beta.atcoder.jp/contests/arc098/tasks/arc098_c){:target="_blank"}<!--_-->
  * N が問題にある制約より大きくても対応できる解法に使います．[noshi91さんの解説](http://noshi91.hatenablog.com/entry/2018/06/08/181455){:target="_blank"}<!--_-->

# 参考

* [http://hos.ac/blog/#blog0001](http://hos.ac/blog/#blog0001){:target="_blank"}

