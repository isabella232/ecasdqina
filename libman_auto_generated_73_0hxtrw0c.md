---
layout: lib
title: 2次元FFT
permalink: math/FFT/FFT2

---


[2変数の多項式](https://ja.wikipedia.org/wiki/多変数多項式){:target="_blank"}<!--_--> $f(x, y) = \sum\_{i,j} a\_{i, j}x^i y^j$ 2つをかけ合わせます 
係数 $a$ は実数，$N \times M$ の行列 ( = 2次元配列 ) とします

# 要約

* 1次元のDFT, 畳み込みをそのまま2次元に自然に拡張できる
* 2次元FFTは1次元FFTを各行に行った後，各列に行うだけ．IFFTも同様

# 2次元DFT

2次元DFTは

$$\hat f (x, y) = \mathrm{DFT}_{N, M} (f) (x, y) = \sum_{i = 0}^{N-1} \sum_{j = 0}^{M-1} f(\zeta_N^i, \zeta_M^j) x^i y^j$$

と定義され，2次元IDFTは

$$\mathrm{DFT}_{N, M}^{-1} (f) (x, y) = \frac{1}{NM} \sum_{i = 0}^{N-1} \sum_{j = 0}^{M-1} \hat f(\zeta\_N^{-i}, \zeta_M^{-j}) x^i y^j$$

### 係数による表示

証明に移る前に，係数での表示がわかりやすいので，係数行列 $a$ とそれをDFTした $A = \mathrm{DFT}(a)$ を定義する

#### 1次元の場合

通常のDFTを係数で表示すると以下のようになる

$$A_i = \sum_{j=0}^{N-1} a_j \zeta_N^{ij}$$

$$a_k = \frac{1}{N} \sum_{i=0}^{N-1} A_i \zeta_N^{-jk}$$

#### 2次元の場合

2次元DFTの定義より，以下のようになる

$$\text{(DFT) } A_{i, j} = \sum_{k=0}^{N-1} \sum_{l=0}^{M-1} a_{k, l} \zeta_N^{ik} \zeta_M^{jl}$$

$$\text{(IDFT) } a_{m, n} = \frac{1}{NM} \sum_{i=0}^{N-1} \sum_{j=0}^{M-1} A_{i, j} \zeta_N^{-im}\zeta_M^{-jn}$$

### 証明

**DFTの定義を仮定**して**IDFTの定義が成り立つ**ことを証明する． IDFTの式にDFTの式を代入する

$$
\begin{aligned}
&&&(\text{IDFTの右辺}) &&\\
&&=& \frac{1}{NM} \sum_{i, j, k, l} a_{k, l} \times \zeta_N^{i(k-m)} \times \zeta_M^{j(l-n)} && \text{(ループ範囲は省略している)}\\
&&=& \frac{1}{NM} \sum_{k, l} a_{k, l} \sum_i \zeta_N^{i(k-m)} \sum_j \zeta_M^{j(l-n)} && \text{(シグマを潜り込ませた)}\\
&&=& \frac{1}{NM} \sum_{k, l} a_{k, l}
\left(\begin{cases}
N & (k = m)\\
0 & (k \not= m)\\
\end{cases}\right)
\left(\begin{cases}
M & (l = n)\\
0 & (l \not= n)\\
\end{cases}\right)
&& (\spadesuit : m,n\text{は定数})\\
&&=\ & a_{m, n} = (\text{IDFTの左辺})\\
\end{aligned}
$$

$(\spadesuit)$ の行での式変形については [FFT導入]({{ "FFT/introduction" | absolute_url }}) で説明している

# 2次元畳み込み

$$(f * g) = \mathrm{DFT}_{N, M}^{-1} (\mathrm{DFT}_{N, M} (f) \cdot \mathrm{DFT}_{N, M} (g))$$

なお，多項式同士の演算 $\cdot$ は係数ごとの掛け算 (component-wise multiplication)

### 証明

$f, g, (f*g)$ のそれぞれの係数行列を$a, b, c$ とし，それらのDFTをそれぞれ $A, B, C$ とする

$$
\begin{aligned}
\widehat{(f * g)}(x, y)  
&&=& \sum_{i, j} (f * g) (\zeta_N^i, \zeta_M^j)x^iy^j && (\text{DFTの定義より})\\
&&=& \sum_{i, j} f(\zeta_N^i, \zeta_M^j)g(\zeta_N^i, \zeta_M^j)x^iy^j && (\text{畳み込みの定義より})\\
&&=& \sum_{i, j} A_{i, j} B_{i, j} x^iy^j \\
\end{aligned}
$$

ところで，$$\widehat{(f * g)}(x, y) = \sum_{i, j} C_{i, j}x^i y^j$$ より，  
$$C_{i, j} = A_{i, j} B_{i, j}$$ が言えます

これを多項式での表示に戻すと上記の畳込みの式となります

# 2次元FFT

係数行列 $a$ を $N \times M$ の行列とし，$N, M$ は2冪とする

1次元のDFTにより2次元のDFTを計算できる

各行，各列に対して1次元の係数列とみて$N + M$ 回のFFTをすればよい．IDFTも同様

計算量は $O(NM \log NM)$

### 証明

$a$ の各列をDFTしたものを $$a'$$，$$a'$$ の各行をFFTしたものを $$a''$$ とする

わかりやすくするために，1次元DFTに色付けする

$$(\mathrm{DFT}_\textcolor{green}N(a))_{\textcolor{red}i} = \sum_{\textcolor{blue}j=0}^{\textcolor{green}N-1} a_\textcolor{blue}j \zeta_\textcolor{green}N^{\textcolor{red}i\textcolor{blue}j}$$

以下の式変形では，対応する文字を同じ色にした

$$
\begin{aligned}

&A_{i, j}
&=& \sum_{k=0}^{N-1} \sum_{l=0}^{M-1} a_{k, l} \zeta_N^{ik} \zeta_M^{jl} \\

&A_{\textcolor{red}i, j}
&=& \sum_{l=0}^{M-1}
\left( \sum_{\textcolor{blue}k=0}^{\textcolor{green}N-1} a_{\textcolor{blue}k, l} \zeta_\textcolor{green}N^{\textcolor{red}i\textcolor{blue}k} \right)
\zeta_M^{jl}
&& (\textcolor{green}N\text{に注目})\\

&
&=& \sum_{l=0}^{M-1}
a'_{i, l}
\zeta_M^{jl}
&& (a'\text{の定義より})\\

&A_{i, \textcolor{red}j}
&=& \sum_{\textcolor{blue}l=0}^{\textcolor{green}M-1}
a'_{i, \textcolor{blue}l}
\zeta_\textcolor{green}M^{\textcolor{red}j\textcolor{blue}l}
&& (\textcolor{green}M\text{に注目})\\

&
&=\ &
a''_{i, j}
&& (a''\text{の定義より})\\

\end{aligned}
$$

よって $A$ と $$a''$$ が同じ行列であることが示された

行，列の順にDFTした場合も同様．IDFTも同様

---

無理やりラムダ式で書くとこんな感じ

$$
a'_{i, j} = (\mathrm{DFT}_N(\lambda y \rightarrow a_{y,j}))_i
\\
a''_{i, j} = (\mathrm{DFT}_M(\lambda x \rightarrow a'_{i,x}))_j
$$

# 実装

[複素数によるFFT]({{ "math/FFT/standard" | absolute_url }}) が必要

畳み込みについては [DFTを同時に求める高速化]({{ "math/FFT/standard#畳み込み高速化" | absolute_url }}) をしています


```cpp
// require FFT standard
/// --- FFT2 {{"{{"}}{ ///

template < int MAX_N, int MAX_NM, class Real, class Complex = complex< Real > >
class fft2_base : public fft_base< MAX_N, Real, Complex > {
  // semi-private
  template < class T >
  static void transpose(T *a, size_t n, size_t m, size_t mh) {
    // assert(m == (1u << mh));
    if(n == m) {
      for(size_t i = 0; i < n - 1; ++i)
        for(size_t j = i + 1; j < m; ++j) swap(a[(i << mh) | j], a[(j << mh) | i]);
    } else {
      static T *tmp = new T[MAX_NM];
      for(size_t i = 0; i < n * m; ++i) tmp[i] = a[i];
      for(size_t i = 0; i < n; ++i)
        for(size_t j = 0; j < m; ++j) a[(i << mh) | j] = tmp[(j << mh) | i];
    }
  }

  // semi-private
  static void fft2(Complex *a, size_t n, size_t nh, size_t m, size_t mh, bool inverse) {
    // assert(n == (1u << nh));
    // assert(m == (1u << mh));
    for(int t = 0; t < 2; ++t) {
      for(size_t i = 0; i < n * m; i += m) fft(a + i, m, mh, inverse);
      transpose(a, n, m, mh);
      swap(n, m);
      swap(nh, mh);
    }
  }

  static vector< vector< ll > > conv2Fast(const vector< vector< ll > > &ar,
                                          const vector< vector< ll > > &br) {
    int nt = ar.size() + br.size() - 1, mt = ar[0].size() + br[0].size() - 1;
    int n = 1, m = 1, nh = 0, mh = 0;
    while(n < nt) n <<= 1, ++nh;
    while(m < mt) m <<= 1, ++mh;
    static Complex *a = new Complex[MAX_NM], *c = new Complex[MAX_NM];
    for(size_t i = 0; i < n * m; ++i) a[i] = 0;
    for(size_t i = 0; i < ar.size(); ++i)
      for(size_t j = 0; j < ar[0].size(); ++j) a[(i << mh) | j].real(ar[i][j]);
    for(size_t i = 0; i < br.size(); ++i)
      for(size_t j = 0; j < br[0].size(); ++j) a[(i << mh) | j].imag(br[i][j]);
    fft2(a, n, nh, m, mh, 0);
    for(int i = 0; i < n; ++i)
      for(int j = 0; j < m; ++j) {
        int k = i == 0 ? 0 : n - i;
        int l = j == 0 ? 0 : m - j;
        c[(i << mh) | j] = (a[(i << mh) | j] + conj(a[(k << mh) | l])) *
                           (a[(i << mh) | j] - conj(a[(k << mh) | l])) * Complex(0, -.25);
      }
    fft2(c, n, nh, m, mh, 1);
    vector< vector< ll > > cr(n, vector< ll >(m));
    for(int i = 0; i < n; ++i)
      for(int j = 0; j < m; ++j) cr[i][j] = round(c[(i << mh) | j].real());
    return cr;
  }
};

/// }}}--- ///

using fft2 = fft2_base< 1 << 22, 1 << 22, double >;
```


# 2次元NTT

[NTT]({{ "math/FFT/NTT" | absolute_url }}) が必要


```cpp
// require NTT
// NTT2 {{"{{"}}{
namespace NTT {

// NTT2 Core {{"{{"}}{
template < int MAX_NM >
struct Pool2 {
  static ll *tmp, *A, *B;
};
template < int MAX_NM >
ll *Pool2< MAX_NM >::tmp = new ll[MAX_NM];
template < int MAX_NM >
ll *Pool2< MAX_NM >::A = new ll[MAX_NM];
template < int MAX_NM >
ll *Pool2< MAX_NM >::B = new ll[MAX_NM];

// transpose {{"{{"}}{
// semi-private
template < int MAX_NM, class T >
static void transpose(T *a, size_t n, size_t m, size_t mh) {
  // assert(m == (1u << mh));
  ll *tmp = Pool2< MAX_NM >::tmp;
  if(n == m) {
    for(size_t i = 0; i < n - 1; ++i)
      for(size_t j = i + 1; j < m; ++j) swap(a[(i << mh) | j], a[(j << mh) | i]);
  } else {
    for(size_t i = 0; i < n * m; ++i) tmp[i] = a[i];
    for(size_t i = 0; i < n; ++i)
      for(size_t j = 0; j < m; ++j) a[(i << mh) | j] = tmp[(j << mh) | i];
  }
}
// }}}

template < int MAX_H, int MAX_NM, ll mod, ll primitive >
class Core2 {
public:
  void fft2(ll *a, uint n, uint nh, uint m, uint mh, bool inverse) const {
    static const Core< MAX_H, mod, primitive > ntt;
    // assert(n == (1u << nh));
    // assert(m == (1u << mh));
    for(int t = 0; t < 2; ++t) {
      for(uint i = 0; i < n * m; i += m) ntt.fft(a + i, m, mh, inverse);
      transpose< MAX_NM >(a, n, m, mh);
      swap(n, m);
      swap(nh, mh);
    }
  }
  vector< vector< ll > > conv2(const vector< vector< ll > > &a,
                               const vector< vector< ll > > &b) const {
    uint nt = a.size() + b.size() - 1, mt = a[0].size() + b[1].size() - 1;
    assert(nt <= (1 << MAX_H) && mt <= (1 << MAX_H));
    uint n = 1, m = 1, nh = 0, mh = 0;
    while(n < nt) n <<= 1, ++nh;
    while(m < mt) m <<= 1, ++mh;
    return conv2Strict(a, b, n, nh, m, mh);
  }
  vector< vector< ll > > conv2Strict(const vector< vector< ll > > &a,
                                     const vector< vector< ll > > &b, uint n, uint nh,
                                     uint m, uint mh) const {
    ll *A = Pool2< MAX_NM >::A, *B = Pool2< MAX_NM >::B;
    for(uint i = 0; i < n * m; ++i) A[i] = B[i] = 0;
    for(uint i = 0; i < a.size(); ++i) copy(a[i].begin(), a[i].end(), A + (i << mh));
    for(uint i = 0; i < b.size(); ++i) copy(b[i].begin(), b[i].end(), B + (i << mh));
    fft2(A, n, nh, m, mh, 0), fft2(B, n, nh, m, mh, 0);
    for(uint i = 0; i < n * m; ++i) A[i] = A[i] * B[i] % mod;
    fft2(A, n, nh, m, mh, 1);
    vector< vector< ll > > c(n, vector< ll >(m));
    for(uint i = 0; i < n; ++i)
      for(uint j = 0; j < m; ++j) c[i][j] = A[(i << mh) | j];
    return c;
  }
};
// }}}

// Convolution2 With Garner {{"{{"}}{
template < int MAX_H, int MAX_NM, int I >
class Convolution2WithGarnerCore {
public:
  static void conv2_for(uint n, uint nh, uint m, uint mh, const vector< vector< ll > > &a,
                        const vector< vector< ll > > &b, vector< ll > &mods,
                        vector< ll > &coeffs,
                        vector< vector< vector< ll > > > &constants) {
    static const Core2< MAX_H, MAX_NM, NTT_PRIMES[I][0], NTT_PRIMES[I][1] > ntt2;
    auto c = ntt2.conv2Strict(a, b, n, nh, m, mh);
    mods[I] = NTT_PRIMES[I][0];
    Convolution2WithGarnerCore< MAX_H, MAX_NM, I - 1 >::conv2_for(
        n, nh, m, mh, a, b, mods, coeffs, constants);
    // garner
    for(size_t i = 0; i < c.size(); ++i)
      for(size_t j = 0; j < c[0].size(); ++j) {
        ll v = (c[i][j] - constants[I][i][j]) * modinv(coeffs[I], mods[I]) % mods[I];
        if(v < 0) v += mods[I];
        for(size_t k = I + 1; k < mods.size(); ++k) {
          constants[k][i][j] = (constants[k][i][j] + coeffs[k] * v) % mods[k];
        }
      }
    for(size_t j = I + 1; j < mods.size(); ++j) {
      coeffs[j] = (coeffs[j] * mods[I]) % mods[j];
    }
  }
};

template < int MAX_H, int MAX_NM >
class Convolution2WithGarnerCore< MAX_H, MAX_NM, -1 > {
public:
  static void conv2_for(uint, uint, uint, uint, const vector< vector< ll > > &,
                        const vector< vector< ll > > &, vector< ll > &, vector< ll > &,
                        vector< vector< vector< ll > > > &) {}
};

template < int MAX_H, int MAX_NM >
class Convolution2WithGarner {
public:
  template < int USE >
  static vector< vector< ll > > conv2(const vector< vector< ll > > &a,
                                      const vector< vector< ll > > &b, ll mod) {
    static_assert(USE >= 1, "USE must be positive");
    static_assert(USE <= sizeof(NTT_PRIMES) / sizeof(*NTT_PRIMES), "USE is too big");
    uint nt = a.size() + b.size() - 1, mt = a[0].size() + b[0].size() - 1;
    uint n = 1, m = 1, nh = 0, mh = 0;
    while(n < nt) n <<= 1, ++nh;
    while(m < mt) m <<= 1, ++mh;
    vector< ll > coeffs(USE + 1, 1);
    vector< vector< vector< ll > > > constants(
        USE + 1, vector< vector< ll > >(n, vector< ll >(m)));
    vector< ll > mods(USE + 1, mod);
    Convolution2WithGarnerCore< MAX_H, MAX_NM, USE - 1 >::conv2_for(
        n, nh, m, mh, a, b, mods, coeffs, constants);
    return constants.back();
  }
};

// }}}

} // namespace NTT
// }}}

NTT::Core2< 18, 1 << 22, NTT::NTT_PRIMES[0][0], NTT::NTT_PRIMES[0][1] > ntt2Big;
NTT::Core2< 18, 1 << 22, 998244353, 5 > ntt2;
using nttconv2 = NTT::Convolution2WithGarner< 11, 1 << 22 >;
// nttconv2::conv2< USE >(a, b, mod)
```


ほぼ [NTT]({{ "math/FFT/NTT" | absolute_url }}) のコピペなので少し長くなった

# 検証

* FFT2 - [JAG2013Spring - F - Point Distance - AtCoder](https://beta.atcoder.jp/contests/jag2013spring/submissions/3684135){:target="_blank"}<!--_-->
* NTT2 - [JAG2013Spring - F - Point Distance - AtCoder](https://beta.atcoder.jp/contests/jag2013spring/submissions/3686678){:target="_blank"}<!--_-->

# 参考

* [2次元FFT - とどの日記](http://d.hatena.ne.jp/todo314/touch/20130811/1376221445){:target="_blank"}<!--_-->
* [2-D fast Fourier transform - MATLAB](https://jp.mathworks.com/help/matlab/ref/fft2.html?lang=en){:target="_blank"}<!--_-->
* [多変数多項式 - Wikipedia](https://ja.wikipedia.org/wiki/多変数多項式){:target="_blank"}<!--_-->

