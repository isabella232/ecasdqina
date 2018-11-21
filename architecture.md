---
layout: page
title: 設計思想
top: true
permalink: architecture
---

うーんあんま決めてない（え

これから決めるうく

# 守れてそうなこと

* query(l, r)は半開区間[l, r)
* range(l, r)は閉区間[l, r]
* `std::` 書かない
  * 急いで変更したいときに，マクロで黒魔術できたりと便利なのでそうしている
* ミニマルで再実装しやすい実装
* perbuild(), build() に統一

# これから変えていきたい

* コンストラクタ(InputIter frist, InputIter last)をちゃんと用意
* exampleはuncommentされた状態で（formatとか考慮して)
* ASCII-onlyに（HackerRank対策）
* `dat` vs `data`
* ll 前提をなしにするか...?
  * 悩んでいる．ll = int128, bigintみたいに一斉変更ができる
* `#inlcude<bits/stdc++.h>` の前提を無しに
  * コンパイル時間考慮
* "~ Library" って名前のライブラリは "Library" の部分を取り除きたい

## 記事について

* 文体がバラバラなのはもう諦めている
* 最後のピリオドはなしにする

