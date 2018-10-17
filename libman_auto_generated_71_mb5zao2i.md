---
layout: lib
title: 有理数ライブラリ
permalink: misc/Rational

---


整数 / 整数 の形で有理数を保持することで欠損なく値を保持できる．分数

# boost


```cpp
#include <boost/rational.hpp>
using rational = boost::rational< ll >;
```


# 自作

別にboostに似せたりはしていない


```cpp
/// --- Rational Library {{"{{"}}{ ///

#include <iostream>
#include <utility>
template < class Integer = long long >
struct Rational {
  static Integer gcd(Integer a, Integer b) {
    return b == 0 ? a : gcd(b, a % b);
  }
  Integer numer, denom;
  Rational(Integer numer = 0, Integer denom = 1) : numer(numer), denom(denom) {
    reduce();
  }
  void reduce() {
    assert(denom != 0);
    Integer g = gcd(numer, denom);
    numer /= g, denom /= g;
    if(denom < 0) numer = -numer, denom = -denom;
  }
  template < class T >
  explicit operator T() {
    return T(T(numer) / T(denom));
  }
  Rational operator+() const { return *this; }
  Rational operator-() const { return Rational(-numer, denom); }
  // Rational <arithmetic operator> Rational {{"{{"}}{
  Rational operator+(const Rational &a) const {
    return Rational(a.numer * denom + numer * a.denom, denom * a.denom);
  }
  Rational operator-(const Rational &a) const { return *this + -a; }
  Rational operator*(const Rational &a) const {
    return Rational(numer * a.numer, denom * a.denom);
  }
  Rational operator/(const Rational &a) const {
    return Rational(numer * a.denom, denom * a.numer);
  }
  Rational &operator+=(const Rational &a) {
    *this = *this + a;
    return *this;
  }
  Rational &operator-=(const Rational &a) {
    *this = *this - a;
    return *this;
  }
  Rational &operator*=(const Rational &a) {
    *this = *this * a;
    return *this;
  }
  Rational &operator/=(const Rational &a) {
    *this = *this / a;
    return *this;
  }
  // }}}
  // Rational <arithmetic operator> Integer {{"{{"}}{
  Rational operator+(Integer a) const { return *this + Rational(a); }
  Rational operator-(Integer a) const { return *this + Rational(-a); }
  Rational operator*(Integer a) const { return *this * Rational(a); }
  Rational operator/(Integer a) const { return *this * Rational(1, a); }
  Rational &operator+=(Integer a) {
    *this = *this + a;
    return *this;
  }
  Rational &operator-=(Integer a) {
    *this = *this - a;
    return *this;
  }
  Rational &operator*=(Integer a) {
    *this = *this * a;
    return *this;
  }
  Rational &operator/=(Integer a) {
    *this = *this / a;
    return *this;
  }
  // }}}
  Rational inverse() const {
    assert(numer != 0);
    return Rational(denom, numer);
  }
  /// Rational <comparison operator> Rational {{"{{"}}{
  bool operator==(const Rational &a) const {
    return numer == a.numer && denom == a.denom;
  }
  bool operator<(const Rational &a) const {
    return numer * a.denom < a.numer * denom;
  }
  bool operator!=(const Rational &a) const {
    return rel_ops::operator!=(*this, a);
  }
  bool operator>(const Rational &a) const {
    return rel_ops::operator>(*this, a);
  }
  bool operator<=(const Rational &a) const {
    return rel_ops::operator<=(*this, a);
  }
  bool operator>=(const Rational &a) const {
    return rel_ops::operator>=(*this, a);
  }
  // }}}
  /// Rational <comparison operator> Integer {{"{{"}}{
  bool operator==(Integer a) const { return *this == Rational(a); }
  bool operator<(Integer a) const { return *this < Rational(a); }
  bool operator!=(Integer a) const { return *this != Rational(a); }
  bool operator>(Integer a) const { return *this > Rational(a); }
  bool operator<=(Integer a) const { return *this <= Rational(a); }
  bool operator>=(Integer a) const { return *this >= Rational(a); }
  // }}}
  friend ostream &operator<<(ostream &os, const Rational &a) {
    os << a.numer << "/" << a.denom;
    return os;
  }
  Integer bunsi() const { return numer; }
  Integer bunbo() const { return denom; }
};

/// }}}--- ///
typedef Rational<> rational;
```


分母が必ず正になるように保持

各種比較・算術演算が可能

コンストラクタ : Rational(Integer numer = 0, Integer denom = 1)

一項演算 : `+`, `-`, 明示的な `(cast)`  
二項演算 : `+`, `-`, `*`, `/`, `+=`, `-=`, `*=`, `/=`  
右辺の対象は `Rational<Integer>` クラスか `Integer` にキャスト可能なもの  

分子取得: `.numer` or `.bunsi()`  
分母取得: `.denom` or `.bunbo()`  

約分 (正規化) : `.reduce()`  
`.numer` か `.denom` を自分で変更したときのみ

# 検証

* いくつかの演算子 [E - Game of +- (800) - AtCoder](https://beta.atcoder.jp/contests/code-festival-2018-qualb/submissions/3411192){:target="_blank"}<!--_-->

