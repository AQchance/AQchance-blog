---
title: Effective C++读书笔记(1)
published: 2025-11-10
description: '本小节主要介绍C++的四种类型转换'
image: ''
tags: ['C++', '编程语言']
category: '技术'
draft: false 
lang: ''
---

> ## C++的四种类型转换

### 静态转换(static_cast)

静态转换用于在编译时进行类型转换，适用于基本类型之间的转换，如int到float的转换。它不会执行运行时检查，因此在使用时需要确保转换是安全的。

```cpp
int i = 10;
float f = static_cast<float>(i);
```

### 动态转换(dynamic_cast)

动态转换用于在类层次结构中进行类型转换，主要用于将基类指针转换为派生类指针。它在运行时进行类型检查，如果转换不安全，则返回nullptr。

```cpp
Base* basePtr = new Derived();
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
if (derivedPtr) {
    // 转换成功
} else {
    // 转换失败
}

```
### 常量转换(const_cast)

常量转换用于去除对象的const或volatile属性。它主要用于修改通过const指针或引用传递的对象。

```cpp
const int* constPtr = &i;
int* nonConstPtr = const_cast<int*>(constPtr);
```

### 重新解释转换(reinterpret_cast)

重新解释转换用于在不相关类型之间进行低级别的转换。它通常用于指针类型之间的转换，但需要谨慎使用，因为它可能导致未定义行为。

```cpp
long address = 0x12345678;
int* intPtr = reinterpret_cast<int*>(address);
```

通过理解和正确使用这四种类型转换，C++程序员可以更有效地管理类型转换，提高代码的安全性和可读性。

