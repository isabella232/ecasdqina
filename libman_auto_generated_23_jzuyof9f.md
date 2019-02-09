---
layout: lib
title: 木の重心 (Centroid)
permalink: graph/Centroid

---


# 重心列挙

変数は察して


```cpp
vector< int > centroid;
int c[N];
void dfs(int i, int p) {
  c[i] = 1;
  int f = 1;
  for(int j : g[i])
    if(j != p) {
      dfs(j, i);
      c[i] += c[j];
      f &= c[j] <= n / 2;
    }
  f &= (n - c[i]) <= n / 2;
  if(f) centroid.emplace_back(i);
}
```


# 検証

* [F - Squirrel Migration (800) - AtCoder](https://beta.atcoder.jp/contests/arc087/submissions/2634201){:target="_blank"}<!--_-->

# 練習問題

* [F - Squirrel Migration (800) - AtCoder](https://beta.atcoder.jp/contests/arc087/tasks/arc087_d){:target="_blank"}<!--_-->

# 参考

* 蟻本
* [木の重心列挙アルゴリズム - Learning Algorithms](http://www.learning-algorithms.com/entry/2018/01/03/215559){:target="_blank"}<!--_-->

