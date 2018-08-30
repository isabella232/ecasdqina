---
layout: lib
title: 最長回文半径 (Manacher)
permalink: string/Manacher

---


奇数長のものしか求められないので，偶数長を求めたいときは`$`など使わない文字を1字ごとに挿入します．


```cpp
// [i] = max radius as a palindrome when [i] is center
// NOTE : 偶数長ほしいなら'$'とか挿入してね
vector< int > Manacher(string s) {
  int n = s.size();
  int i = 0, j = 0;
  vector< int > R(n);
  while(i < n) {
    while(i - j >= 0 && i + j < n && s[i - j] == s[i + j]) ++j;
    R[i] = j;
    int k = 1;
    while(i - k >= 0 && R[i - k] < j - k) R[i + k] = R[i - k], ++k;
    i += k, j -= k;
  }
  return R;
}
```


# 参考

* [文字列の頭良い感じの線形アルゴリズムたち２](http://snuke.hatenablog.com/entry/2014/12/02/235837){:target="_blank"}
* [アルゴリズムは女の子#1 \| るまブログ](https://tomorinao.blogspot.com/2018/03/1_20.html){:target="_blank"}
  * なにこれ

# 検証

* [D - ukuku \| うくこん](https://ukuku09.contest.atcoder.jp/submissions/2799976){:target="_blank"}
