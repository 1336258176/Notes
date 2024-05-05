# 万能引用
所谓的万能引用（又称转发引用），即**接受左值表达式那形参类型就推导为左值引用，接受右值表达式，那就推导为右值引用**。
```cpp
template<typename T>
void f(T&&t){} // T&& 为万能引用

int a = 10;
f(a);       // a 是左值表达式，f 是 f<int&> 但是它的形参类型是 int&
f(10);      // 10 是右值表达式，f 是 f<int> 但它的形参类型是 int&&
```

# 引用折叠
**右值引用的右值引用折叠成右值引用，所有其他组合均折叠成左值引用**。
```cpp
typedef int&  lref;
typedef int&& rref;
int n;
 
lref&  r1 = n; // r1 的类型是 int&
lref&& r2 = n; // r2 的类型是 int&
rref&  r3 = n; // r3 的类型是 int&
rref&& r4 = 1; // r4 的类型是 int&&
```

# 三目表达式
三目表达式要求第二项和第三项之间能够隐式转换，然后整个表达式的类型会是 **“公共”类型**。
```cpp
using T = decltype(true ? 1 : 1.2);   //T -> double
using T2 = decltype(false ? 1 : 1.2); //T2-> double
```

```cpp
using namespace std::string_literals;

template<typename T1,typename T2,typename RT = 
    decltype(true ? T1{} : T2{}) >
RT max(const T1& a, const T2& b) { // RT 是 std::string
    return a > b ? a : b;
}

int main(){
    auto ret = ::max("1", "2"s);
    std::cout << ret << '\n';
}
```

# 重载函数模板
通常优先选择非模板的函数。

```cpp
template<typename T>
void test(T) { std::puts("template"); }

void test(int) { std::puts("int"); }

test(1);        // 匹配到test(int)
test(1.2);      // 匹配到模板
test("1");      // 匹配到模板
```

# 可变参数模板
- **形参包**：**类型形参包**是接受零个或更多个模板实参（非类型、类型或模板）的模板形参。**函数形参包**是接受零个或更多个函数实参的函数形参。
- **形参包展开**：
	- 模式：后随省略号且其中至少有一个形参包的名字的**模式**会被展开成零个或更多个**逗号分隔**的模式实例。

```cpp
// 类型形参包：Args 函数形参包：args
// (std::cout << args << ' ' ,0)... 形参包展开
// (std::cout << args << ' ' ,0) 模式，即形参包中的参数都按照此种模式以逗号表达式展开
template<typename...Args>
void print(const Args&...args){
    int _[]{ (std::cout << args << ' ' ,0)... };
}

int main() {
    print("luse", 1, 1.2);
}
```
实现 `sum`：
```cpp
#include <iostream>
#include <numeric>
#include <type_traits>

template<typename...Args,typename RT = std::common_type_t<Args...>>
RT sum(const Args&...args) {
    RT _[]{ static_cast<RT>(args)... };
    RT n{};
    for (int i = 0; i < sizeof...(args); ++i) {
        n += _[i];
    }
    return n;
}

// 简化后的函数
template<typename...Args,typename RT = std::common_type_t<Args...>>
RT sum(const Args&...args) {
    RT _[]{ args... };
    return std::accumulate(std::begin(_), std::end(_), RT{});
}

int main() {
    double ret = sum(1, 2, 3, 4, 5, 6.7);
    std::cout << ret << '\n';       // 21.7
}
```

# 类模板推导指引
- C++17 之后才有的类模板**自动推导**
```cpp
template<class T>
struct A{
    A(T, T);
};
auto y = new A{1, 2}; // 分配的类型是 A<int>
```

- 用户定义的**推导指引**（显式指定不用于推导指引）
```cpp
template<typename T>
struct Test{
    Test(T v) :t{ v } {}
private:
    T t;
};

//自动推导为int，让他实际成为std::size_t
Test(int) -> Test<std::size_t>;

Test t(1);      // t 是 Test<size_t>
```

```cpp
template<class Ty, std::size_t size> 
struct array { 
	Ty arr[size]; 
};


```