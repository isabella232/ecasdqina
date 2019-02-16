---
layout: lib
title: ナップサック問題
permalink: dynamic-programming/knapsack

---


個数制限付きナップサック問題を考えます

# ナップサック問題

荷物の種類 $N$  
重さ : $x_i \leq X$  
価値 : $y_i \leq Y$  
個数 : $1 \leq z_i \leq Z$  

重さの和制限 $W$

全て整数

$i$ 番目を $c_i$ 個選ぶとする

重さの合計に関する条件 $\sum {c_i x_i} \leq W$  
個数に関する条件 $0 \leq c_i \leq z_i$

という条件を満たしながら $\sum c_i y_i$ を最大化せよ


# 概要

問題によると思いますが，価値が0以下のものは全て捨てる，重さが0以下のものは全て採用とする，などで $1 \leq x_i, y_i$ を仮定します (問題によっては両方起こり得たり，$W$以下ではなく**ちょうど**$W$，といったこともあるかと思いますがここでは考えません)

蟻本には`dp[i番目まで入れた][重さ合計j] = 最大価値` とスライド最小値を用いて $O(NW)$ で求める方法が紹介されています

重さの自明な上界 $\sum{x_i z_i}$ や $NXZ$ を用いて評価してもいいと思います

同様にして `dp[i番目まで入れた][価値合計j] = 最小重さ` をスライド最小値を用いて $O(N\sum{y_i z_i}) \subset O(N^2YZ)$ でできます

ここでは [ARC096 F - Sweet Alchemy](https://atcoder.jp/contests/arc096/tasks/arc096_d){:target="_blank"}<!--_--> の解説にて紹介されていた (若干の改善を含む) $O(N^2Y^2)$ での解法を紹介します．解説PDFになかった証明などを書きました

# 復習

|アイデア|計算量|備考|
|--|--|--|
|`dp[i番目までで]` <br> `[重さの和j] = 最大価値`|$O(NW)$|$z_i = 1$,$z_i = \infty$|
|`dp[i番目までで]` <br> `[重さの和j] = 最大価値` <br> + スライド最小値|$$O(N \min \{W, \sum{x_i z_i} \})$$ <br> $\subset O(N^2XZ)$||
|`dp[i番目までで]` <br> `[価値の和j] = 最小重さ` <br> + スライド最小値|$O(N \sum{y_i z_i})$ <br> $\subset O(N^2YZ)$|$W + 1$ を $\infty$ として用いて不可能性を保持します|
|一個上のもの <br> + **特殊なある条件** を使った探索領域の削減 |$O(N^2 Y^2)$|今回紹介するもの|


申し訳ないのですが，上の3つについては前提知識とします 

実際は3つめだけで大丈夫です．3つめを**アルゴリズムB**と呼ぶことにします

蟻本と以下の実装を参考にすれば理解できるかと思います

# 観察

ナップサック問題は重さ順や価値順，効率 ( = 価値 / 重さ) 順などにして貪欲でやってもそれら全てで最適解が得られないことがあります

勝手に引用しますが，drkenさんが <span style="color:red"> "DPの凸凹したイメージ" </span> ([マトロイドの凸構造 #2. 凸凹でも通用！DPのパワー！](http://drken1215.hatenablog.com/entry/20121212/1355280288#2){:target="_blank"}<!--_-->) というのを説明してくださっているように，一筋縄ではいきません

制約を守っている，ある解 (つまり $c$ という数列) を**実行可能解** と呼ぶことにします

ここでは効率順にソートすることを考えてみます．つまり $y_i / x_i$ の大きい順にします

基本的に効率が良いものを選ぶのが良いです．より具体的には，どちらもピッタリ埋めることができる領域が与えられたなら効率が良い方を選ぶのが必ず良いです

ここで２つの荷物 $p \lt q$ を考えます．ソートしたので $y_p / x_p \geq y_q / x_q$ を満たします

ある実行可能な解を得たとして，その解が $z_p - c_p \geq Y$ かつ $c_q \geq Y$ だとします． $z_p - c_p$ は $p$**番目のあまり** です

この場合，$p$番目を $y_q$ 個採用して $q$ 番目を $y_p$ 個捨てることで今より少ない重さの実行可能解が得られることがわかります

>  説明  
>  $y_p / x_p \geq y_q / x_q$ より $y_q x_p \leq y_p x_q$ なのでより少ない重さとなる  
>  また $y_p y_q$ 減らして同じ分だけ増えているので，結果的により少ない重さで同じ価値を得ている

以下のように最適条件Xを定義する

> 定義 : 以下の条件を最適条件Xとする  
> $p < q$ で $z_p - c_p \geq Y$ かつ $c_q \geq Y$ となるような $p, q$ が存在しない

上記の操作は有限回しか行なえません．よっていつかは最適条件Xを満たす実行可能解になります

逆に言えば，**最適条件Xをみたすものを全探索できればよい** ということになります

# 最適条件Xのイメージ

最適条件Xってなんなのでしょうか

![knapsack1]({{ "img/knapsack1.png" | absolute_url }})

上記の図のように "最後にY個以上使った" "最初にY個以上余った" の順に現れる (同時でも構わない) ような実行可能解が最適条件Xを満たす，ということがわかります

このような解を全て探索できればよいわけです

# アルゴリズム

ここで以下のようなアルゴリズムAを考えます

> アルゴリズムA  
> まず効率の順にソートする  
> 2つの袋1,2を考える．各$i$について$z_i$個を，袋1には $$\min \{Y - 1, z_i\}$$ 個，袋2にはその残りを入れる  
> 袋1について**アルゴリズムB**を用いて $O(N^2 Y^2)$ で各価値に対する最小重さを求める  
> 各価値について，それが袋1で実行可能なら，残りの重さの余裕がなくなるまで添字が小さい順に袋2から採用する．採用できなくなったらもしその後ろに採用できるものがあったとしてもやめて良い  
> その最大値が最適解となる

あまりを $r_i \coloneqq z_i - c_i$ 個，袋1に入れた数を $$s_i \coloneqq \min \{Y - 1, c_i\}$$ 個とします

アルゴリズムAは一体どんな解を与えるのでしょうか．例を考えてみます

![knapsack2]({{ "img/knapsack2.png" | absolute_url }})

値が先程の図と全て同じなので混乱してしまうかもしれないんですが，**アルゴリズムAによって得られた実行可能解**と考えてください

この例えが言いたいのは，ある$p$が存在して，$i \leq p$については $z_i - c_i \lt Y$ で $i \geq p$ については $c_i \lt Y$ である，ということです

これは **アルゴリズムAが生成する実行可能解は全て最適条件Xを満たす** と言えます

次に **最適条件Xを満たす最適な実行可能解を少なくとも1つアルゴリズムAが生成する** ことを示します．上記の議論は消して無駄なわけではなく，以下の議論をわかりやすくするためのものです

# アルゴリズムAの正当性

> 極大な最適解が存在する

これはつまり，まだ入れることができるなら入れても悪くならないからです

> アルゴリズムAは全ての最適条件を満たす極大な実行可能界を探索している

最初の図について，極大であることを仮定して，各$i$について選ばれたものがどちらの袋に属していたかを `(袋1から採用した数) + (袋2から採用した数)` という形で書き表しました

まず袋1については $O(N^2Y^2)$ の時間を書けて**アルゴリズムB** によって全て探索されています

![knapsack3]({{ "img/knapsack3.png" | absolute_url }})

これがアルゴリズムAによって探索されることは，まず**実行可能解である**という前提から，袋2の上の方から順に追加していっても容量$W$ が足りなくならないので構成可能なことがわかります

**極大である**という前提より，それより先が足されることはありません

"最後にY個以上使った" "最初にY個以上余った"と言えるようなインデックスを $i, j$ と置くことにします (最適条件を満たすので $i \leq j$)

どちらを基準にしてもいいのですが，$p = i$とすると $k \leq p$ については $c_k = (s_k - r_k) + (c_k - (s_k - r_k))$ と分解できます (ここで $k \lt p$ については袋2の中身はすべて使っている**ようにできる**に注意してください)

$k \gt p$ については $c_k \lt Y$ なわけですから，$c_k = c_k + 0$ と分解できます

$i = j$ だとしても $p~(= i = j)$については適当にすれば構築できます (図で説明した "袋2の一部を使う" に該当するため)

よって，極大な最適条件Xを満たす実行可能界は，**常にアルゴリズムAによって探索されるような分解ができる**ことが言えたので，アルゴリズムAは最適解を生成します

---

復元もできます．紹介したアルゴリズムは `knapsack3` として実装しています

計算量については二分探索などを行うことにより $O(N^2 Y^2)$ で行えます

# 実装


```cpp
// x : weight
// y : value
// z : limit
// knapsack

// O(min{NW, N \sum {xz}}) < O(N^2 XZ) time
// O(W) space
// knapsack1(<ull>, <ll>, <uint>, uint) {{"{{"}}{
#include <cassert>
#include <deque>
#include <vector>
vector< ll > knapsack1(const vector< unsigned long long >& x, const vector< ll >& y,
                       const vector< unsigned >& z, unsigned W) {
  using ull = unsigned long long;
  size_t n = x.size();
  assert(n == y.size());
  assert(n == z.size());
  if(n == 0) return vector< ll >();
  ull w_max = 0;
  for(size_t i = 0; i < n && w_max < W && (x[i] < W || z[i] == 0); i++) {
    w_max += x[i] * z[i];
    if(i == n - 1 && w_max < W) W = w_max;
  }
  vector< ll > dp(W + 1);
  for(size_t i = 0; i < n; i++)
    if(z[i]) {
      if(x[i]) {
        vector< ll > dp0(W + 1);
        swap(dp, dp0);
        for(size_t r = 0; r < x[i] && r <= W; r++) {
          // k := z[i]
          // a[i][j] := dp[i][j * x[i] + r]
          // dp[i + 1][(j + k) * x[i] + r] ( = a[i + 1][j + k] )
          //  = max{a[i][j] + k * y[i], a[i][j + 1] + (k - 1) * y[i], ...}
          //  = max{a[i][j] + -j * y[i], a[i][j + 1] + -(j + 1) * y[i], ...}
          //      + (j + k) * y[i]
          size_t u = (W - r) / x[i];
          // window sliding technique
          deque< pair< ll, size_t > > deq;
          for(size_t j = 0; j <= u; j++) {
            ll nval = dp0[j * x[i] + r] - j * y[i];
            while(deq.size() && deq.back().first < nval) deq.pop_back();
            deq.emplace_back(nval, j);
            dp[j * x[i] + r] = deq.front().first + j * y[i];
            if(deq.front().second + z[i] == j) deq.pop_front();
          }
        }
      } else {
        for(auto& el : dp) el += y[i] * z[i];
      }
    }
  return dp;
}
// }}}

// check whether \sum {yz} won't overflow
// O(N \sum {yz}) < O(N^2 YZ) time
// knapsack2(<ll>, <uint>, <uint>, ull) {{"{{"}}{
vector< ll > knapsack2(const vector< ll >& x, const vector< unsigned >& y,
                       const vector< unsigned >& z, unsigned long long W) {
  using ull = unsigned long long;
  size_t n = x.size();
  assert(n == y.size());
  assert(n == z.size());
  if(n == 0) return vector< ll >();
  ull value_max = 0;
  for(size_t i = 0; i < n; i++) value_max += y[i] * z[i];
  vector< ll > dp(value_max + 1, W + 1);
  dp[0] = 0;
  for(size_t i = 0; i < n; i++)
    if(y[i] && z[i]) {
      vector< ll > dp0(value_max + 1, W + 1);
      swap(dp, dp0);
      for(size_t r = 0; r < y[i]; r++) {
        // k := z[i]
        // a[i][j] := dp[i][j * y[i] + r]
        // dp[i + 1][(j + k) * y[i] + r] ( = a[i + 1][j + k] )
        //  = min{a[i][j] + k * x[i], a[i][j + 1] + (k - 1) * x[i], ...}
        //  = min{a[i][j] + -j * x[i], a[i][j + 1] + -(j + 1) * x[i], ...}
        //      + (j + k) * x[i]
        size_t u = (value_max - r) / y[i];
        // window sliding technique
        deque< pair< ll, size_t > > deq;
        for(size_t j = 0; j <= u; j++) {
          if((ull) dp0[j * y[i] + r] <= W) {
            ll nval = dp0[j * y[i] + r] - (ll) j * x[i];
            while(deq.size() && deq.back().first > nval) deq.pop_back();
            deq.emplace_back(nval, j);
          }
          if(deq.size()) dp[j * y[i] + r] = deq.front().first + j * x[i];
          if(deq.size() && deq.front().second + z[i] == j) deq.pop_front();
        }
      }
    }
  return dp;
}
// }}}

// check whether \sum {yz} won't overflow
// O(N^2 Y^2)
// knapsack3(<ull>, <uint>, <ull>, ull) {{"{{"}}{
#include <algorithm>
#include <numeric>
unsigned long long knapsack3(const vector< unsigned long long >& x,
                             const vector< unsigned >& y,
                             const vector< unsigned long long >& z,
                             unsigned long long W) {
  using ull = unsigned long long;
  size_t n = x.size();
  assert(n == y.size());
  assert(n == z.size());
  if(n == 0) return 0;
  size_t Y = 0;
  for(size_t i = 0; i < n; i++)
    if(Y < y[i]) Y = y[i];
  if(Y == 0) return 0;
  vector< size_t > ord(n);
  iota(begin(ord), end(ord), 0);
  sort(rbegin(ord), rend(ord),
       [&](size_t a, size_t b) { return (double) y[a] * x[b] < (double) y[b] * x[a]; });

  vector< ll > nx(n);
  vector< unsigned > ny(n);
  vector< unsigned > nz(n);
  vector< ull > zz(n);

  for(size_t i = 0; i < n; i++) {
    nx[i] = x[ord[i]];
    ny[i] = y[ord[i]];

    nz[i] = min(z[ord[i]], (ull) Y - 1);
    zz[i] = z[ord[i]] - nz[i];
  }

  auto sub = knapsack2(nx, ny, nz, W);

  vector< ull > xz_sum(n);
  vector< ull > yz_sum(n);
  for(size_t i = 0; i < n; i++) xz_sum[i] = nx[i] * zz[i];
  for(size_t i = 0; i < n; i++) yz_sum[i] = ny[i] * zz[i];
  for(size_t i = 1; i < n; i++) xz_sum[i] = min(xz_sum[i - 1] + xz_sum[i], W + 1);
  for(size_t i = 1; i < n; i++) yz_sum[i] = yz_sum[i - 1] + yz_sum[i];

  ull res = 0;
  // O(N Y^2 log N)
  for(unsigned v = 0; v < sub.size(); v++) {
    ull use = sub[v];
    if(use > W) continue;
    ull rest = W - use;
    size_t xp = upper_bound(begin(xz_sum), end(xz_sum), rest) - begin(xz_sum);
    ull val = v;
    if(xp >= 1) val += yz_sum[xp - 1], rest -= xz_sum[xp - 1];
    assert(xp == n || nx[xp] != 0); // explicit
    if(xp < n) val += rest / nx[xp] * ny[xp];
    if(res < val) res = val;
  }

  return res;
}
// }}}
```


# 検証

* [ARC096 F - Sweet Alchemy (900) - AtCoder](https://atcoder.jp/contests/arc096/submissions/4273002){:target="_blank"}<!--_-->

# 参考

* [ARC096 F - Sweet Alchemy (900) - AtCoder](https://atcoder.jp/contests/arc096/tasks/arc096_d){:target="_blank"}<!--_-->

