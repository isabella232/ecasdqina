---
layout: lib
title: XorShift
permalink: misc/XorShift

---


mt19937などより速い ([僕の記事](http://tomorinao.blogspot.com/2018/08/xorshift128.html){:target="_blank"}<!--_-->)

[Treap]({{ "data-structure/BBST/Treap" | absolute_url }}) などでつかったけど  
マラソンで使ったりすることが多いのかな

## XorShift128

要件 [UniformRandomBitGenerator](https://ja.cppreference.com/w/cpp/named_req/UniformRandomBitGenerator){:target="_blank"}<!--_--> を満たします (例えば，[shuffle](https://cpprefjp.github.io/reference/algorithm/shuffle.html){:target="_blank"}<!--_-->で使えるなどの嬉しさがあります)


```cpp
/// --- XorShift128 {{"{{"}}{ ///

#include <cstdint>
struct XorShift128 {
  using result_type = uint_fast32_t;
  static constexpr result_type min() { return 0; }
  static constexpr result_type max() { return 0xFFFFFFFF; }
  result_type x = 123456789, y = 362436069, z = 521288629, w = 88675123;
  XorShift128(result_type seed = 0) { z ^= seed; }
  result_type operator()() {
    result_type t = x ^ (x << 11);
    x = y, y = z, z = w;
    return w = (w ^ (w >> 19)) ^ (t ^ (t >> 8));
  }
};

/// }}}--- ///
```


# 参考

* [Xorshift - Wikipedia](https://ja.wikipedia.org/wiki/Xorshift){:target="_blank"}<!--_-->
* [http://hexadrive.sblo.jp/article/63660775.html](http://hexadrive.sblo.jp/article/63660775.html){:target="_blank"}<!--_-->

