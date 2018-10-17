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

mod 1e9+7 といったような任意modについては [中国剰余定理 (CRT)]({{ "math/CRT" | absolute_url }}) などで復元する必要がある

以下の実装では `NTT::conv< USE > (vector<ll> a, vector<ll> b, ll m)` を使うと `USE` 個のmod上で値を出して [Garnerのアルゴリズム]({{ "algorithm/Garner" | absolute_url }}) を用いて mod m に復元を行う

NTTを行うmodを $m_1, m_2, \cdots, m_{USE}$ とすると，厳密値 (予めmodを取った場合はそれを厳密値とする) の最大値 (mod $m$に復元したいとした時，$m^2\min(N, M)$ ぐらい) を $m_1m_2\cdots m_{USE}$ が超えるように取らなければならない

## 求めたい値が厳密な値のとき (64bitにおさまるとき)

基本的にmod上で求めるときと同じように，最大値より大きな値をmodとすればよい

掛け算でoverflowする可能性がある．これを防ぐには，**int128などで掛け算を行うか**，
[数学>基本]({{ "math/general" | absolute_url }}) にあるmodmulなどがある (このような状況はたまにあるだろうが，特別な実装はしていない (ジャッジによるところもあるかもしれない))


## 求めたい値が厳密な値の時 (64bitに収まらない時)

復元時にint128, int256など多倍長整数使うしか無い

# 実装


```cpp
constexpr int MAX_H = 18;

// MAX_N is max size of OUTPUT, DOUBLED INPUT
// MAX_RES_VALUE = MAX_VALUE^2 * MAX_N
// if MAX_N > 2^20, comment out primes!
/// --- NTT Library {{"{{"}}{ ///
namespace NTT {
ll extgcd(ll a, ll b, ll &x, ll &y) {
  ll d;
  return b == 0 ? (x = 1, y = 0, a)
                : (d = extgcd(b, a % b, y, x), y -= a / b * x, d);
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
template < ll mod, ll primitive, int MAX_H >
struct Core {
  static_assert((mod & ((1 << MAX_H) - 1)) == 1,
                "mod is too small; comment out");
  using uint = uint_fast32_t;
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
  void fft(vector< ll > &a, uint n, uint h, bool inv) const {
    static vector< ll > tmp;
    tmp.resize(n);
    uint mask = n - 1;
    for(uint i = n >> 1, ih = h - 1; i >= 1; i >>= 1, --ih) {
      ll zeta = inv ? zetaInvList[h - 1 - ih] : zetaList[h - 1 - ih];
      ll powZeta = 1;
      for(uint j = 0; j < n; j += i) {
        for(uint k = 0; k < i; ++k) {
          tmp[j + k] = (a[((j << 1) & mask) + k] +
                        powZeta * a[(((j << 1) + i) & mask) + k]) %
                       mod;
        }
        powZeta = powZeta * zeta % mod;
      }
      swap(a, tmp);
    }
    if(inv) {
      ll invN = modinv(n, mod);
      for(uint i = 0; i < n; ++i) a[i] = a[i] * invN % mod;
    }
  }
  vector< ll > conv(const vector< ll > &a, const vector< ll > &b) const {
    assert(a.size() + b.size() - 1 <= (1 << MAX_H));

    if(a.size() == 0) return {};
    if(b.size() == 0) return {};
    return _conv(a, b);
  }
  vector< ll > _conv(vector< ll > a, vector< ll > b) const {
    uint deg = a.size() + b.size() - 1;
    uint h = 0, n = 1;
    while(n < deg) n <<= 1, ++h;
    a.resize(n), b.resize(n);
    return _convStrict(a, b, n, h);
  }
  vector< ll > _convStrict(vector< ll > a, vector< ll > b, uint n,
                           uint h) const {
    fft(a, n, h, 0), fft(b, n, h, 0);
    for(uint i = 0; i < n; ++i) a[i] = a[i] * b[i] % mod;
    fft(a, n, h, 1);
    return a;
  }
};
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
template < int I >
void conv_for(int n, int h, const vector< ll > &a, const vector< ll > &b,
              vector< ll > &mods, vector< ll > &coeffs,
              vector< vector< ll > > &constants) {
  static const Core< NTT_PRIMES[I][0], NTT_PRIMES[I][1], MAX_H > ntt;
  auto c = ntt._convStrict(a, b, n, h);
  mods[I] = NTT_PRIMES[I][0];
  conv_for< I - 1 >(n, h, a, b, mods, coeffs, constants);
  // garner
  for(size_t i = 0; i < c.size(); ++i) {
    ll v = (c[i] - constants[I][i]) * modinv(coeffs[I], mods[I]) % mods[I];
    if(v < 0) v += mods[i];
    for(size_t j = I + 1; j < mods.size(); ++j) {
      constants[j][i] = (constants[j][i] + coeffs[j] * v) % mods[j];
    }
  }
  for(size_t j = I + 1; j < mods.size(); ++j) {
    coeffs[j] = (coeffs[j] * mods[I]) % mods[j];
  }
}

template <>
void conv_for< -1 >(int, int, const vector< ll > &, const vector< ll > &,
                    vector< ll > &, vector< ll > &, vector< vector< ll > > &) {}

template < int USE >
vector< ll > conv(vector< ll > a, vector< ll > b, ll mod) {
  static_assert(USE >= 1, "USE must be positive");
  static_assert(
      USE <= sizeof(NTT_PRIMES) / sizeof(*NTT_PRIMES), "USE is too big");
  int deg = a.size() + b.size() - 1;
  int n = 1, h = 0;
  while(n < deg) n <<= 1, ++h;
  a.resize(n), b.resize(n);
  vector< ll > coeffs(USE + 1, 1);
  vector< vector< ll > > constants(USE + 1, vector< ll >(n, 0));
  vector< ll > mods(USE + 1, mod);
  conv_for< USE - 1 >(n, h, a, b, mods, coeffs, constants);
  return constants.back();
}
} // namespace NTT
/// }}}--- ///

NTT::Core< NTT::NTT_PRIMES[0][0], NTT::NTT_PRIMES[0][1], MAX_H > nttBig;
NTT::Core< 924844033, 5, MAX_H > ntt;
// NTT::conv< USE >(a, b, mod)
```


コード中の `NTT_PRIMES` が素数であること，原始根が正しいことを確認した

原始根判定については [数学関係のライブラリ全般]({{ "math/general" | absolute_url }}) の中においてある

# 検証

* [C - 高速フーリエ変換 - AtCoder](https://beta.atcoder.jp/contests/atc001/submissions/3370462){:target="_blank"}<!--_-->
* Garnerのアルゴリズムによる復元 [C - 高速フーリエ変換 - AtCoder](https://beta.atcoder.jp/contests/atc001/submissions/3370425){:target="_blank"}<!--_-->

# 練習問題

[基本的なFFT]({{ "math/FFT/standard" | absolute_url }}) にまとめてあります

# 参考

* [任意modでの畳み込み演算をO(n log(n))で - math314のブログ](http://kirika-comp.hatenablog.com/entry/2017/12/18/143923){:target="_blank"}<!--_-->
* [第２回　可換環の定義 - Pythonで学ぶ「プログラミング可換環論」](http://groebner-basis.hatenadiary.jp/entry/2017/12/02/000043){:target="_blank"}<!--_-->

