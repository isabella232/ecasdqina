---
layout: lib
title: 全方位木DP (るま式)
permalink: graph/DP-all-subtree

---


基本アイデアの説明は端折ります．（そのうち書くかもね）

従来の全方位木DPを簡潔に，書きやすくした全方位木DPをつくり，  
なんとなく**るま式**と名付けましたが，解説記事を書いてる途中でめんどくさくなったため，  
解説の無いコードがただ残るのみです

ところで全方位木DPという名前，嫌いではないのですが，全部分木DP，とかのほうがしっくりします．（英語圏では何ていうでしょう）

# るま式全方木DP特徴

* DFS一個（一回とは言っていない）
  * 大抵は2個以上のDFSを使うと思いますが，一個で足ります
* ずっしりした問題にも使える
* DPテーブル最低限（最低1個）
* 場合分けがいらない（N = 1, 2の場合分けがいらない）
* 実装時の思考がスッキリ
* 森にも対応でき

# 基本

$K$ は $dp$ のステート数

$O(KN \log N)$


```cpp
const int N = 1e5;
vector< vector< int > > g(N);
using Value = int;
map< int, Value > dp[N];
Value dfs(int i, int p, int f) {
  if(dp[i].count(p)) return dp[i][p];
  int deg = g[i].size() - (p != -1);
  Value res = 0;
  if(f || p == -1) {
    // O(deg)
    // go only child
    for(int j : g[i])
      if(j != p) {
        res += dfs(j, i, f);
      }
  } else {
    // O(1)
    dfs(i, -1, f), dfs(p, i, f);
  }
  return dp[i][p] = res;
}

int n;
int main() {
  std::ios::sync_with_stdio(false), std::cin.tie(0);
  cin >> n;
  for(int i = 0; i < n - 1; i++) {
    int a, b;
    cin >> a >> b;
    a--;
    b--;
    g[a].emplace_back(b);
    g[b].emplace_back(a);
  }
  dfs(0, -1, 1);
  for(int i = 0; i < n; i++) dfs(i, -1, 0);
  return 0;
}
```


# 高速化

準備中

`map` を `vector` に変えるだけです

$O(KN)$ になります

# 検証 (使用例)

* [D - Driving on a Tree (800) - AC](https://beta.atcoder.jp/contests/s8pc-4/submissions/3232753){:target="_blank"}<!--_-->
  * 簡単です．全方位木DPの基本だと思います
* [F - Monochrome Cat (800) - AC](https://beta.atcoder.jp/contests/arc097/submissions/3233286){:target="_blank"}<!--_-->
  * 僕が思う，「ずっしりした問題」です
  * 使わない部分木があったり，計算途中を引き継いだりといったことが，  
るま式ではスッキリ書けます
  * 全方位木DP使わなくても解けるためなんか短いコードが結構ありますね
* [F - Distance Sums (900) - AC](https://beta.atcoder.jp/contests/arc103/submissions/3305207){:target="_blank"}<!--_-->

# 練習問題

* [D - Driving on a Tree (800) - AC](https://beta.atcoder.jp/contests/s8pc-4/tasks/s8pc_4_d){:target="_blank"}<!--_-->
* [F - Monochrome Cat (800) - AC](https://beta.atcoder.jp/contests/arc097/tasks/arc097_d){:target="_blank"}<!--_-->
* [F - Distance Sums (900) - AC](https://beta.atcoder.jp/contests/arc103/tasks/arc103_d){:target="_blank"}<!--_-->

