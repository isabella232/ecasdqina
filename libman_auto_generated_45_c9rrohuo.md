---
layout: lib
permalink: misc/XorShift
title: XorShift

---


reference : http://hexadrive.sblo.jp/article/63660775.html
reference : https://ja.wikipedia.org/wiki/Xorshift

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

