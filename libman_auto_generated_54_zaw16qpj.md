---
layout: lib
title: 文字列ユーティリティ
id: string-utility
permalink: string/string-utility

---


文字列をデリミタで区切るやつ


```cpp
vector< string > split(const string& s, char delim) {
  vector< string > elms;
  size_t offset = 0;
  while(true) {
    size_t next = s.find_first_of(delim, offset);
    if(next == string::npos) {
      elms.emplace_back(s.substr(offset));
      return elms;
    }
    elms.emplace_back(s.substr(offset, next - offset));
    offset = next + 1;
  }
}
```


