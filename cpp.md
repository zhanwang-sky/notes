### 多继承时的指针偏移现象

```
// 有如下类定义
class Base1 {
 public:
  int x;
};

class Base2 {
 public:
  int x;
};

class Derived : public Base1, public Base2 {
 public:
  int x;
};

// 创建不同类型的指针，指向同一个派生类对象
Derived d;
Base1* p1 = &d;
Base2* p2 = &d;
Derived* p3 = &d;

// 则有：
p1 != p2; p1 == p3;
```
