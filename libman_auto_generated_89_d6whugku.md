---
layout: lib
title: ローリングハッシュ法 (Rabin-Karp法)
permalink: string/Rolling-Hash

---


各文字を数字にして，$B$進法と解釈して数値にします

**比較を定数で行える文字列**，といった扱い方ができます

ローリングハッシュ法を使った文字列検索がRabin-Karp法です

modを$M$，文字列の長さを$N$とし，$M$は文字種より十分大きいとします．$k$を $(B, M)$ のペアの数とします

通常はハッシュが一致したら実際の値が一致するか調べるんですが，競プロでは調べて得するケースはあまりないと思います

# 衝突しやすいmodに注意

[hosさんの解説](http://hos.ac/blog/#blog0003){:target="_blank"}<!--_--> がとてもわかりやすいです

$M$が$2^64$の約数では危ない．たった4096文字のhackがある

$M$ に**素数**を用いることで衝突する確率が $N/M$ ぐらいになる (多項式と根の数が関係しています)．$B$ は一様ランダムに選べば何でもよし．0はダメだと思うけど

実際は衝突する確率を小さくするために $(B, M)$ の組み合わせを複数用意します

別に素数という制約がある $M$ をたくさん用意する必要はないと思うので $B$ をいくつか用意するというものでいいと思います

現実的に $10^6$ 回程度の比較，$N \approx 10^6, M \approx 10^9$ だとすると，$B$を3つ用意すれば成功確率 `99.90%`となり十分です ([計算に用いたファイル - gist](https://gist.github.com/LumaKernel/709d69a5788c1bf333f5bb7df3777c95){:target="_blank"}<!--_-->)

# 実装

一応ライブラリとして書いてみました


```cpp
// mystring( <string-like> )
// .shift(ll n)
// .push_front(c)
// .changeAt(index, from, to)
// [ll index] -= , +=
/// --- HashString with rolling-hash {{"{{"}}{ ///

// #include <functional>
#include <iterator>
#include <string>
#include <vector>

template < int N, int mod, int* B, int SZ >
class HashString {
  static_assert(N >= 1, "N must be positive");

  class PowTable {
  public:
    int table[SZ][(N + 1) * 2 + 1];
    PowTable() {
      for(int i = 0; i < SZ; i++) {
        table[i][N] = 1;
        for(int j = 1; j <= N; j++) {
          table[i][j + N] = (ll) table[i][j - 1 + N] * B[i] % mod;
        }
        table[i][N - 1] = modinv(B[i]);
        for(int j = -2; j >= -N; j--) {
          table[i][j + N] = (ll) table[i][j + 1 + N] * table[i][N - 1] % mod;
        }
      }
    }
  };
  static PowTable table;

public:
  static int modpow(int x, ll y) {
    int r = 1;
    if(y < 0) y = -y, x = modinv(x);
    while(y) {
      if(y & 1) r = (ll) r * x % mod;
      x = (ll) x * x % mod;
      y >>= 1;
    }
    return r;
  }
  static int extgcd(int a, int b, int& x, int& y) {
    int d;
    return b == 0 ? (x = a < 0 ? -1 : 1, y = 0, a < 0 ? -a : a)
                  : (d = extgcd(b, a % b, y, x), y -= a / b * x, d);
  }
  static int modinv(int a) {
    int x, y;
    extgcd(a, mod, x, y);
    if(x < mod) x += mod;
    return x;
  }

  int hashed[SZ] = {};

  HashString() {}
  HashString(const char* str) : HashString(str, str + char_traits< char >::length(str)) {}
  HashString(const string& str) : HashString(str.begin(), str.end()) {}
  template < class T >
  HashString(const vector< T >& v) : HashString(v.begin(), v.end()) {}

  template < class InputIter, class = typename iterator_traits< InputIter >::value_type >
  HashString(const InputIter& first, const InputIter& last) {
    for(int i = 0; i < SZ; i++) {
      ll tmp = 1;
      for(InputIter now = first; now != last; ++now, tmp = tmp * B[i] % mod) {
        hashed[i] = (hashed[i] + tmp * ll(*now) % mod) % mod;
      }
    }
  }

  class Accessor {
    const ll n;
    HashString& str;

  public:
    Accessor(ll n, HashString& str) : n(n), str(str) {}
    template < class T >
    void operator+=(T c) {
      for(ll i = 0; i < SZ; i++) {
        ll bn = -N <= n && n <= N ? table.table[i][n + N] : modpow(B[i], n);
        str.hashed[i] = (str.hashed[i] + bn * ll(c) % mod + mod) % mod;
      }
    }
    template < class T >
    void operator-=(T c) {
      *this += -ll(c);
    }
    void operator+=(const HashString& a) {
      for(ll i = 0; i < SZ; i++) {
        ll bn = -N <= n && n <= N ? table.table[i][n + N] : modpow(B[i], n);
        str.hashed[i] = (str.hashed[i] + bn * a.hashed[i] % mod + mod) % mod;
      }
    }
    void operator-=(const HashString& a) {
      for(ll i = 0; i < SZ; i++) {
        ll bn = -N <= n && n <= N ? table.table[i][n + N] : modpow(B[i], n);
        str.hashed[i] = (str.hashed[i] - bn * a.hashed[i] % mod + mod) % mod;
      }
    }
  };

  Accessor operator[](ll i) { return Accessor(i, *this); }

  void shift(ll n = 1) {
    for(int i = 0; i < SZ; i++) {
      ll bn = -N <= n && n <= N ? table.table[i][n + N] : modpow(B[i], n);
      hashed[i] = (ll) hashed[i] * bn % mod;
    }
  }

  bool operator==(const HashString& a) const {
    for(int i = 0; i < SZ; i++) {
      if(hashed[i] != a.hashed[i]) return 0;
    }
    return 1;
  }

  HashString& operator+=(const HashString& a) {
    for(int i = 0; i < SZ; i++) {
      hashed[i] += a.hashed[i];
      if(hashed[i] >= mod) hashed[i] -= mod;
    }
    return *this;
  }

  HashString& operator-=(const HashString& a) {
    for(int i = 0; i < SZ; i++) {
      hashed[i] += mod - a.hashed[i];
      if(hashed[i] >= mod) hashed[i] -= mod;
    }
    return *this;
  }

  HashString operator+(const HashString& a) const {
    HashString tmp = *this;
    tmp += a;
    return tmp;
  }

  HashString operator-(const HashString& a) const {
    HashString tmp = *this;
    tmp -= a;
    return tmp;
  }

  template < class T >
  void push_front(const T& a) {
    for(int i = 0; i < SZ; i++) {
      hashed[i] = (ll) hashed[i] * B[i] % mod;
      hashed[i] = (hashed[i] + ll(a)) % mod;
    }
  }

  template < class T >
  void changeAt(ll n, T a, T b) {
    for(int i = 0; i < SZ; i++) {
      ll bn = -N <= n && n <= N ? table.table[i][n + N] : modpow(B[i], n);
      hashed[i] = (hashed[i] + (mod + ll(b) - ll(a)) * bn % mod) % mod;
    }
  }

  friend ostream& operator<<(ostream& os, const HashString& a) {
    // HASH_STRING{, }
    os << "HASH_STRING{" << a.hashed[0];
    for(int i = 1; i < SZ; i++) {
      os << ", " << a.hashed[i];
    }
    os << "}";
    return os;
  }
};
template < int N, int mod, int* B, int SZ >
typename HashString< N, mod, B, SZ >::PowTable HashString< N, mod, B, SZ >::table;

constexpr int _hs_N = 1e6;

int _hs_B[] = {
    114514,
    1919,
    810,
};

using mystring =
    HashString< _hs_N, 1'000'000'007, _hs_B, sizeof(_hs_B) / sizeof(_hs_B[0]) >;

namespace std {
template < int N, int mod, int* B, int SZ >
struct hash< HashString< N, mod, B, SZ > > {
  template < class T >
  static void hash_combine(size_t& seed, T const& v) {
    seed ^= hash< T >{}(v) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
  }
  size_t operator()(const HashString< N, mod, B, SZ >& a) {
    size_t seed = 0;
    for(int i = 0; i < SZ; i++) {
      hash_combine(seed, a.hashed[i]);
    }
    return seed;
  }
};
} // namespace std

/// }}}--- ///
```


## 使い方

実際の文字列の情報は (長さ含め) 保持しません

* コンストラクタ `mystring( <string-like> )` : イテレータや`string`, `vector` などから `mystring` に変換できます
* `push_front(T c)` : 1文字シフトして0文字目に文字`c`を挿入
* `shift(ll n = 1)` : n文字シフトする．負でも可
* `changeAt(ll x, T from, T to)` : x文字目をfromからtoに変える (fromは自分で用意する)
* `str[ll x] -= `, `str[ll x] += ` : x文字目から引いたり足したり．負でも可

`unordered_set` や `unordered_map` の `Key` として用いることができます

もとの文字列保持しててもいいなと思ったんですが，コピーも $O(1)$ にしようみたいなどうでもいいことのために

すべて $O(k\log M)$ 時間, 比較は $O(k)$

# 検証

した (主張)．OJにはまだsubmitしていない TODO

# 参考

* 蟻本

