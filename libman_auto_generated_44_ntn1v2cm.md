---
layout: lib
title: log_a b mod p (Baby-Step Gian-Step algortihm)
permalink: math/number-theory/Baby-Step-Giant-Step

---


離散対数

与えられた $a, b, p$ に対し，

$a^x \equiv b \mod p$ を満たす非負整数 $x$ の最小値を求めます

# Baby-Step Gian-Step algortihm

$\gcd(a, p) = 1$ とします

$x^{\phi(m)} \equiv 1 \mod m$ , $\phi(m) \leq m - 1$ より周期性があり，解が存在する場合，$0 \leq x \lt m-1$ が成立する $x$ が存在することがわかります

([オイラーのファイ関数の特徴]({{ "math/number-theory/EulerTotient" | absolute_url }}))

$\displaystyle r = \lceil \sqrt{m} \rceil$ とすると

$\displaystyle x = pr + q \\ \\ (0 \leq p \leq r, 0 \leq q \lt r)$ と置くことができます

代入して式変形すると

$b(a^{-1})^q \equiv (a^r)^p$ がわかります

左辺を各$q$ について求めて，`map[値] = その値を達成する最小のq` を保持します (Baby-Step)

右辺に$p$ を代入して **左辺がその値になり得るか** を`map`で判定できます (Giant-Step)

なり得る場合は $x$ を復元できます

計算量は $r = O(\sqrt m)$ なので， $O(\sqrt m \log m)$ ，hashmapを使うと $O(\sqrt m)$ でできます

## 補足

$q$ から求めることで早期リターンできます

Baby-Step Giant-Step アルゴリズムは中間一致攻撃のひとつです

巡回群 $G$ と生成元 $\alpha$ から $\alpha^x = \beta$ になる $x$ を求めていると言うことができます

なので $\gcd(a, p) \not= 1$ のときはこのアルゴリズムの範疇ではないんですが，もし求めるなら，

$x = 0$ の場合は自明なので $x \geq 1$ とします．$g=\gcd(a, p)$ とすると

まず $g \nmid b$ (<=> $g$ は $b$ の約数でない) なら解はありません


$g \mid b$ なら

$\displaystyle a^x \equiv b \mod p$

$\displaystyle a a^{x-1} \equiv b \mod p$

$\displaystyle (\frac{a}{g}) a^{x-1} \equiv \frac{b}{g} \mod \frac{p}{g}$

$\displaystyle \gcd(\frac{a}{g}, \frac{p}{g}) = 1$ なので逆元が存在して，

$\displaystyle a^{x-1} \equiv \frac{b}{g}(\frac{a}{g})^{-1} \mod \frac{p}{g}$

となりサイズの小さい問題に帰着できました (**このときの割り算と逆元は意味が違うので注意してください**)

$p \\ \\ (\geq 1)$ は単調減少なのでいつか終了します．また，$g \geq 2$ であるため，$O(\log p)$ 回で $\gcd(a, p)=1$ になります．毎回gcd, modinvに$O(\log p)$かかりますが$O(\log^2 p) \subset O(\sqrt{p})$ より悪化なしです

# 実装

aとpが互いに素でなくてもよい．解が存在する場合は非負の最小値，存在しない場合は `-1` を返す

計算量は $O(\sqrt p \log p)$ です


```cpp
/// --- math {{"{{"}}{ ///
ll gcd(ll a, ll b) { return b == 0 ? a : gcd(b, a % b); }
ll lcm(ll a, ll b) { return a / gcd(a, b) * b; }
ll extgcd(ll a, ll b, ll &x, ll &y) {
  ll d;
  return b == 0 ? (x = 1, y = 0, a)
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

// require math library
// modlog(a, b, p) = log_a b
// gcd(a, p) = 1
// modlog {{"{{"}}{

// Baby-Step Giant-Step algorithm
ll modlog(ll a, ll b, ll p) {
  a %= p;
  b %= p;
  if(a < 0) a += p;
  if(b < 0) b += p;

  if(a == 1 && b != 1) return -1;
  if(b == 1) return 0;

  ll g;
  ll bias = 0;
  while((g = gcd(a, p)) != 1) {
    if(b % g != 0) return -1;
    b /= g;
    p /= g;
    b = b * modinv(a / g, p) % p;
    bias++;
    if(b == 1) return bias;
  }

  // p <= r^2

  ll ok = p, ng = 0;
  while(ok - ng > 1) {
    ll mid = (ok + ng) >> 1;
    if(mid * mid >= p)
      ok = mid;
    else
      ng = mid;
  }

  int r = ok;

  ll A = modpow(a, r, p);
  ll ainv = modinv(a, p);

  map< ll, int > table;

  // baby step
  ll baby = b;
  for(int i = 0; i < r; i++, baby = baby * ainv % p) {
    table[baby] = max(table[baby], r - i);
  }

  // giant step
  ll giant = 1;
  for(int i = 0; i <= r; i++, giant = giant * A % p) {
    if(table.count(giant)) {
      return (ll) i * r + r - table[giant] + bias;
    }
  }

  return -1;
}

// }}}
```


# 検証

* 互いに素のみ [D - あまり - AtCoder](https://beta.atcoder.jp/contests/arc042/submissions/3573120){:target="_blank"}<!--_-->

互いに素でない場合はなるべく慎重にテストしました

# 練習問題

* [D - あまり - AtCoder](https://beta.atcoder.jp/contests/arc042/tasks/arc042_d){:target="_blank"}<!--_-->

# 参考

* [「1000000007 で割ったあまり」の求め方を総特集！ 〜 逆元から離散対数まで 〜 #離散対数 - Qiita](https://qiita.com/drken/items/3b4fdf0a78e7a138cd9a#6-離散対数-log_a-b){:target="_blank"}<!--_-->
* [Baby-step giant-step - Wikipedia (en)](https://en.wikipedia.org/wiki/Baby-step_giant-step){:target="_blank"}<!--_-->

