---
layout: lib
title: Rollback可能なデータ構造とクエリ平方分割（Moの上位互換）
permalink: algorithm/MoEx

---


参考: [Mo's algorithm の上位互換の話 - あなたは嘘つきですかと聞かれたら「YES」と答えるブログ](http://snuke.hatenablog.com/entry/2016/07/01/000000){:target="_blank"}<!--_-->

基本的なアイデアはMoと似ていて，  
ある瞬間を記録するsnapshotと，  
記録した状態に戻すrollbackを定義できればいいです．

追加操作は残りますが，削除操作は必要なくなります．

追加操作の計算量は均しが $O(1)$ とかだとしても，  
ある瞬間に $O(N)$ とかかかる場合，その瞬間を何回も繰り返されると死にます．

Rollback可能なことを永続というかちょっとわからないんですが，wikipedia見る限りではちょっと違うなと思いました．(過去の任意の時間のものを参照できるというのが必要そう)

# 実装

区間はすべて $[l, r)$ (半開区間) です
<!--]-->

MoのCFでの記事の[コメント](https://codeforces.com/blog/entry/7383?#comment-161520){:target="_blank"}<!--_-->を参考に書いてみました．


```cpp
// MoEx(N, JUST Q, double k)
// favored : k = 2
// 1: insert
// 2: build
/// --- Mo with Persistent Data Structure Library {{"{{"}}{ ///

struct MoEx {
  const int width;
  int q = 0;
  vector< int > le, ri, order;
  MoEx(int n, int q, double k = 1)
      : width(int(k* n / sqrt(q) + 1.0)), le(q), ri(q), order(q) {}
  inline void insert(int l, int r) {
    le[q] = l;
    ri[q] = r;
    order[q] = q;
    q++;
  }
  inline void build() {
    for(int i = 0; i < q; i++)
      if(ri[i] - le[i] < width) {
        init();
        for(int j = le[i]; j < ri[i]; j++) add(j);
        next(i);
      }
    sort(begin(order), begin(order) + q, [&](int a, int b) {
      const int ab = le[a] / width, bb = le[b] / width;
      return ab != bb ? ab < bb : ri[a] < ri[b];
    });
    int last = -1;
    int nr;
    for(int i = 0; i < q; i++) {
      int id = order[i];
      if(ri[id] - le[id] < width) continue;
      int b = le[id] / width;
      if(last != b) init(), nr = (b + 1) * width;
      last = b;
      while(nr < ri[id]) add(nr++);
      snapshot();
      for(int j = (b + 1) * width - 1; j >= le[id]; j--) add(j);
      next(id);
      rollback();
    }
  }
  inline void next(int id);
  inline void init();
  inline void snapshot();
  inline void rollback();
  inline void add(int i);
};

/// }}}--- ///

inline void MoEx::next(int id) {}
inline void MoEx::init() {}
inline void MoEx::snapshot() {}
inline void MoEx::rollback() {}
inline void MoEx::add(int i) {}
```


## UnionFindを使う例

検証にもあります


```cpp
const int N = 1e5;

int par[N * 2];
int col = 1;
int used[N * 2];
int ans[N];
using P = pair< int, int >;
vector< P > history;

void reset(int i) {
  if(used[i] != col) used[i] = col, par[i] = -1;
}
int find(int a) {
  reset(a);
  return par[a] < 0 ? a : find(par[a]);
}
bool same(int a, int b) { return find(a) == find(b); }
void unite(int a, int b) {
  a = find(a), b = find(b);
  if(a == b) return;
  if(par[a] < par[b]) swap(a, b);
  history.emplace_back(b, par[b]);
  history.emplace_back(a, par[a]);
  par[b] += par[a];
  par[a] = b;
}

inline void MoEx::next(int id) { ans[id]; /* */ }
inline void MoEx::init() {
  //
  history.clear();
  col++;
}
inline void MoEx::snapshot() {
  oldNow = now;
  history.clear();
}
inline void MoEx::rollback() {
  now = oldNow;
  while(history.size()) {
    int i, x;
    tie(i, x) = history.back();
    par[i] = x;
    history.pop_back();
  }
}
inline void MoEx::add(int i) {
  unite(a[i] + N, b[i]);
  unite(a[i], b[i] + N);
  //
}
```


# 検証

* [A - Nasta Rabbara - CF](https://codeforces.com/gym/100513/submission/42741944){:target="_blank"}<!--_-->
  * rollback可能なUFを使います
  * みんな5000ms前後かなーという印象ですがなんか200msとかいうヤバも見えます

# 練習

* [A - Nasta Rabbara - CF](https://codeforces.com/gym/100513/problem/A){:target="_blank"}<!--_-->

