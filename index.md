---
layout: home
last_update : 2018-10-18
---

えかライブラリだよ

(工事中，完成はだいぶ先になりそう)

# データ構造

## セグメントツリー

* [普通のセグメントツリー]({{ "data-structure/SegmentTree/SegmentTree" | relative_url }})
* [遅延セグメントツリー]({{ "data-structure/SegmentTree/LazySegmentTree" | relative_url }})
* [二次元セグメントツリー]({{ "data-structure/SegmentTree/SegmentTree2D" | relative_url }})
* [動的セグメントツリー]({{ "data-structure/SegmentTree/DynamicSegmentTree" | relative_url }})
* [フラクショナルカスケーディング]({{ "data-structure/SegmentTree/FractionalCascadingSegmentTree" | relative_url }})
* [永続セグメントツリー]({{ "data-structure/SegmentTree/PersistentSegmentTree" | relative_url }})

## いろいろ

* [Binary Indexed Tree (Fenwick Tree)]({{ "data-structure/misc/BIT" | relative_url }})
* [BIT2D]({{ "data-structure/misc/BIT2D" | relative_url }})
* [Sparse Table]({{ "data-structure/misc/SparseTable" | relative_url }})
* [Link/Cut Tree]({{ "data-structure/misc/LinkCutTree" | relative_url }})
* [Union Find (DSU)]({{ "data-structure/misc/UnionFind" | relative_url }})

## Heap

* [併合可能Heap (LeftistHeap)]({{ "data-structure/Heap/LeftistHeap" | relative_url }})
* [併合可能Heap (SkewHeap)]({{ "data-structure/Heap/SkewHeap" | relative_url }})
* [RadixHeap]({{ "data-structure/Heap/RadixHeap" | relative_url }})

## [平衡二部探索木(Balanced Binary Search Tree; BBST)]({{ "data-structure/BBST" | relative_url }})

* [RBST]({{ "data-structure/BBST/RBST" | relative_url }})
* [Treap]({{ "data-structure/BBST/Treap" | relative_url }})

# グラフ

* [基本]({{ "graph/general" | relative_url }})
* [強連結成分分解 (Kosaraju's algorithm)]({{ "graph/StronglyConnectedComponent" | relative_url }})
* [木のダブリング]({{ "graph/DoublingTree" | relative_url }})
* [HL分解]({{ "graph/HL-Decomposition" | relative_url }})
* [LowLink]({{ "graph/lowlink" | relative_url }})
* [二重連結成分分解]({{ "graph/BiconnectedComponent" | relative_url }})
* [二辺連結成分分解]({{ "graph/BiedgeConnectedComponent" | relative_url }})
* [全方位木DP (るま式)]({{ "graph/DP-all-subtree" | relative_url }})
* [ユーティリティ]({{ "graph/utility" | relative_url }})
* [木の同型判定]({{ "graph/is-same-tree" | relative_url }})
* [最小全域木 (TODO)]({{ "graph/MST" | relative_url }})

## フロー

* [二部マッチング]({{ "graph/flow/bipartiteMatching" | relative_url }})
* [最大フロー (Dinic)]({{ "graph/flow/Dinic" | relative_url }})
* [最大フロー (Ford-Fulkerson)]({{ "graph/flow/FordFulkerson" | relative_url }})
* [最小費用流]({{ "graph/flow/MinCostFlow" | relative_url }})

# 数学

* [基本]({{ "math/general" | relative_url }})
* [連立一次方程式 (Gauss-Jordan)]({{ "math/GaussJordan" | relative_url }})
* [連立一次方程式 (Givens)]({{ "math/Givens" | relative_url }})
* [行列の積と累乗]({{ "math/matrix" | relative_url }})
* [メビウス関数]({{ "math/moebius" | relative_url }})
* [オイラーのトーシェント関数（ファイ関数）の性質]({{ "math/EulerTotient" | relative_url }})
* [包除原理]({{ "math/PIE" | relative_url }})
* [中国剰余定理 (CRT)]({{ "math/CRT" | relative_url }})
* [モンゴメリ乗算]({{ "math/Montgomery" | relative_url }})
* [モノイド，モノイド作用]({{ "math/Monoid" | relative_url }})
* [ラグランジュ緩和問題 (永遠に未完成)]({{ "math/LagrangianRelaxation" | relative_url }})

## FFT

* [FFT導入]({{ "math/FFT/introduction" | relative_url }})
* [基本的なFFT]({{ "math/FFT/standard" | relative_url }})
* [数論変換 (Number Theoretic Transform; NTT)]({{ "math/FFT/NTT" | relative_url }})
* [Cooley-Tukey FFT]({{ "math/FFT/Cooley-Tukey-FFT" | relative_url }})

# 幾何

* [幾何基本]({{ "geometory/geometory" | relative_url }})
* [凸包 (Graham Scan, Andrew Scan)]({{ "geometory/ConvexHull" | relative_url }})
* [3次元幾何基本ライブラリ]({{ "geometory/geometory3D" | relative_url }})
* [3次元凸包]({{ "geometory/ConvexHull3D" | relative_url }})

# アルゴリズム

* [クエリ平方分割 (Mo's Algorithm)]({{ "algorithm/Mo" | relative_url }})
* [Rollback可能なデータ構造とクエリ平方分割（Moの上位互換）]({{ "algorithm/MoEx" | relative_url }})
* [時空間Mo (三次元Mo，Mo with Update)]({{ "algorithm/Mo3D" | relative_url }})
* [きたまさ法]({{ "algorithm/Kitamasa" | relative_url }})
* [Garnerのアルゴリズム]({{ "algorithm/Garner" | relative_url }})
* [高速ゼータ変換]({{ "algorithm/FastZetaTransform" | relative_url }})
* [最適二分探索木 (Hu-Tucker)]({{ "algorithm/Hu-Tucker" | relative_url }})

# [文字列]({{ "string" | relative_url }})

* [最長回文半径 (Manacher)]({{ "string/Manacher" | relative_url }})
* [s\[0,i-1\]の接頭辞と接尾辞が一致する最大の長さ (KMP法), 最小周期]({{ "string/KMP" | relative_url }})
* [sとs\[i, -1\]の最長共通接頭辞 (z-algorithm)]({{ "string/z-algorithm" | relative_url }})
* [文字列検索 (Aho-Corasick法)]({{ "string/AhoCorasick" | relative_url }})
* [接尾辞配列 (Suffix Array) (Manber & Myers)]({{ "string/SA-with-Manber-Myers" | relative_url }})
* [接尾辞配列 (Suffix Array) (SA-IS)]({{ "string/SA-IS" | relative_url }})
* [LCP配列 (Kasai's algorithm)]({{ "string/LCP-Array" | relative_url }})
* [木を文字列に変換する]({{ "string/tree-to-string" | relative_url }})
* [文字列ユーティリティ]({{ "string/string-utility" | relative_url }})

# いろいろ

* [Convex Hull Trick (動的も)]({{ "misc/ConvexHullTrick" | relative_url }})
* [Modulo Integer]({{ "misc/ModuloInteger" | relative_url }})
* [有理数ライブラリ]({{ "misc/Rational" | relative_url }})
* [多倍長整数, 任意精度演算]({{ "misc/BigInteger" | relative_url }})
* [ビット操作ユーティリティ]({{ "misc/bit-utility" | relative_url }})
* [階乗の事前計算]({{ "misc/Factorial" | relative_url }})
* [XorShift]({{ "misc/XorShift" | relative_url }})

# テクニック

* [巨大な累乗・階乗計算の誤差をlogで抑える]({{ "trick/log-exp" | relative_url }})

# ハマりポイント

* [C++における左辺と右辺の評価順序]({{ "trap/eval-order" | relative_url }})
* [不適切な評価関数をsortなどに使っている]({{ "trap/improper-comparison" | relative_url }})

# 関連話題

* [誤差]({{ "topic/error" | relative_url }})

# リンク集

* [ei1333's page (うしさんのすごいやつ)](https://ei1333.github.io){:target="_blank"}<!--_-->
  * アルゴリズムのリンク集が便利だよ
* [DEGwerさんの数え上げテクニック集](http://d.hatena.ne.jp/DEGwer/20171220){:target="_blank"}<!--_-->
* [kirikaさんの整数論集](https://github.com/kirika-comp/articles){:target="_blank"}<!--_-->
  * seisuuron.tex
* [E-Maxx Algorithms in English](https://cp-algorithms.com){:target="_blank"}<!--_-->
  * いろいろある
* [競技プログラミング練習問題集 - はまやんはまやんはまやん](https://www.hamayanhamayan.com/entry/2100/01/01/000000){:target="_blank"}<!--_-->
* [COMPILER EXPLORER](https://godbolt.org/){:target="_blank"}<!--_-->
  * 最適化されているか，などを確認できる

