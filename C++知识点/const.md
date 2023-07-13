# 常量指针

即指向一个常量的指针（不能修改指针指向地址上的值）

```cpp
const int* p = &a;
```

# 指针常量

即这个指针是个常量（不能修改指针指向）

```cpp
int* const p = &a;
```

# 作用于引用&

```cpp
int const& a = x;
const int& a = x;
int &const a = x;//这种方式定义是C、C++编译器未定义，虽然不会报错，但是该句效果和int &a一样。
```
以上两种方式是等价的，此时引用 a 不能被更新。

# 作用于类成员函数后面

- 声明一个成员函数的时候用 `const` 关键字是用来说明这个函数是 “只读(read-only)”函数，也就是**说明这个函数不会修改任何数据成员(object)**。为了声明一个 `const` 成员函数，把 `const` 关键字放在函数括号的后面。声明和定义的时候都应该放 const 关键字。
- 另外要说明的是：被声明为 `const` 的成员函数，即不能修改普通成员变量，也不能修改 `const` 成员变量。

示例：
```cpp
#include<iostream>
using namespace std;
class temp
{
public:
    temp(int age);
    int getAge() const;
    void setNum(int num);
private:
    int age;
};

temp::temp(int age)
{
    this->age = age;
}

int temp::getAge() const
{
    age+=10; // #Error...error C2166: l-value specifies const object #
    return age;
}

void main()
{
    temp a(22);
    cout << "age= " << a.getAge() << endl;
}
```

# 参考资料

- [常量指针和指针常量-每日一个 C++/AI 知识点系列](https://www.bilibili.com/video/BV15M4y1e7R3/?spm_id_from=444.64.top_right_bar_window_history.content.click&vd_source=6b66c0593ea4047913f9c45aa790afa1)
- [C++函数后面加const修饰](https://blog.csdn.net/lixpjita39/article/details/106440569)