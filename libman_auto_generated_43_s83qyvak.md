---
layout: lib
title: メビウス関数
permalink: math/number-theory/moebius

---


メビウスの反転公式などで使う


```cpp
// O(N^.5)
unordered_map< ll, int > moebius(ll n) {
  unordered_map< ll, int > res;
  vector< ll > primes;
  for(ll i = 2; i * i <= n; i++) {
    if(n % i == 0) primes.emplace_back(i);
  }
  if(n != 1) primes.emplace_back(n);
  int m = primes.size();
  for(int i = 0; i < (1 << m); i++) {
    int mu = 1;
    ll num = 1;
    for(int j = 0; j < m; j++)
      if(i & (1 << j)) {
        mu *= -1;
        num *= primes[j];
      }
    res[num] = mu;
  }
  return res;
}
```


# 検証

* [F - ModularPowerEquation!! - AtCoder](https://beta.atcoder.jp/contests/tenka1-2017/submissions/3311870){:target="_blank"}<!--_-->

# 練習問題

* [F - ModularPowerEquation!! (1400) - AtCoder](https://beta.atcoder.jp/contests/tenka1-2017/tasks/tenka1_2017_f){:target="_blank"}<!--_-->
  * 解説 : [F: ModularPowerEquation!!の謎を紐解く - るまブログ](https://beta.atcoder.jp/contests/tenka1-2017/submissions/3311870){:target="_blank"}<!--_-->


