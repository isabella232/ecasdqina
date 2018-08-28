---
layout: page
title: 設計思想
top: true
permalink: architecture
---

うーんあんま決めてない（え．

これから決めるうく．

# 守れてそうなこと

* query(l, r)は半開区間[l, r)
* range(l, r)は閉区間[l, r]
* 無駄にstd::書かない

# これから変えていきたい

* build()に統一
* コンストラクタ(InputIter frist, InputIter last)をちゃんと用意
* 自動でビルドするやつは手動に...？（要検討）
  * コンストラクタ(int size)のものだけ手動とか
