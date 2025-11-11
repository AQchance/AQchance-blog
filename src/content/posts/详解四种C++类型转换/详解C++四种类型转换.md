---
title: 详解C++四种类型转换
published: 2025-11-11
description: '本文主要介绍C++的四种类型转换'
image: ''
tags: ['C++', '编程语言']
category: '技术'
draft: false 
lang: ''
---

> ## C++的四种类型转换

> ### 静态转换(`static_cast`)

对于静态类型转换，它主要用于以下几种情况：

1. 基本数据类型之间的转换，例如`int`到`float`，`char`到`int`等，因此可以替换C风格的强制类型转换。
2. 指针或引用类型之间的转换，例如将派生类指针转换为基类指针。
3. 枚举类型与整数类型之间的转换。

我们一一来看几个例子：

- 基本数据类型之间的转换：（这里应该考虑到不同类型之间的转换可能会导致精度的损失，应该仔细考虑是否需要转换）
```cpp
int int_number = 42;
float float_number = static_cast<float>(int_number);
double double_number = static_cast<double>(int_number);
```

- 指针或引用类型之间的转换：
```cpp
class Base {};
class Derived : public Base {};
Base* basePtr = new Derived();
Derived* derivedPtr = static_cast<Derived*>(basePtr);
```

需要注意的是，`static_cast`不会进行运行时类型检查，因此在进行向下转换（从基类指针转换为派生类指针）时，必须确保转换是安全的，否则可能会导致未定义行为。


- 枚举类型与整数类型之间的转换：
```cpp
enum Color { RED, GREEN, BLUE };
Color color = RED;
int colorInt = static_cast<int>(color);
Color newColor = static_cast<Color>(colorInt);
```

> ### 动态转换(`dynamic_cast`)

动态转换用于在类层次结构中进行类型转换，主要用于将基类指针转换为派生类指针。它在运行时进行类型检查，如果转换不安全，则返回`nullptr`。
来看下面的例子：

```cpp
#include <iostream>

class Base {
public:
  Base() = default;
  virtual void show() {
    std::cout << "Base class show function called." << std::endl;
  }
  virtual void read(){
    std::cout << "Base class read function called." << std::endl;
  }
};

class Derived : public Base {
public:
  Derived() = default;
  void show() override {
    std::cout << "Derived class show function called." << std::endl;
  }
  void function() {
    std::cout << "Derived class specific function called." << std::endl;
  }
};

int main() {
  Base *base_ptr = new Derived();

  base_ptr->show(); // 这里会调用Derived的show函数，因为它是虚函数

  // 这里为了调用派生类的特有函数，我们需要进行动态转换
  Derived *derived_ptr = dynamic_cast<Derived *>(base_ptr);
  if (derived_ptr) {
    derived_ptr->function(); // Now we can call Derived's specific function
  } else {
    std::cout << "Downcasting failed." << std::endl;
  }

  delete base_ptr; // Clean up
  return 0;
}
```
这段代码的运行结果如下所示：
![dynamic_cast原理讲解](3.png)

- `dynamic_cast`在这里确保了只有当`base_ptr`实际上指向一个`Derived`对象时，转换才会成功。如果`base_ptr`指向一个`Base`对象，那么`dynamic_cast`将返回`nullptr`，从而避免了不安全的类型转换。
也就是说，`dynamic_cast`主要用于类层次结构中的安全类型转换，特别是在涉及多态性的情况下。但是由于`dynamic_cast`在运行时进行类型检查，因此它的性能开销相对较大，在能够明确类型的情况下，建议使用`static_cast`。
下面我们来分析一下`dynamic_cast`的性能开销比较大的原因。

- 当一个类中没有虚函数的情况下，编译器不会为该类生成`vptr`指针和虚函数表(`vtable`)。这意味着在运行时，编译器无法获取对象的实际类型信息，从而无法进行动态类型检查。因此`dynamic_cast`在这种情况下无法工作。
而当一个函数中含有虚函数的时候，编译器会为该类生成一个隐藏的指针，通常称为`vptr`（虚指针），它指向该类的虚函数表(`vtable`)。这个虚函数表包含了类的所有虚函数的地址，以及类型信息。
以我们上面的代码为例，派生类的vtable结构大致如下所示：

> |      **vtable** |
> |:--------:|
> | type_info for Derived |
> | &Derived::show() |
> | &Base::read() |

- 首先，这个表只会记录虚函数的信息，而不会记录非虚函数的信息。由于我们重写了`show()`函数而没有重写`read()`函数，因此`vtable`中`show()`函数的条目指向了派生类的实现，而`read()`函数的条目仍然指向基类的实现。
当我们试图访问调用一个虚函数时，编译器会通过`vptr`访问`vtable`，然后根据函数的偏移量找到对应的函数地址并进行调用。

- 其次，`vtable`的第一个条目通常是一个指向类型信息(`type_info`)的指针，这个信息用于在运行时识别对象的实际类型。
import std;


另外，需要说明`dynamic_cast`只有在开启了运行时类型信息(RTTI)的情况下才能工作。RTTI允许程序在运行时获取对象的类型信息，这对于动态类型检查是必不可少的。
对于g++编译器，可以通过编译选项`-frtti`来启用RTTI（默认情况下是启用的）。如果禁用了RTTI，`dynamic_cast`将无法进行类型检查，可能会导致编译错误或运行时错误。

`type_info`的结构大致如下所示：
> |      **type_info for Derived** |
> |:-------------------:|
> | "Derived" (class name) |
当我们使用`dynamic_cast`进行类型转换时，编译器会通过`vptr`访问`vtable`，然后检查第一个条目中的`type_info`，以确定对象的实际类型。如果类型匹配，转换成功；否则，返回`nullptr`。

>**另外**，需要说明`dynamic_cast`只有在开启了运行时类型信息(RTTI)的情况下才能工作。RTTI允许程序在运行时获取对象的类型信息，这对于动态类型检查是必不可少的。
>对于g++编译器，可以通过编译选项`-frtti`来启用RTTI（默认情况下是启用的）。如果禁用了RTTI，`dynamic_cast`将无法进行类型检查，可能会导致编译错误或运行时错误。禁用RTTI可以使用`-fno-rtti`选项。
>对于上边的示例代码，禁用之后编译如下所示：

![禁用RTTI编译错误](4.png)

> ### 常量转换(const_cast)

常量转换用于去除对象的`const`或`volatile`属性。它主要用于修改通过`const`指针或引用传递的对象。来看下面的这个例子

```cpp
int main() {
  const int N = 10;
  const int *const_ptr = &N;
  std::cout << "常量指针所指向的值：" << *const_ptr << std::endl;
  int *ptr = const_cast<int *>(const_ptr);
  *ptr += 10; // 未定义行为
  std::cout << "非常量指针所指向的值：" << *ptr << std::endl;

  const int &const_ref = N;
  std::cout << "常量引用的值" << const_ref << std::endl;
  int &ref = const_cast<int &>(const_ref);
  ref += 10; // 未定义行为
  std::cout << "非常量引用的值" << ref << std::endl;
}
```
这段代码在g++ 15.2 版本的运行结果如下所示

![const_cast原理讲解](1.png)

我们试图修改一个原本是常量的对象，尽管看上去没有什么问题，但是实际上这种做法是未定义行为，这段代码的运行结果取决于编译器的实现。也就是说，`const_cast`只能用于去除`const`或`volatile`属性，但不能用于修改原本是常量的对象。
或许你会有疑问：那这样的话`const_cast`还有什么用呢？实际上，`const_cast`在某些情况下是有用的，比如当你需要调用一个接受非常量参数的函数，而你手头只有一个常量对象时，就可以使用`const_cast`来去除`const`属性。
如下面的代码所示

```cpp
struct MyData {
  int value;
  double moreData;
};

int func(MyData *data) { return data->value; }

int main() {
  MyData d{42, 3.14};
  const MyData *const_ptr = &d;
  MyData *ptr = const_cast<MyData *>(const_ptr);
  int result = func(ptr);
  std::cout << "Result: " << result << std::endl;
}
```


> ### 重新解释转换(reinterpret_cast)

重新解释转换用于在不相关类型之间进行低级别的转换。它通常用于指针类型之间的转换，但需要谨慎使用，因为它可能导致未定义行为。
下面的例子展示了如何逐字节访问一个`int`整数的内存表示：

```cpp
#include <cstdint>
#include <iostream>

int main() {
  int32_t number = 42;
  int *int_ptr = &number;
  char *char_ptr = reinterpret_cast<char *>(int_ptr);
  // 在这样转换之后，可以逐字节访问整数的内存表示
  for (size_t i = 0; i < sizeof(number); ++i) {
    std::cout << "Byte " << i << ": " << static_cast<int>(char_ptr[i])
              << std::endl;
  }
}
```
下面是该代码的运行结果：

![reinterpret_cast原理讲解](2.png)

需要注意的是，`reinterpret_cast`的使用需要非常谨慎，因为它可能导致未定义行为，尤其是在涉及不同类型的指针转换时。因此，只有在确实需要进行低级别内存操作时，才应使用`reinterpret_cast`。

