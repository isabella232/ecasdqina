---
layout: lib
title: '連立一次方程式, 行列式, 行列のランク (ガウスの消去法 (Gaussian Elimination) )'
permalink: math/linear-algebra/GaussianElimination

---


Gaussian Elimination

ガウスの消去法 : 連立方程式を解くアルゴリズム

ガウス・ジョルダンの消去法 : 行基本変形を用いて行列を行階段形にすること．ガウスの消去法において後退代入を行わずに対角化を目指して直接解を求めること
 
# 行標準形

**一意に定まり**，算出方法に依存しない

ガウスの消去法で求めることができる

行簡約階段形とも

行階段形とは違って，主成分が1であるという条件，主成分がある列については，主成分以外は0であるといった条件がある

# 計算量

$N \times M$ 行列とすると，全て $O(N^2M)$

# ガウスの消去法

$N \times M$ 行列と サイズ $N$ のベクトルで表される方程式を解く

ガウス・ジョーダンの消去法をする

与えられた行列を破壊する (行標準形にする)．不定の場合はそれを自分でどうにかする

不能の場合はtupleのsecondの値がfalseになる


```cpp
// return ( values, isSolvable, rank )
// return 0-length vector when unsolvable
/// --- Gaussian Elimination {{"{{"}}{ ///
// using Gauss-Jordan Elimination
#include <cassert>
#include <cmath>
#include <utility>
#include <vector>
template < class T >
tuple< vector< T >, bool, int > Gauss_float(vector< vector< T > > &mat, vector< T > v,
                                            T eps = T(1e-9)) {
  size_t n = mat.size(), m = mat[0].size();
  assert(n == v.size());
  for(size_t i = 0; i < n; i++) {
    mat[i].push_back(v[i]);
  }
  size_t now = 0;
  for(size_t i = 0; i < m; i++) {
    int pivot = now;
    // pivotting
    for(size_t j = now + 1; j < n; j++) {
      if(abs(mat[pivot][i]) < abs(mat[j][i])) pivot = j;
    }
    if(abs(mat[pivot][i]) <= eps) continue;
    swap(mat[now], mat[pivot]);
    for(size_t j = i + 1; j <= m; j++) {
      mat[now][j] /= mat[now][i];
    }
    for(size_t j = 0; j < n; j++)
      if(j != i) {
        for(size_t k = i + 1; k <= m; k++) {
          mat[j][k] -= mat[j][i] * mat[now][k];
        }
      }
    now++;
    if(now == n) break;
  }
  for(size_t i = 0; i < n; i++) v[i] = mat[i][m];
  for(size_t i = now; i < n; i++) {
    if(abs(mat[i][m]) > eps) return make_tuple(v, false, now);
  }
  return make_tuple(v, true, now);
}
template < class T >
tuple< vector< T >, bool, int > Gauss(vector< vector< T > > &mat, vector< T > v) {
  size_t n = mat.size(), m = mat[0].size();
  assert(n == v.size());
  for(size_t i = 0; i < n; i++) {
    mat[i].push_back(v[i]);
  }
  size_t now = 0;
  for(size_t i = 0; i < m; i++) {
    int pivot = now;
    // pivotting
    for(size_t j = now + 1; j < n; j++) {
      if(mat[pivot][i] != T(0)) break;
      pivot = j;
    }
    if(mat[pivot][i] == T(0)) continue;
    swap(mat[now], mat[pivot]);
    for(size_t j = i + 1; j <= m; j++) {
      mat[now][j] /= mat[now][i];
    }
    for(size_t j = 0; j < n; j++)
      if(j != i) {
        for(size_t k = i + 1; k <= m; k++) {
          mat[j][k] -= mat[j][i] * mat[now][k];
        }
      }
    now++;
    if(now == n) break;
  }
  for(size_t i = 0; i < n; i++) v[i] = mat[i][m];
  for(size_t i = now; i < n; i++) {
    if(mat[i][m] != T(0)) return make_tuple(v, false, now);
  }
  return make_tuple(v, true, now);
}
/// }}}--- ///
```


# 行列式

ガウスの消去法と同様に，行階段形を作ることを目標にして行列式を求める


```cpp
// det {{"{{"}}{
#include <cassert>
#include <cmath>
#include <vector>
template < class T >
T det_float(vector< vector< T > > mat, T eps = T(1e-9)) {
  size_t n = mat.size();
  assert(n == mat[0].size());
  T res(1);
  for(size_t i = 0; i < n; i++) {
    int pivot = i;
    // pivotting
    for(size_t j = i + 1; j < n; j++) {
      if(abs(mat[pivot][i]) < abs(mat[j][i])) pivot = j;
    }
    if(abs(mat[pivot][i]) <= eps) return T(0);
    swap(mat[i], mat[pivot]);
    res *= mat[i][i];
    for(size_t j = i + 1; j < n; j++) {
      mat[i][j] /= mat[i][i];
    }
    for(size_t j = i + 1; j < n; j++)
      for(size_t k = i + 1; k < n; k++) {
        mat[j][k] -= mat[j][i] * mat[i][k];
      }
  }
  return res;
}
template < class T >
T det(vector< vector< T > > mat) {
  size_t n = mat.size();
  assert(n == mat[0].size());
  T res(1);
  for(size_t i = 0; i < n; i++) {
    int pivot = i;
    // pivotting
    for(size_t j = i + 1; j < n; j++) {
      if(mat[pivot][i] != T(0)) break;
      pivot = j;
    }
    if(mat[pivot][i] == T(0)) return T(0);
    swap(mat[i], mat[pivot]);
    res *= mat[i][i];
    for(size_t j = i + 1; j < n; j++) {
      mat[i][j] /= mat[i][i];
    }
    for(size_t j = i + 1; j < n; j++)
      for(size_t k = i + 1; k < n; k++) {
        mat[j][k] -= mat[j][i] * mat[i][k];
      }
  }
  return res;
}
// }}}
```


# 行列のランク

ガウスの消去法と同様に，行階段形を目指す

少数に対して求めるのは現実的ではないので，整数は有理数に変換するなどしたほうがよさそう


```cpp
// matrank, degree_of_freedom {{"{{"}}{
#include <vector>
template < class T >
int matrank(vector< vector< T > > mat) {
  size_t n = mat.size(), m = mat[0].size();
  size_t now = 0;
  for(size_t i = 0; i < m; i++) {
    int pivot = now;
    // pivotting
    for(size_t j = now + 1; j < n; j++) {
      if(mat[pivot][i] != T(0)) break;
      pivot = j;
    }
    if(mat[pivot][i] == T(0)) continue;
    swap(mat[now], mat[pivot]);
    for(size_t j = i + 1; j < m; j++) {
      mat[now][j] /= mat[now][i];
    }
    for(size_t j = now + 1; j < n; j++)
      for(size_t k = i + 1; k < m; k++) {
        mat[j][k] -= mat[j][i] * mat[now][k];
      }
    now++;
    if(now == n) break;
  }
  return now;
}
template < class T >
int degree_of_freedom(const vector< vector< T > > &mat) {
  return mat[0].size() - matrank(mat);
}
// }}}
```


# 前進消去と後退代入

ガウスの消去法に置いて，前進消去は行階段形にするステップ，後退代入は値を後ろから代入していく

# 部分選択法と完全選択法

pivottingの方法で，部分選択はその列についてのみ見る，完全選択は右下の領域全てをみる

完全選択法は定数が重くなる他，計算量が $O((N + M)NM)$ になる

# 検証

* [Strange Couple - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/review.jsp?rid=3349472#1){:target="_blank"}<!--_-->

ランク (自由度)

* [JAG春2013 J - Tree Reconstruction - AtCoder](https://atcoder.jp/contests/jag2013spring/submissions/4068497){:target="_blank"}<!--_-->
  * [rational]({{ "misc/Rational" | absolute_url }}) は少し時間がかかるため，gcdを行わないことで対応した

# 練習問題

* [Strange Couple - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2171){:target="_blank"}<!--_-->
* [D - 数列 XOR (600) - AtCoder](https://beta.atcoder.jp/contests/bitflyer2018-final-open/tasks/bitflyer2018_final_d){:target="_blank"}<!--_-->

# 参考

* [ガウスの消去法 - Wikipedia](https://ja.wikipedia.org/wiki/ガウスの消去法){:target="_blank"}<!--_-->
* [行階段形 - Wikipedia](https://ja.wikipedia.org/wiki/行階段形){:target="_blank"}<!--_-->
* [不定・不能と線型代数 - pepsin-amylaseの日記](https://topcoder.g.hatena.ne.jp/pepsin-amylase/20131203/1385984601){:target="_blank"}<!--_-->
* [競技プログラミングにおける連立方程式問題まとめ - はまやんはまやんはまやん](https://www.hamayanhamayan.com/entry/2017/03/15/221719){:target="_blank"}<!--_-->
* [線形代数Ｉ - 武内修＠筑波大](https://dora.bk.tsukuba.ac.jp/~takeuchi/?線形代数Ｉ){:target="_blank"}<!--_-->

