---
layout: lib
title: 基本
permalink: math/general

---



```cpp
// O(N^.5)
/// --- isPrime {{"{{"}}{ ///

bool isPrime(ll n) {
  if(n < 2) return false;
  for(ll i = 2; i * i <= n; i++) {
    if(n % i == 0) return false;
  }
  return true;
}

/// }}}--- ///
```



```cpp
// O(N^.5)
/// --- divisor {{"{{"}}{ ///
#include <vector>
vector< ll > divisor(ll n) {
  vector< ll > res;
  for(ll i = 1; i * i <= n; i++) {
    if(n % i == 0) {
      res.emplace_back(i);
      if(i != n / i) res.emplace_back(n / i);
    }
  }
  return res;
}
/// }}}--- ///
```



```cpp
// O(N^.5)
/// --- primeFactors {{"{{"}}{ ///
#include <map>
map< ll, int > primeFactors(ll n) {
  map< ll, int > res;
  for(ll i = 2; i * i <= n; i++) {
    while(n % i == 0) n /= i, res[i]++;
  }
  if(n != 1) res[n] = 1;
  return res;
}
/// }}}--- ///
```


# オイラーのトーシェント関数

[オイラーのトーシェント関数の性質]({{ "math/number-theory/EulerTotient" | absolute_url }}) に色々書きました


```cpp
// O(N^.5)
/// --- phi {{"{{"}}{ ///
ll phi(ll n) {
  ll res = n;
  for(ll i = 2; i * i <= n; i++) {
    if(n % i == 0) {
      res = res / i * (i - 1);
      while(n % i == 0) n /= i;
    }
  }
  if(n != 1) res = res / n * (n - 1);
  return res;
}
/// }}}--- ///
```



```cpp
// O(N log log N)
/// --- phi2 {{"{{"}}{ ///
#include <vector>
vector< int > phi2(int n) {
  n++;
  vector< int > euler(n);
  for(int i = 0; i < n; i++) euler[i] = i;
  for(int i = 2; i < n; i++) {
    if(euler[i] == i) {
      for(int j = i; j < n; j += i) euler[j] = euler[j] / i * (i - 1);
    }
  }
  return euler;
}
/// }}}--- ///
```


---


```cpp
// O(N log log N)
/// --- primes {{"{{"}}{ ///
#include <vector>
vector< int > primes(int n) {
  vector< int > res;
  for(int i = 2; i <= n; ++i) {
    bool isp = 1;
    for(int p : res) {
      if((ll) p * p > i) break;
      if(i % p == 0) {
        isp = 0;
        break;
      }
    }
    if(isp) res.emplace_back(i);
  }
  return res;
}
/// }}}--- ///
```


# 原始根判定

奇素数 $p$ に対し $p-1$ の約数を乗じたときに $1 \pmod p$ でなければ原始根であると言えます

このことはフェルマーの小定理などから簡単に導けます


```cpp
// O(log p)
/// --- isPrimitive {{"{{"}}{ ///
bool isPrimitive(ll x, ll p) {
  auto ds = divisor(p - 1);
  for(ll d : ds)
    if(d != p - 1) {
      if(modpow(x, d, p) == 1) return false;
    }
  return true;
}
/// }}}--- ///
```


---

gcd, lcm, extgcd, modinv, modpow

gcd(a,b)の計算量は $O(\log \max(a, b))$ です

証明は蟻本にありますが，第一引数が2段階の再帰で $1/2$ 以下になることから言えます

extgcd(a, b, x, y)は $ab \not= 0$ のとき $$|x| \leq b, |y| \leq a$$ が言えます  
帰納法で証明することができます

modinv(a, m)は $a^{-1} \bmod m$ を返します  
$\gcd(a, m) = 1$ であれば $m$ が素数でなくとも逆元を返せるようにextgcdを使っています  
速度もこちらのほうが速い気がします


```cpp
/// --- math {{"{{"}}{ ///
ll gcd(ll a, ll b) { return b == 0 ? a : gcd(b, a % b); }
ll lcm(ll a, ll b) { return a / gcd(a, b) * b; }
ll extgcd(ll a, ll b, ll &x, ll &y) {
  ll d;
  return b == 0 ? (x = a < 0 ? -1 : 1, y = 0, a < 0 ? -a : a)
                : (d = extgcd(b, a % b, y, x), y -= a / b * x, d);
}
ll modinv(ll a, ll mod) {
  ll x, y;
  extgcd(a, mod, x, y);
  if(x < 0) x += mod;
  return x;
}
ll modpow(ll a, ll b, ll mod) {
  ll r = 1;
  a %= mod;
  while(b) {
    if(b & 1) r = r * a % mod;
    a = a * a % mod;
    b >>= 1;
  }
  return r;
}
/// }}}--- ///
```


# 剰余を使わないgcd

`%2` などは `&1` と読み替えてください

```cpp
Integer gcd(Integer a, Integer b) {
  if(a < 0) a = -a;
  if(b < 0) b = -b;
  Integer res = 1;
  while(a != b) {
    if(a < b) swap(a, b);
    if(b == 0) break;
    if(!(a % 2) && !(b % 2)) res *= 2, a /= 2, b /= 2;
    else if(!(a % 2)) a /= 2;
    else if(!(b % 2)) b /= 2;
    else a = (a - b) / 2;
  }
  return res * a;
}
```

# 大きなmodでの掛け算

二倍しても大丈夫なmodでの掛け算をしたいときに，二分累乗や繰り返し二乗法を使うことでoverflowを防ぐことができますが，ちょっと時間がかかります．[多倍長整数]({{ "misc/BigInteger" | absolute_url }})を使うのもありです

### long doubleを使った方法

参考 : [巨大modでの掛け算の高速化 (Codeforces Round #259 D Little Pony and Elements of Harmony) - kazuma8128’s blog](http://kazuma8128.hatenablog.com/entry/2018/06/04/144254){:target="_blank"}<!--_-->


```cpp
/// --- modmul {{"{{"}}{ ///
ll modmul(ll x, ll y, ll mod) {
  ll res = (x * y - (ll)((long double) x * y / mod) * mod) % mod;
  return res < 0 ? res + mod : res;
}
/// }}}--- ///
```


overflowを打ち消しています

### 繰り返し二乗法

対数時間かかります

```cpp
ll modmul_slow(ll a, ll b, ll mod) {
  if(b < 0) a *= -1, b *= -1;
  ll res = 0;
  while(b) {
    if(b & 1) res = (res + a) % mod;
    a = (a + a) % mod;
    b >>= 1;
  }
  return res;
}
```

# 参考

* 蟻本
* アルゴリズムイントロダクション
* [巨大modでの掛け算の高速化 (Codeforces Round #259 D Little Pony and Elements of Harmony) - kazuma8128’s blog](http://kazuma8128.hatenablog.com/entry/2018/06/04/144254){:target="_blank"}<!--_-->
