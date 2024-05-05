# unique_ptr

拥有独有对象所有权语义的智能指针，通过指针**占有（不共享）** 并管理另一对象，并在 `unique_ptr` 离开作用域时释放该对象的智能指针。

- `release` 返回一个指向被管理对象的指针，并释放所有权
- `reset` 替换被管理对象
- `swap` 交换被管理对象
- `get` 返回指向被管理对象的指针
- 类满足可移动构造 (MoveConstructible) 和可移动赋值 (MoveAssignable)的要求，但不满足可复制构造 (CopyConstructible)或可复制赋值 (CopyAssignable)的要求。
> 可移动构造：指定该类型的实例可以从一个[右值]( https://zh.cppreference.com/w/cpp/language/value_category "cpp/language/value category")实参构造。
> 可移动赋值：指定该类型的实例可以从[右值]( https://zh.cppreference.com/w/cpp/language/value_category "cpp/language/value category")实参赋值。
> 可复制构造：指定该类型的实例可以从[左值表达式]( https://zh.cppreference.com/w/cpp/language/value_category "cpp/language/value category")进行复制构造。（泛左值：一个表达式，其值可确定某个对象或函数的标识）
> 可复制赋值：指定该类型的实例可从[左值表达式](https://zh.cppreference.com/w/cpp/language/value_category "cpp/language/value category")复制赋值。


# shared_ptr

拥有共享对象所有权语义的智能指针，通过指针**保持对象共享所有权**的智能指针。多个 `shared_ptr` 对象可占有同一对象。

`shared_ptr` 的所有特化满足[可复制构造 (CopyConstructible)](https://zh.cppreference.com/w/cpp/named_req/CopyConstructible "cpp/named req/CopyConstructible") 、[可复制赋值 (CopyAssignable)](https://zh.cppreference.com/w/cpp/named_req/CopyAssignable "cpp/named req/CopyAssignable") 和[可小于比较 (LessThanComparable)](https://zh.cppreference.com/w/cpp/named_req/LessThanComparable "cpp/named req/LessThanComparable") 的要求并[可按语境转换](https://zh.cppreference.com/w/cpp/language/implicit_conversion "cpp/language/implicit conversion")为 `bool` 。