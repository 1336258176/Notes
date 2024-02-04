> **模板是对代码的描述**。
# Declaration and Definition
类模板（class template）、函数模板（function template）、变量模板（variable template）、别名模板（alias template）、和概念（concept）

```cpp
// declarations
template <typename T> struct  class_tmpl;
template <typename T> void    function_tmpl(T);
template <typename T> T       variable_tmpl;          // since c++14
template <typename T> using   alias_tmpl = T;         // since c++11
template <typename T> concept no_constraint = true;   // since c++20
```
前三种模板都可以拥有定义（definition），而后两种模板不需要提供定义，因为它们不产生运行时的实体
```cpp
// definitions
template <typename T> struct  class_tmpl {};
template <typename T> void    function_tmpl(T) {}
template <typename T> T       variable_tmpl = T(3.14);
```
# Parameters and Arguments
非类型模板形参（Non-type Template Parameters）、类型模板形参（Type Template Parameters）和模板模板形参（Template Template Parameters）
```cpp
// There are 3 kinds of template parameters:
template <int n>                               struct NontypeTemplateParameter {};
template <typename T>                          struct TypeTemplateParameter {};
template <template <typename T> typename Tmpl> struct TemplateTemplateParameter {};
```
要注意的是，非类型模板实参必须是常量，因为模板是在编译期被展开的，在这个阶段只有常量，没有变量。
# Template Instantiation
按需（或隐式）实例化（on-demand (or implicit) instantiation） 和显示实例化（explicit instantiation）
```cpp
// t.hpp
template <typename T> void foo(T t) { std::cout << t << std::endl; }

// t.cpp
// on-demand (implicit) instantiation
#include "t.hpp"
foo<int>(1);
foo(2);
std::function<void(int)> func = &foo<int>;

// explicit instantiation
#include "t.hpp"
template void foo<int>(int);
template void foo<>(int);
template void foo(int);
```
# Template Specialization
模板的特化（Template Specialization）允许我们替换一部分或全部的形参，并定义一个对应改替换的模板实现。其中，替换全部形参的特化称为[全特化（Full Specialization）](https://en.cppreference.com/w/cpp/language/template_specialization)，替换部分形参的特化称为[偏特化（Partial Specialization）](https://en.cppreference.com/w/cpp/language/partial_specialization)，非特化的原始模板称为主模板（Primary Template）。只有类模板和变量模板可以进行偏特化，函数模板只能全特化。在实例化模板的时候，编译器会从所有的特化版本中选择最匹配的那个实现来做替换（Substitution），如果没有特化匹配，那么就会选择主模板进行替换操作。
```cpp
// function template
template <typename T, typename U> void foo(T, U)       {}     // primary template
template <>                       void foo(int, float) {}     // full specialization

// class template
template <typename T, typename U> struct S             {};    // #1, primary template
template <typename T>             struct S<int, T>     {};    // #2, partial specialization
template <>                       struct S<int, float> {};    // #3, full specialization

S<int, int>();      // choose #2
S<int, float>();    // choose #3
S<float, int>();    // choose #1
```
这里你可能已经注意到了，特化声明与显式实例化（explicit instantiation）的语法非常相似，注意不要混淆了。
```cpp
// Don't mix the syntax of "full specialization declaration" up with "explict instantiation"
template    void foo<int, int>;   // this is an explict instantiation
template <> void foo<int, int>;   // this is a full specialization declaration
```
# TMP
```cpp
// primary template
template <int N>  // non-type parameter N
struct binary {
  // an template instantiation inside the template itself, which contructs a recursion
  static constexpr int value = binary<N / 10>::value << 1 | N % 10;
};
// full specialization when N == 0
template <> 
struct binary<0> {
  static constexpr int value = 0;
};

std::cout << binary<101>::value << std::endl;    // instantiation
```
1. 模板像函数一样被使用，模板的形参就像是函数的形参，模板的静态成员作为函数的返回值。
2. 通过实例化（Instantiation）来“调用”模板。
3. 通过特化（Specialization）和重载（Overloading）来实现分支选择。
4. 通过递归来实现循环逻辑。
5. 所有过程发生在编译期间，由编译器驱动。
# 参考资料
- https://zhuanlan.zhihu.com/p/384826036