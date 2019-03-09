---
layout: lib
title: Binary Indexed Tree (Fenwick Tree)
permalink: data-structure/misc/BinaryIndexedTree

---


`.get(i)` は自分で配列を用意することで $O(\log N)$ を $O(1)$ にできるので注意

[ei1333さんの解説](https://ei1333.github.io/algorithm/binary-indexed-tree.html){:target="_blank"}<!--_--> を参考に $O(\log N)$ の二分探索も入っている  
`.sum(i)` が `i` について単調非減少であればよい

# 一般化しようとすると

まずはBITの操作を一般化します

作用を受ける半群を $(X, \ast)$, 作用する集合を $M$ とおく

(左)外部演算を $\circ \colon M \times X \mapsto X$ とします

`.add(i, x)` は $a_i \coloneqq x \circ a_i$ とする (1点に対する作用)  
`.sum(i)` は $a_0 \ast a_1 \ast a_2 \ast \cdots \ast a_i$ を返す  

さて，BITは $a_0 \ast \cdots \ast a_5 = (a_0 \ast \cdots \ast a_3) \ast (a_4 \ast a_5)$のように分解して`.sum`を計算します．結合則を要求するのはいいかと思います

単位元はなくとも，予め値を受け取り各ノードが支配する値を計算していけば $O(N \log N)$ で処理できます

逆元があれば任意の区間について `.sum`が求められます．交換則はいらないです．$(a \ast b)^{-1} = b^{-1} \ast a^{-1}$であるため

### 要件

$a_i \in X, x \in M$

$a_0 \ast a_1 \ast \cdots \ast a_i \ast \cdots \ast a_{n-1}= A$  
$a_0 \ast a_1 \ast \cdots \ast (x \circ a_i) \ast \cdots \ast a_{n-1} = B$

とします

$A,x,a_i,n$ から $B$ を $O(1)$ などで求められる必要があります  
($a_0, a_{n-1}$なども使っていいですが $O(1)$ から遠ざかるだけかと思います)


正直良くわからなくないですか．例を挙げます

### 例1

$X = M = \Z$ とし, $a \ast b = a \circ b = a + b$ とします．つまり通常のBITを考えます

$B = A + x$ となります

### 例2

$X = M = \Z$ とし, $a \ast b = a \circ b = \min(a, b)$ とします．逆元は無いので群にはなりませんが有名なBITです

$B = \min(A, x)$ となります

---

２つの例を見ましたがこれがどう一般化できるかはよくわかりません

さらに上記の条件を満たすなら (BIT特有の添字条件でのみ乗せられるとかいう特殊を除き) セグメントツリーに乗せられることもわかります

あまり一般化せず足し算についてのみ持っていれば十分だと思います

# 実装


```cpp
// .add(i, v)   : bit[i] += v
// .get(i)      : bit[i]
// .sum(i)      : bit[0] + ... + bit[i]
// .range(l, r) : bit[l] + ... + bit[r]
// .lower_bound(T v) : min i that satisfies .sum(i) >= v
//    - use only when bit[i] >= 0 for all i > 0
/// --- Binary Indexed Tree {{"{{"}}{ ///
#include <cassert>
#include <vector>
template < class T = long long >
struct BinaryIndexedTree {
  using size_type = std::size_t;
  size_type n, m;
  T identity;
  std::vector< T > data;
  BinaryIndexedTree() : n(0) {}
  BinaryIndexedTree(int n, T identity = T())
      : n(n), identity(identity), data(n, identity) {
    m = 1;
    while(m < n) m <<= 1;
  }
  void add(size_type i, T x) {
    assert(i < n);
    i++;
    while(i <= n) {
      data[i - 1] = data[i - 1] + x;
      i += i & -i;
    }
  }
  T sum(int i) {
    if(i < 0) return identity;
    if(i >= n) i = n - 1;
    i++;
    T s = identity;
    while(i > 0) {
      s = s + data[i - 1];
      i -= i & -i;
    }
    return s;
  }
  T get(int i) { return sum(i) - sum(i - 1); }
  T range(int a, int b) { return sum(b) - sum(a - 1); }
  size_type lower_bound(T w) {
    size_type i = 0;
    for(size_type k = m; k > 0; k >>= 1) {
      if(i + k <= n && data[i + k - 1] < w) w -= data[(i += k) - 1];
    }
    return i;
  }
};
/// }}}--- ///

template < class T = long long >
using BIT = BinaryIndexedTree< T >;
```


# 参考

* 蟻本 p.159
* [BIT(Binary-Indexed-Tree) - ei1333's page](https://ei1333.github.io/algorithm/binary-indexed-tree.html){:target="_blank"}<!--_-->

