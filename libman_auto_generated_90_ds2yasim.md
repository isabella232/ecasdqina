---
layout: lib
title: XorShift
permalink: misc/XorShift

---


mt19937などより速い（[僕の記事](http://tomorinao.blogspot.com/2018/08/xorshift128.html){:target="_blank"}）

[Treap]({{ "data-structure/BBST/Treap" | absolute_url }}) などでつかったけど．  
マラソンで使ったりすることが多いのかな．

TreapはXorShift32のほうがよさそう…どうだろう，あとで変えるかも．

## XorShift128


```cpp
/// --- XorShift128 {{"{{"}}{ ///
struct XorShift128 {
  using u32 = uint32_t;
  u32 x = 123456789, y = 362436069, z = 521288629, w = 88675123;
  XorShift128(u32 seed = 0) { z ^= seed; }
  u32 operator()() {
    u32 t = x ^ (x << 11);
    x = y, y = z, z = w;
    return w = (w ^ (w >> 19)) ^ (t ^ (t >> 8));
  }
};
/// }}}--- ///
```


# 参考

* [Xorshift \| wikipedia](https://ja.wikipedia.org/wiki/Xorshift){:target="_blank"}
* [http://hexadrive.sblo.jp/article/63660775.html](http://hexadrive.sblo.jp/article/63660775.html){:target="_blank"}

