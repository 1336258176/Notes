# explicit

- 在 C++中，`explicit` 关键字用来修饰类的构造函数，被修饰的构造函数的类，不能发生相应的隐式类型转换，只能以显示的方式进行类型转换。
- `explicit` 关键字作用于单个参数的构造函数（默认参数不算在内）。
- `explicit` 关键字只能用于类内部的构造函数声明上。

示例：
```cpp
#include <iostream>
using namespace std;

class Point {
public:
    int x, y;
    Point(int x = 0, int y = 0)
        : x(x), y(y) {}
};

void displayPoint(const Point& p) 
{
    cout << "(" << p.x << "," 
         << p.y << ")" << endl;
}

int main()
{
	displayPoint(1);//隐式类型转换
    Point p = 1;//在对象刚刚定义时, 即使你使用的是赋值操作符 = ，也是会调用构造函数, 而不是重载的 operator= 运算符。
}
```

修改后：
```cpp
#include <iostream>
using namespace std;

class Point {
public:
    int x, y;
    explicit Point(int x = 0, int y = 0)
        : x(x), y(y) {}
};

void displayPoint(const Point& p) 
{
    cout << "(" << p.x << "," 
         << p.y << ")" << endl;
}

int main()
{
    displayPoint(Point(1));
    Point p(1);
}
```

# friend

- 用于声明友元关系，允许一个类或者函数访问另一个类的私有成员。

# mutable

- 将成员变量声明为 `mutable`，表示即使在 `const` 成员函数（详见 [[const]]）中，该成员变量仍可以被修改。通常在需要缓存或者计数的情况下使用。

# constexpr

- 用于声明常量表达式，即在编译时求值的表达式。

# decltype

- 用于推导表达式的类型，并在编译时获取表达式的类型。

# alignas

- 用于指定对齐要求，可以应用于类型、变量或者成员变量，用于控制内存对齐方式。

# thread_local

- 用于声明线程局部存储，指示变量在每个线程中具有各自的副本。（详见[[Thread]]）

# 参考资料

- [C++ explicit 关键字](https://zhuanlan.zhihu.com/p/52152355)
- [C++ explicit的作用](https://www.cnblogs.com/this-543273659/archive/2011/08/02/2124596.html)
- Chat GPT