---
layout: lib
title: 文字列検索 (Aho-Corasick法)
permalink: string/AhoCorasick

---


複数パターンを事前に用意するタイプの検索


構築は 辞書の大きさ，各文字列の長さに対して線形

検索は 検索する文字列に対しても，みつかったマッチングの数に対しても線形

解説は [ここ](http://d.hatena.ne.jp/naoya/touch/20090405/aho_corasick){:target="_blank"} がとてもわかりやすいです．<!--_-->

問題によって要求されるものがかわってくるので，それに合わせていきます


```cpp
// add(string)
// === build() ===
// match(string, f)           : [left, right], f(int left, int right, int key)
// next(node, char, f): Trie* : f(int len, int key)
// - right-inc -> left-dec order
// === --- ===
// if ML is tight, comment out "delete"
/// --- AhoCorasick Library {{"{{"}}{ ///

#include <functional>
#include <unordered_map>

struct AhoCorasick {
  struct Trie {
    unordered_map< char, Trie * > child;
    int id;
    Trie *failure = nullptr;
    Trie *next = nullptr; // next failure that has the word
    int word = -1;
    Trie *add(char c) {
      if(child.count(c)) {
        return child[c];
      } else {
        return child[c] = new Trie;
      }
    }
    Trie *go(char c) {
      if(child.count(c)) {
        return child[c];
      } else {
        if(failure == nullptr) { // only top can come here
          return this;
        } else {
          return failure->go(c);
        }
      }
    }
    ~Trie() {
      for(auto &x : child) {
        delete x.second;
      }
    }
  };
  Trie *top = new Trie;
  ~AhoCorasick() {
    // delete top;
  }
  vector< string > dict;
  void add(string word) {
    Trie *now = top;
    for(size_t i = 0; i < word.size(); i++) {
      now = now->add(word[i]);
    }
    now->word = dict.size();
    dict.emplace_back(word);
  }
  void build() {
    queue< Trie * > q;
    q.emplace(top);
    while(q.size()) {
      Trie *now = q.front();
      q.pop();
      for(pair< char, Trie * > ch : now->child) {
        q.emplace(ch.second);
        Trie *failure = ch.second->failure =
            now == top ? top : now->failure->go(ch.first);
        ch.second->next = failure->word >= 0 ? failure : failure->next;
      }
    }
  }
  Trie *next(Trie *now, char c,
             const function< void(int, int) > &f = [](int, int) {}) {
    now = now->go(c);
    Trie *tmp = now;
    while(tmp != nullptr && tmp != top) {
      int word = tmp->word;
      if(word >= 0) {
        f(dict[word].size(), word);
      }
      tmp = tmp->next;
    }
    return now;
  }
  void match(string s,
             const function< void(int, int, int) > &f = [](int, int, int) {}) {
    Trie *now = top;
    for(size_t i = 0; i < s.size(); i++) {
      now = now->go(s[i]);
      Trie *tmp = now;
      while(tmp != nullptr && tmp != top) {
        int word = tmp->word;
        if(word >= 0) {
          f(i - dict[word].size() + 1, i, word);
        }
        tmp = tmp->next;
      }
    }
  }
};

/// }}}--- ///

using Trie = AhoCorasick::Trie;
```


MLが厳しい場合は自分で `delete` の行のコメントを取る

# 検証

* [No.430 文字列検索 - yukicoder](https://yukicoder.me/submissions/281765){:target="_blank"}<!--_-->
  * 貼 (って少しいじ) るだけ
  * ライブラリを修正し，いじる必要がないようにしました．いじってもいいです
* [G： 検閲により置換 (Censored String) - AOJ](https://onlinejudge.u-aizu.ac.jp/status/users/luma/submissions/1/2873/judge/3116720/C++14){:target="_blank"}<!--_-->
  * 問題に合わせて少し変える必要があります．
* [H - Separate String - AtCoder](https://beta.atcoder.jp/contests/jag2017autumn/submissions/3107157){:target="_blank"}<!--_-->
  * この問題でいろいろもとの実装を見直しました

# 練習問題

* [No.430 文字列検索 - yukicoder](https://yukicoder.me/problems/no/430){:target="_blank"}<!--_-->
* [G： 検閲により置換 (Censored String) - AOJ](https://onlinejudge.u-aizu.ac.jp/problems/2873){:target="_blank"}<!--_-->
* [H - Separate String - AtCoder](https://beta.atcoder.jp/contests/jag2017autumn/tasks/jag2017autumn_h){:target="_blank"}

# 参考

* [Aho Corasick 法 - naoyaのはてなダイアリー](http://d.hatena.ne.jp/naoya/touch/20090405/aho_corasick){:target="_blank"}<!--_-->


