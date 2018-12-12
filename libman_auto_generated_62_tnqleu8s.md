---
layout: lib
title: 数論変換 (Number Theoretic Transform; NTT)
permalink: math/FFT/NTT

---


多項式2つを高速にかけ合わせます

[FFT導入]({{ "math/FFT/introduction" | absolute_url }}) で紹介しているように，FFTをする際の $\zeta_N$ に複素数ではなく $\mathrm{mod}\ m$ 上の原始N乗根を使うというのがNTTです

掛け算と足し算が定義されている可換環 $R$ において，$N$ で割ること，位数 $N$ の元が存在することができれば，畳み込みができますね (正確には $N = x_1x_2\cdots x_t (x_i \in R)$ と表したときに，各 $x_i$ で任意の元を割り算できるような $x$ があれば十分となります) (可換環については整数みたいに足し算掛け算ができて掛け算が $a\*b=b\*a$ を満たす，程度に考えてください)

なので，$m$ と $N$ が互いに素で，位数 $N$ の元があればいいので，$m$ を素数とし，$N \mid (m - 1)$ となるように $m$ を選べば良くなります (素数modでは逆元が存在するため割り算が可能)

ところで $N$ についてですが，FFTのアルゴリズムの選び方に依存します．NTTはどのような構造の上でFFTをするか，という話で，FFTのアルゴリズムは別に決定することができます

以下では [基本的なFFT]({{ "math/FFT/standard" | absolute_url }}) で紹介している非再帰の2基底FFTを使用します．$N$ は2ベキとします

# 使い方

$k, n$ を正整数とし，$m = 2^nk + 1$ と表せるもので， $ 2^n \geq N$ であるような **素数** $m$ を選び， $\mathrm{mod}\ m$ でFFTができる  

出力の最大値について考える．荒い評価として，$a, b$ を入力，各サイズを $N, M$ とすると，$\max(a)\max(b)\min(N, M)$ ぐらいとなる

## 求めたい値がmod上での値のとき

まず，予めmodをとることができるので各要素の値をmodしておく

上記の条件を満たすようなmodの場合はそのままNTTを使えばいい (以下の実装では `NTT::Core<mod, primitive, MAX_H> ntt` を宣言し `ntt::conv(vector<ll> a, vecror<ll> b)`)

mod 1e9+7 といったような任意modについては [中国剰余定理 (CRT)]({{ "math/number-theory/CRT" | absolute_url }}) などで復元する必要がある

以下の実装では `NTT::conv< USE > (vector<ll> a, vector<ll> b, ll m)` を使うと `USE` 個のmod上で値を出して [Garnerのアルゴリズム]({{ "math/number-theory/Garner" | absolute_url }}) を用いて mod m に復元を行う

NTTを行うmodを $m_1, m_2, \cdots, m_{USE}$ とすると，厳密値 (予めmodを取った場合はそれを厳密値とする) の最大値 (mod $m$に復元したいとした時，$m^2\min(N, M)$ ぐらい) を $m_1m_2\cdots m_{USE}$ が超えるように取らなければならない

<!--_-->

## 求めたい値が厳密な値のとき (64bitに収まるとき)

基本的にmod上で求めるときと同じように，最大値より大きな値をmodとすればよい

[Garnerのアルゴリズム]({{ "math/number-theory/Garner" | absolute_url }}) に基づいているため，その [特殊ケース]({{ "math/number-theory/Garner#特殊ケース" | absolute_url }}) が成り立つような，`USE=2` の場合などは特に気にしなくていい

それ以外のパターンでoverflowしそうなら，**int128などで掛け算を行うか**， [数学>基本]({{ "math/general" | absolute_url }}) にあるmodmulに置き換えるといった対策をする

## 求めたい値が厳密な値の時 (64bitに収まらないとき)

復元時にint128, int256など多倍長整数使う

# 実装


```cpp
// MAX_N is max size of OUTPUT, DOUBLED INPUT
// MAX_RES_VALUE = MAX_VALUE^2 * MAX_N
// if MAX_N > 2^20, comment out primes!
// NTT {{"{{"}}{
#include <cassert>
#include <cstdint>
#include <vector>

namespace NTT {
using uint = uint_fast32_t;

// NTT_PRIMES {{"{{"}}{
constexpr ll NTT_PRIMES[][2] = {
    {1224736769, 3}, // 2^24 * 73 + 1,
    {1053818881, 7}, // 2^20 * 3 * 5 * 67 + 1
    {1051721729, 6}, // 2^20 * 17 * 59 + 1
    {1045430273, 3}, // 2^20 * 997 + 1
    {1012924417, 5}, // 2^21 * 3 * 7 * 23 + 1
    {1007681537, 3}, // 2^20 * 31^2 + 1
    {1004535809, 3}, // 2^21 * 479 + 1
    {998244353, 3},  // 2^23 * 7 * 17 + 1
    {985661441, 3},  // 2^22 * 5 * 47 + 1
    {976224257, 3},  // 2^20 * 7^2 * 19 + 1
    {975175681, 17}, // 2^21 * 3 * 5 * 31 + 1
    {962592769, 7},  // 2^21 * 3^3 * 17 + 1
    {950009857, 7},  // 2^21 * 4 * 151 + 1
    {943718401, 7},  // 2^22 * 3^2 * 5^2 + 1
    {935329793, 3},  // 2^22 * 223 + 1
    {924844033, 5},  // 2^21 * 3^2 * 7^2 + 1
    {469762049, 3},  // 2^26 * 7 + 1
    {167772161, 3},  // 2^25 * 5 + 1
};
// }}}

// general math {{"{{"}}{
ll extgcd(ll a, ll b, ll &x, ll &y) {
  ll d;
  return b == 0 ? (x = 1, y = 0, a) : (d = extgcd(b, a % b, y, x), y -= a / b * x, d);
}
ll modinv(ll a, ll mod) {
  ll x, y;
  extgcd(a, mod, x, y);
  x %= mod;
  return x < 0 ? x + mod : x;
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
// }}}

// NTT Core {{"{{"}}{
template < int MAX_H >
struct Pool {
  static ll *tmp, *A, *B;
};
template < int MAX_H >
ll *Pool< MAX_H >::tmp = new ll[1 << MAX_H];
template < int MAX_H >
ll *Pool< MAX_H >::A = new ll[1 << MAX_H];
template < int MAX_H >
ll *Pool< MAX_H >::B = new ll[1 << MAX_H];

template < int MAX_H, ll mod, ll primitive >
class Core {
public:
  static_assert((mod & ((1 << MAX_H) - 1)) == 1, "mod is too small; comment out");
  // ord zetaList[i] = 2^(i + 1)
  ll zetaList[MAX_H], zetaInvList[MAX_H];
  // constexpr
  Core() {
    zetaList[MAX_H - 1] = modpow(primitive, (mod - 1) / (1 << MAX_H), mod);
    zetaInvList[MAX_H - 1] = modinv(zetaList[MAX_H - 1], mod);
    for(int ih = MAX_H - 2; ih >= 0; --ih) {
      zetaList[ih] = zetaList[ih + 1] * zetaList[ih + 1] % mod;
      zetaInvList[ih] = zetaInvList[ih + 1] * zetaInvList[ih + 1] % mod;
    }
  }
  void fft(ll *a, uint n, uint nh, bool inverse) const {
    ll *tmp = Pool< MAX_H >::tmp;
    uint mask = n - 1;
    for(uint i = n >> 1, ih = nh - 1; i >= 1; i >>= 1, --ih) {
      ll zeta = inverse ? zetaInvList[nh - 1 - ih] : zetaList[nh - 1 - ih];
      ll powZeta = 1;
      for(uint j = 0; j < n; j += i) {
        for(uint k = 0; k < i; ++k) {
          tmp[j | k] =
              (a[((j << 1) & mask) | k] + powZeta * a[(((j << 1) | i) & mask) | k]) % mod;
        }
        powZeta = powZeta * zeta % mod;
      }
      swap(a, tmp);
    }
    if(nh & 1) {
      swap(a, tmp);
      for(uint i = 0; i < n; ++i) a[i] = tmp[i];
    }
    if(inverse) {
      ll invN = modinv(n, mod);
      for(uint i = 0; i < n; ++i) a[i] = a[i] * invN % mod;
    }
  }
  vector< ll > conv(const vector< ll > &a, const vector< ll > &b) const {
    uint t = a.size() + b.size() - 1;
    uint n = 1, nh = 0;
    while(n < t) n <<= 1, ++nh;
    return convStrict(a, b, n, nh);
  }
  vector< ll > convStrict(const vector< ll > &a, const vector< ll > &b, uint n,
                          uint nh) const {
    ll *A = Pool< MAX_H >::A, *B = Pool< MAX_H >::B;
    for(uint i = 0; i < n; ++i) A[i] = B[i] = 0;
    copy(a.begin(), a.end(), A);
    copy(b.begin(), b.end(), B);
    fft(A, n, nh, 0), fft(B, n, nh, 0);
    for(uint i = 0; i < n; ++i) A[i] = A[i] * B[i] % mod;
    fft(A, n, nh, 1);
    return vector< ll >(A, A + n);
  }
};
// }}}

// Convolution With Garner {{"{{"}}{
template < int MAX_H, int I >
class ConvolutionWithGarnerCore {
public:
  static void conv_for(uint n, uint nh, const vector< ll > &a, const vector< ll > &b,
                       vector< ll > &mods, vector< ll > &coeffs,
                       vector< vector< ll > > &constants) {
    static const Core< MAX_H, NTT_PRIMES[I][0], NTT_PRIMES[I][1] > ntt;
    auto c = ntt.convStrict(a, b, n, nh);
    mods[I] = NTT_PRIMES[I][0];
    ConvolutionWithGarnerCore< MAX_H, I - 1 >::conv_for(
        n, nh, a, b, mods, coeffs, constants);
    // garner
    for(size_t i = 0; i < c.size(); ++i) {
      ll v = (c[i] - constants[I][i]) * modinv(coeffs[I], mods[I]) % mods[I];
      if(v < 0) v += mods[I];
      for(size_t j = I + 1; j < mods.size(); ++j) {
        constants[j][i] = (constants[j][i] + coeffs[j] * v) % mods[j];
      }
    }
    for(size_t j = I + 1; j < mods.size(); ++j) {
      coeffs[j] = (coeffs[j] * mods[I]) % mods[j];
    }
  }
};

template < int MAX_H >
class ConvolutionWithGarnerCore< MAX_H, -1 > {
public:
  static void conv_for(uint, uint, const vector< ll > &, const vector< ll > &,
                       vector< ll > &, vector< ll > &, vector< vector< ll > > &) {}
};

template < int MAX_H >
class ConvolutionWithGarner {
public:
  template < int USE >
  static vector< ll > conv(const vector< ll > &a, const vector< ll > &b, ll mod) {
    static_assert(USE >= 1, "USE must be positive");
    static_assert(USE <= sizeof(NTT_PRIMES) / sizeof(*NTT_PRIMES), "USE is too big");
    uint nt = a.size() + b.size() - 1;
    uint n = 1, nh = 0;
    while(n < nt) n <<= 1, ++nh;
    vector< ll > coeffs(USE + 1, 1);
    vector< vector< ll > > constants(USE + 1, vector< ll >(n));
    vector< ll > mods(USE + 1, mod);
    ConvolutionWithGarnerCore< MAX_H, USE - 1 >::conv_for(
        n, nh, a, b, mods, coeffs, constants);
    return constants.back();
  }
};

// }}}

} // namespace NTT
// }}}

// 1st param is MAX_H
NTT::Core< 18, NTT::NTT_PRIMES[0][0], NTT::NTT_PRIMES[0][1] > nttBig;
NTT::Core< 18, 998244353, 5 > ntt;
using nttconv = NTT::ConvolutionWithGarner< 18 >;
// nttconv::conv< USE >(a, b, mod)
```


コード中の `NTT_PRIMES` が素数であること，原始根が正しいことを確認した

素数，原始根判定，素因数分解については [数学関係のライブラリ全般]({{ "math/general" | absolute_url }}) の中においてある

# 2次元NTT

[2次元FFT]({{ "math/FFT/FFT2" | absolute_url }}) とアイデアは変わりません．実装もそっちにおいてあります

# 検証

* [C - 高速フーリエ変換 - AtCoder](https://beta.atcoder.jp/contests/atc001/submissions/3370462){:target="_blank"}<!--_-->
* Garnerのアルゴリズムによる復元 [C - 高速フーリエ変換 - AtCoder](https://beta.atcoder.jp/contests/atc001/submissions/3370425){:target="_blank"}<!--_-->

# 練習問題

[基本的なFFT]({{ "math/FFT/standard" | absolute_url }}) にまとめてあります

# 参考

* [任意modでの畳み込み演算をO(n log(n))で - math314のブログ](http://kirika-comp.hatenablog.com/entry/2017/12/18/143923){:target="_blank"}<!--_-->
* [第２回　可換環の定義 - Pythonで学ぶ「プログラミング可換環論」](http://groebner-basis.hatenadiary.jp/entry/2017/12/02/000043){:target="_blank"}<!--_-->

