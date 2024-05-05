# 参数包

- `参数包(parameter packet)` 可变数目的参数。
- `模板参数包(template parameter packet)`表示零个或多个模板参数。
- `函数参数包(function parameter packet)` 表示零个或多个函数参数。

用省略号指出一个 `模板参数` 或 `函数参数` 表示一个包，如 `class...` 或者 `typename...`；一个类型名后面跟一个省略号表示零个或多个给定类型的 `非类型参数` 的列表(可以是一个函数的实参列表)。
```cpp
template<typename T, typename... Args>
void func(const T& t, const Args&... args){}
```
`sizeof...()` 可以获取模板参数包的参数个数和函数参数包的参数个数
# 参数包拓展

对包中的每一个元素都应用一个指定的模式,并得到展开后的逗号分隔的列表, 这里的 `模式` 通常为一些类型限定修饰符.通过在模式右边放一个省略号(...)触发扩展操作。`const Args&... args` 扩展模板参数包,将 `const type&` 应用到包中的每一个元素
# 转发参数包

在 C++ 中，完美转发（perfect forwarding）是通过使用右值引用来实现的。转发参数包（forwarding parameter pack）通常用于在一个函数中接受一组参数，并将这组参数传递给另一个函数。右值引用（`&&`）在这个上下文中的主要优势是它允许将原始调用中的左值和右值的引用性质保持不变地传递到下一层函数。

使用右值引用和完美转发的主要目的是避免不必要的拷贝或移动操作，提高性能。在函数模板中使用右值引用，可以在传递参数时保留参数的原始引用性质，同时避免不必要的数据复制。
# Fold expressions (since C++17)

- 折叠表达式必须用括号包围起来
- `op` 以下 32 个二元运算符之一 + - * / % ^ & | = < > << >> += -= *= /= %= ^= &= |= <<= >>= == != <= >= && || , .* ->*
- `pack` 参数包
- `init` 初始值

## 一元右折叠 Unary right fold
```
( pack op ... )

( E op ... )   =>   (E1 op (... op (EN-1 op EN)))
```
# 一元左折叠 Unary left fold
```
( ... op pack )

( ... op E )   =>   (((E1 op E2) op ...) op EN)
```
## 二元右折叠 Binary right fold
```
( pack op ... op init )

( E op ... op I )   =>   (E1 op (... op (EN−1 op (EN op I))))
```
## 二元左折叠
```
( init op ... op pack )

( I op ... op E )   =>   ((((I op E1) op E2) op ...) op EN)
```

```cpp
template<typename... Args>
bool all(Args... args) { return (... && args); }
 
bool b = all(true, true, true, false);
// within all(), the unary left fold expands as
//  return ((true && true) && true) && false;
// b is false

template<typename... Args>
int sum(Args&&... args)
{
//  return (args + ... + 1 * 2);   // Error: operator with precedence below cast
    return (args + ... + (1 * 2)); // OK
}

template <typename... Args>
void printer(Args &&... args)
{
  (std::cout << ... << args) << '\n';
}
```