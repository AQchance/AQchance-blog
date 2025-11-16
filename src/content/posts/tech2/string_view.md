---
title: 如何高效返回字符串？std::string_view详解
published: 2025-11-16
description: '本文讲解std::string_view的使用方法及其优势'
image: ''
tags: ['C++', '编程语言']
category: '技术'
draft: false 
lang: ''
---

# 从远古时代谈起

在C语言的早期，字符串通常以字符数组的形式存在，并以空字符`'\0'`结尾。这种表示方式虽然简单，但在处理字符串时存在一些问题，例如需要手动管理内存、容易出现缓冲区溢出等。
在C语言中，如果需要返回一个字符串，通常会使用字符指针`char *`来指向一个字符串，比如下面的这个函数：

```c
const char *get_str() {
  return "Hello, World!";
}
```

这个函数返回一个指向字符串字面值的指针。然而，这种通过`char *`来返回的方式有几个缺点：
1. **内存管理**：如果返回的是动态分配的字符串，调用者需要负责释放内存，容易导致内存泄漏。
2. **长度信息缺失**：返回的指针没有包含字符串的长度信息，调用者需要自己计算长度。

# 半步进入现代

随着C++的发展，引入了`std::string`类来更好地管理字符串。`std::string`封装了字符串的长度和内存管理，使得字符串操作更加安全和方便。然而，`std::string`在某些情况下也存在性能问题，特别是在需要频繁传递和返回字符串时，因为它涉及到内存分配和复制。

一些C++代码中仍然在传递参数以及返回值的时候使用`const char *`，例如下面这样的写法：

```cpp
#include <iostream>
#include <string>

void print_message(const char *str) { std::cout << str << std::endl; }

int main() {
  std::string message = "Hello, World!";
  print_message(message.c_str());
  return 0;
}
```

这样的写一般而言是出于对性能的考虑，避免了`std::string`对象的构造和析构开销。然而，这种方式也有缺点，比如需要调用`c_str()`方法来获取C风格的字符串指针，增加了代码的复杂性。而且，你往往会因为对于该指针所指内存的改变而导致代码出现一些难以察觉的bug。

一种更好的方式是使用引用的方式传递`std::string`，如下所示：

```cpp
#include <iostream>
#include <string>

void print_message(const std::string &str) { std::cout << str << std::endl; }

int main() {
  std::string message = "Hello, World!";
  print_message(message);
  return 0;
}
```

你可能会觉得，看上去`std::string`已经足够替代`char *`了吧？但事实并非如此。`std::string`在某些场景下仍然存在性能瓶颈，特别是在需要返回字符串的时候，因为返回`std::string`对象通常涉及到内存分配和复制操作。

考虑下面的一个函数，它返回一个`std::string`对象：

```cpp
#include <iostream>
#include <string>

std::string get_str() {
  return "Hello, World!";
}
const char *get_cstr() {
  return "Hello, World!";
}

int main() {
  std::cout << get_str() << std::endl;
  return 0;
}
```

在这个例子中，`get_str`函数返回一个`std::string`对象。你可能觉得这是一个比返回`const char *`更好的选择，因为它自动管理内存并包含长度信息。然而事实是，它的性能是不如返回`const char *`的版本的。
这是因为返回`std::string`对象通常涉及到内存分配和复制操作，而返回`const char *`只是返回一个指针，没有额外的开销。这个看似微小的差异在高性能应用中可能会积累成显著的性能损失。
这也是为什么在一些性能敏感的代码中，开发者仍然倾向于使用`const char *`来传递和返回字符串。

# 现代C++的解决方案：`std::string_view`

为了在性能和安全性之间取得平衡，C++17引入了`std::string_view`，它是一种轻量级的字符串视图类型，可以高效地表示字符串而无需复制数据。
`std::string_view`本质上是一个指向字符串数据的指针和一个长度值的组合。
它允许你在不拥有字符串数据的情况下操作字符串，从而避免了不必要的内存分配和复制开销。
下面是一个使用`std::string_view`的例子：

```cpp
#include "string_view"
#include <iostream>
#include <string>

void print_message(std::string_view message) {
  std::cout << message << std::endl;
}

std::string_view get_str() { return "Temporary String"; }

int main() {
  std::string message = "Hello, World!";
  std::string_view sv = message;
  print_message(sv);

  std::string_view temp_sv = get_str();
  print_message(temp_sv);

  return 0;
}
```

在这个例子中，`print_message`函数接受一个`std::string_view`参数，可以高效地处理字符串而无需复制数据。`get_str`函数返回一个`std::string_view`，指向一个临时字符串字面值。
这种方式结合了`const char *`的高效性和`std::string`的安全性，避免了内存管理的问题，同时提供了长度信息。
需要注意的是，`std::string_view`并不拥有它所指向的字符串数据，因此在使用时必须确保字符串数据在`std::string_view`的生命周期内是有效的。否则，可能会导致悬空引用的问题。

例如下面的这个例子就是错误的用法：

```cpp
#include "string_view"
#include <iostream>
#include <string>

int main() {

  std::string_view sv;
  {
    std::string str = "Hello, World!";
    sv = str; // sv points to str's data
    std::cout << sv << std::endl;
  } // str goes out of scope here

  std::cout << sv
            << std::endl; // Undefined behavior: sv points to invalid memory
}
```

在这个例子中，`sv`在`str`的作用域之外被使用，导致未定义行为，因为`sv`指向的内存已经被释放。在g++ 15.2版本下运行结果如下所示：

![string_view悬空引用](1.png)

在g++ 15.2版本的运行结果似乎并没有出现问题，但是这并不能保证你的代码在所有情况下都能正常工作。实际上，这种用法是错误的，会导致不可预测的行为，因此在使用`std::string_view`时务必注意其生命周期管理。

