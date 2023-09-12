---
layout: post
title: cpp基础知识整理
category: cpp
tag:
---

### 构造函数的初始化列表

构造函数可以有初始化列表，用以初始化类中的数据成员。

```
class Point {
public:
    Point() : x(0), y(0) {}
    Point(int xx, int yy) : x(xx), y(yy) {}
private:
    int x, y;
};
```

注意：

- 初始化列表的执行顺序由数据成员在类中定义的先后顺序决定，与成员在初始化列表中的先后顺序无关。
- 调用构造函数时先执行初始化列表，再执行函数体内的语句。
- 以下3种情况必须使用初始化列表：
    - const成员
    - 引用类型成员
    - 没有默认构造函数的类类型成员

### 拷贝构造函数

拷贝构造函数可基于原对象创建一个新的副本，与普通构造函数不同的是，无论是否显式定义了拷贝构造函数，编译器都会生成一个合成的拷贝构造函数。拷贝构造函数的形参为本类型的const引用。

```
class Point {
public:
    Point(const Point &p) : x(p.x), y(p.y) {}
private:
    int x, y;
};
```

哪些情况下拷贝构造函数会被调用？

- 一个对象以值传递的方式传入函数
- 一个对象以值传递的方式从函数返回
- 一个对象通过另外一个对象进行初始化

什么情况下必须显示定义拷贝构造函数？

- 有指针类型的数据成员，或者在构造函数中有动态分配资源的成员，必须显式定义拷贝构造函数进行深拷贝

### 友元函数与友元类

友元可以直接访问类的私有成员。友元分为友元函数和友元类两种。

类有三种访问权限，而友元函数的声明可以放在任何地方，没有区别，只是声明该函数是类的一个友元函数，可以访问类中的私有成员。虽然友元函数的原型在类的定义中出现过，但它并不是类的函数成员。

```
class Point {
    friend void show(Point &p);
public:
    Point(int xx, int yy) : x(xx), y(yy) {}
private:
    int x, y;
};
void show(Point &p) {
    cout << "(" << p.x << "," << p.y << ")" << endl;
}
```

友元类的所有函数成员都是另一个类的友元函数，即友元类的所有函数成员都可以访问另一个类的私有成员。

```
class Point {
    friend class Print;
public:
    Point(int xx, int yy) : x(xx), y(yy) {}
private:
    int x, y;
};
class Print {
public:
    void printx(Point &p) { cout << "x=" << p.x << endl; }
    void printy(Point &p) { cout << "y=" << p.y << endl; }
};
int main() {
    Point a(3, 4);
    Print p;
    p.printx(a);
    p.printy(a);
    return 0;
}
```

### static成员

static成员与类绑定，不与特定的对象绑定，任何由这个类产生的对象都可以调用static成员。

访问static成员一般有3种方法：可通过对象访问，或者通过对象的引用与指针访问，也可直接使用类名来访问。

关于static函数成员，需注意以下几点：

- static函数成员不是类对象的组成部分，因而不能使用this指针。
- 如果在类外面对static函数成员进行定义，那么定义处不能使用static关键字。
- static函数成员只能使用static数据成员，不能访问非static数据成员。

对于static数据成员，它只能在类的外面进行定义和初始化。

```
class Point {
public:
    Point() : x(0), y(0) { cnt += 1; }
    ~Point() { cnt -= 1; }
    static void show();
private:
    int x, y;
    static int cnt;
};
int Point::cnt = 0;
void Point::show() {
    cout << "cnt=" << cnt << endl;
}
int main() {
    Point a, b;
    Point *p = &b;
    a.show();
    p->show();
    Point::show();
    return 0;
}
```

### 重载操作符

重载操作符可以是类的成员函数，也可以不是。当作为成员函数时，形参数比操作符的操作数少1；而对于非成员函数的重载操作符，形参数与操作符的操作数相同。

```
class Point {
public:
    Point(int xx, int yy) : x(xx), y(yy) {}
    bool operator==(const Point &p) const { return x == p.x && y == p.y; }
    bool operator!=(const Point &p) const { return !(*this == p); }
    bool operator<(const Point &p) const { return x < p.x || (x == p.x && y < p.y); }
    bool operator>(const Point &p) const { return p < *this; }
    bool operator<=(const Point &p) const { return !(p < *this); }
    bool operator>=(const Point &p) const { return !(*this < p); }
private:
    int x, y;
};
int main() {
    Point a(3, 4), b(2, 5), c(3, 4);
    if (a != b) cout << "a!=b" << endl;
    if (a == c) cout << "a==c" << endl;
    if (a > b) cout << "a>b" << endl;
    return 0;
}
```

下面是将重载操作符定义为非成员函数的示例。

```
class Point {
public:
    Point() : x(0), y(0) {}
    Point(int xx, int yy) : x(xx), y(yy) {}
public:
    int x, y;
};
Point operator+(Point &a, Point &b) {
    Point c;
    c.x = a.x + b.x;
    c.y = a.y + b.y;
    return c;
}
int main() {
    Point a(3, 4), b(5, 5), c;
    c = a + b;
    return 0;
}
```

重载输入输出操作符的一般定义为：

```
istream& operator>>(istream &in, 类名 &obj) {
    in >> something;
    return in;
}
ostream& operator<<(ostream &out, const 类名 &obj) {
    out << something;
    return out;
}
```

下面是示例程序。

```
class Point {
public:
    Point() : x(0), y(0) {}
    friend ostream& operator<<(ostream &out, const Point &p) {
        out << "x=" << p.x << ", y=" << p.y;
        return out;
    }
    friend istream& operator>>(istream &in, Point &p) {
        in >> p.x >> p.y;
        return in;
    }
private:
    int x, y;
};
int main() {
    Point a;
    cin >> a;
    cout << a << endl;
    return 0;
}
```

自增与自减操作符有前置和后置两种形式，为了区分，定义方式如下：

```
类名& operator++();     // 前置++
类名& operator--();     // 前置--
类名 operator++(int);   // 后置++
类名 operator--(int);   // 后置--
```

以自增为例，示例代码如下：

```
class Point {
public:
    Point(int xx, int yy) : x(xx), y(yy) {}
    Point& operator++() {
        x += 1;
        return *this;
    }
    Point operator++(int) {
        Point r(*this);
        y += 1;
        return r;
    }
    friend ostream& operator<<(ostream &out, const Point &p) {
        out << "x=" << p.x << ", y=" << p.y;
        return out;
    }
private:
    int x, y;
};
int main() {
    Point a(2, 4);
    cout << ++a << endl;
    cout << a << endl;
    cout << a++ << endl;
    cout << a << endl;
    return 0;
}
```
