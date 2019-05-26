---
layout: page
title: 設計思想
top: true
permalink: architecture
---

適当たぷ

C++14以上 を前提

# 理念

* 自己完結は目指していない
* ノートとして思い出すのに役立つようにつくってある

# 守れてそうなこと

## 実装について

* `query(l, r)` は半開区間[l, r)
* `range(l, r)` は閉区間[l, r]
* `std::` 書く
* `assert` を書く 原因不明なまま落ちるのを防ぐ
* `perbuild()`, `build()` に統一
* `.fold` に統一
* `.fold`, `.act` 系は 「全域」と呼べるような範囲が自明であるなら，範囲指定を省略して全域とする

# これから変えていきたい

## 実装について

* `using ll = long long` などの前提をなくす
* `assert` を `throw` に統一すべきか検討中
* コンストラクタ(InputIter frist, InputIter last)をちゃんと用意
* exampleはuncommentされた状態で (formatとか考慮して)
* ASCII-onlyに (HackerRank対策)
* `dat` vs `data`
* ll 前提をなしにするか...?
  * 悩んでいる．ll = int128, bigintみたいに一斉変更ができる
  * 解決策としては，テンプレートで整数型を受け取るように変更する
* `#inlcude<bits/stdc++.h>` の前提を無しに
  * コンパイル時間考慮
* "~ Library" って名前のライブラリは "Library" の部分を取り除きたい

## 記事について

* 文体がバラバラなのはもう諦めている
* 最後のピリオドはなしにする

