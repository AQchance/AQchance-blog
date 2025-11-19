---
title: 缺省参数的虚函数可能导致的问题
published: 2025-11-19
description: '本文讲解虚函数中使用缺省参数可能引发的问题'
image: ''
tags: ['C++', '编程语言']
category: '技术'
draft: false
lang: ''
---

> # 如果定义了一个缺省参数的虚函数，那么在子类中不要重写它

这个建议听起来有些奇怪，但确实是一个重要的C++编程实践知识。让我们通过一个例子来说明为什么会出现问题。

```cpp
#include <iostream>

enum class Color { RED, GREEN, BLUE };

class Shape {
public:
  virtual ~Shape() = default;
  virtual void draw(Color color = Color::RED) const {
    std::cout << "Drawing shape with color: " << static_cast<int>(color)
              << std::endl;
  }
};

class Circle : public Shape {
public:
  void draw(Color color = Color::GREEN) const override {
    std::cout << "Drawing circle with color: " << static_cast<int>(color)
              << std::endl;
  }
};

int main() {
  Shape *shape = new Shape();
  Shape *circle = new Circle();

  shape->draw();  // Should use Shape's default color (RED)
  circle->draw(); // Should use Circle's default color (GREEN)

  delete shape;
  delete circle;

  return 0;
}
```

在上面的代码中，我们定义了一个基类`Shape`，它有一个虚函数`draw`，该函数有一个缺省参数`color`，默认值为`Color::RED`。
然后我们定义了一个派生类`Circle`，它重写了`draw`函数，并为缺省参数提供了一个不同的默认值`Color::GREEN`。
在`main`函数中，我们创建了一个`Shape`对象和一个`Circle`对象，并分别调用了它们的`draw`方法。预期的行为是：
- 对于`shape->draw()`调用，应该使用`Shape`类的默认颜色`RED`。
- 对于`circle->draw()`调用，应该使用`Circle`类的默认颜色`GREEN`。

然而，实际输出结果如下所示：

![image](1.png)

这表明两个调用都使用了`Color::RED`，而不是预期的`Color::GREEN`。

让我们再尝试一个新的例子：
```cpp
#include <iostream>

enum class Color { RED, GREEN, BLUE };

class Shape {
public:
  virtual ~Shape() = default;
  virtual void draw(Color color = Color::RED) const {
    std::cout << "Drawing shape with color: " << static_cast<int>(color)
              << std::endl;
  }
};
class Square : public Shape {
public:
  void draw(Color color) const override {
    std::cout << "Drawing square with color: " << static_cast<int>(color)
              << std::endl;
  }
};

int main() {
  Shape *shape = new Shape();
  Shape *square = new Square();

  shape->draw(); // Should use Shape's default color (RED)
  square->draw();
  square->draw(Color::BLUE); // Must provide color explicitly for Square

  delete shape;
  delete square;

  return 0;
}
```

这次的输出结果是什么呢？

![image](2.png)

为什么会出现这样的结果呢？

> # 原因探索

在C++中，缺省的参数是在编译时期绑定的，而不是在运行时期绑定的。这意味着当你在程序中写下`shape->draw()`的时候，编译器只会知道`shape`是一个`Shape*`类型的指针，
因此它会使用`Shape`类中定义的缺省参数值`Color::RED`。

- 在第一个例子中，即使`shape`实际上指向一个`Circle`对象，编译器在编译时并不知道这一点，所以它不会考虑`Circle`类中定义的缺省参数值`Color::GREEN`。而在实际运行过程中，
由于`draw`函数是虚函数，`shape`对象中会有一个虚函数表的指针，指向`Circle`类的`draw`函数实现，因此调用的是`Circle`类的`draw`函数，但传递的参数仍然是`Color::RED`。

- 在第二个例子中，`Shape`中的`draw`函数有一个缺省函数，而`Square`中的`draw`函数没有缺省参数，因此，当调用`shape->draw()`时，编译器会使用`Shape`类中的缺省参数`Color::RED`。而当调用`square->draw()`时，神奇的一幕发生了！尽管`square`并没有一个缺省参数的`draw`函数，但是由于`Shape`类的`draw`函数为它填充了一个缺省参数`Color::RED`，因此调用时并不会出现错误，而且还会使用这个缺省的参数值来执行。而`square->draw(Color::BLUE)`则是显式传递了一个参数`Color::BLUE`。这个行为是我们预期之内的。它合理的调用了`Square`类中的`draw`函数，并且传递了正确的参数。

> # 一些思考

这个例子可以帮助我们理解C++中缺省参数和虚函数的工作原理。为了避免这种混淆，建议在设计类层次结构时，避免在虚函数中使用缺省参数，或者严格确保所有派生类都使用相同的缺省参数值。
这个例子从侧面也可以看得出，C++为了榨取***性能***，做了一些设计上的妥协，然而，这些妥协有时会导致一些意想不到的行为。此正如孟子所云：“鱼（简洁）我所欲也，熊掌（***性能***）亦我所欲也，二者不可得兼，舍鱼（简洁）而取熊掌（***性能***）者也。”
也许这就是C++的真实写照吧。
