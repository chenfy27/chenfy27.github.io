---
layout: post
title: cpp多态基础
category: cpp
tags:
---

### 构造与析构函数调用顺序

设A为父类，B为A的子类，那么在定义B类的实例时，先调用A的构造函数，再调用B的构造函数，而析构函数的调用顺序正好相反。

```cpp
class A {
public:
    A() { cout << "construct A" << endl; }
    ~A() { cout << "destory A" << endl; }
};
class B : public A {
public:
    B() { cout << "construct B" << endl; }
    ~B() { cout << "destroy B" << endl; }
};
int main() {
    B b;
    return 0;
}
```

运行结果如下：

```
construct A
construct B
destroy B
destory A
```

### 继承方式与访问权限

<table border="1" cellspacing="0" style="width:500px; height:100px; text-align:center" cellpadding="5">
<tr><th>继承方式</th><th>成员属性</th><th>访问权限</th></tr>
<tr><td rowspan="3">public</td><td>public</td><td>public</td></tr>
<tr><td>protected</td><td>protected</td></tr>
<tr><td>private</td><td>no access</td></tr>
<tr><td rowspan="3">protected</td><td>public</td><td>protected</td></tr>
<tr><td>protected</td><td>protected</td></tr>
<tr><td>private</td><td>no access</td></tr>
<tr><td rowspan="3">private</td><td>public</td><td>private</td></tr>
<tr><td>protected</td><td>private</td></tr>
<tr><td>private</td><td>no access</td></tr>
</table>

### 多态

多态指用一个名字定义不同的函数，该函数执行不同但类似的操作，从而实现\"一个接口，多种方法\"。

多态分静态联编和动态联编。静态联编发生在编译阶段，比如函数重载和运算符重载，在编译期间就可以决定调用的是哪个重载函数或运算符。动态联编则发生在运行阶段，比如继承和虚函数。

```cpp
class Shape {
public:
    virtual void show() { cout << "i am shape" << endl; }
};
class Circle : public Shape {
    void show() { cout << "i am circle" << endl; }
};
class Rectangle : public Shape {
    void show() { cout << "i am rectangle" << endl; }
};
int main() {
    Shape s, *p;
    Circle c;
    Rectangle r;
    p = &s; p->show();
    p = &c; p->show();
    p = &r; p->show();
    return 0;
}
```

运行结果：

```
i am shape
i am circle
i am rectangle
```
