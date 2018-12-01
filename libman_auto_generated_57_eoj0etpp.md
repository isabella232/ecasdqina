---
layout: lib
title: 連立一次方程式 (ガウスの消去法)
permalink: math/linear-algebra/GaussJordan

---


Gaussian elimination

TODO : ガウスの消去法とガウス・ジョルダンの消去法の違い (分かっていない)

# 行標準形

ガウスの消去法で得られる

一意に定まり，算出方法に依存しない

# 実装


```cpp
using Vec = vector< double >;
using Mat = vector< Vec >;
/// --- GaussJordan {{"{{"}}{ ///
Vec gaussJordan(Mat mat, Vec v, double eps = 1e-9) {
  int n = mat.size();
  assert(n == (int) mat[0].size() && n == (int) v.size());
  for(int i = 0; i < (int) v.size(); i++) {
    mat[i].emplace_back(v[i]);
  }
  for(int i = 0; i < n; i++) {
    int pivot = i;
    for(int j = i + 1; j < n; j++) {
      if(abs(mat[pivot][i] < abs(mat[j][i]))) pivot = j;
    }
    if(mat[pivot][i] < eps) return Vec();
    swap(mat[i], mat[pivot]);
    for(int j = i + 1; j <= n; j++) {
      mat[i][j] /= mat[i][i];
    }
    for(int j = 0; j < n; j++)
      if(j != i) {
        for(int k = i + 1; k <= n; k++) {
          mat[j][k] -= mat[j][i] * mat[i][k];
        }
      }
  }
  for(int i = 0; i < n; i++) v[i] = mat[i][n];
  return v;
}
/// }}}--- ///
```


# 検証

* [Strange Couple - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/review.jsp?rid=2805209#1){:target="_blank"}<!--_-->

# 練習問題

* [Strange Couple - AOJ](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2171){:target="_blank"}<!--_-->
* [D - 数列 XOR (600) - AtCoder](https://beta.atcoder.jp/contests/bitflyer2018-final-open/tasks/bitflyer2018_final_d){:target="_blank"}<!--_-->

# 参考

* [ガウスの消去法 - Wikipedia](https://ja.wikipedia.org/wiki/ガウスの消去法){:target="_blank"}<!--_-->
* [行階段形 - Wikipedia](https://ja.wikipedia.org/wiki/行階段形){:target="_blank"}<!--_-->

