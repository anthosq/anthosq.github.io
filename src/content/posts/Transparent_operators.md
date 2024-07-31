---
title: Transparent_operators
published: 2024-07-31
description: ''
image: ''
tags: 
- C++
category: C++
draft: false 
---

```cpp
class Hasher {
public:
    using is_transparent = void;
    size_t operator()(std::string_view sv) const {
        return std::hash<std::string_view>{}(sv);
    }
};
// 统一的哈希方法，避免const string &, const char *等类型的实现问题
```

