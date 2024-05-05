# RAII（Resource Acquisition Is Initialization）
RAII 是 C++ 中流行的习惯用法，专注于使用对象的生命周期来管理资源。它鼓励将资源生命周期绑定到相应对象的范围，以便在创建对象时自动获取它，并在对象销毁时自动释放它。这有助于简化代码、避免泄漏并有效管理资源。

# Pimpl（Pointer-to-Implementation）
通过使用私有结构或类的前向声明来隐藏类的实现细节，保持类的公共接口干净，并减少编译时依赖。

# CRTP（Curiously Recurring Template Pattern）
CRTP 可用于实现“编译时多态性”，即基类公开一个接口，派生类实现这样的接口。当您想要在基类中自定义某些行为而不增加虚拟函数调用的开销时，通常会使用 CRTP。简而言之，CRTP可以用于实现编译时多态性，而无需运行时性能成本。

```cpp
#include <cstdio>
 
#ifndef __cpp_explicit_this_parameter // Traditional syntax
#注意this的显式调用只用C++20以后才支持

template <class Derived>
struct Base { void name() { (static_cast<Derived*>(this))->impl(); } };
struct D1 : public Base<D1> { void impl() { std::puts("D1::impl()"); } };
struct D2 : public Base<D2> { void impl() { std::puts("D2::impl()"); } };
 
void test()
{
    // Base<D1> b1; b1.name(); //undefined behavior
    // Base<D2> b2; b2.name(); //undefined behavior
    D1 d1; d1.name();
    D2 d2; d2.name();
}
 
#else // C++23 alternative syntax; https://godbolt.org/z/s1o6qTMnP
 
struct Base { void name(this auto&& self) { self.impl(); } };
struct D1 : public Base { void impl() { std::puts("D1::impl()"); } };
struct D2 : public Base { void impl() { std::puts("D2::impl()"); } };
 
void test()
{
    D1 d1; d1.name();
    D2 d2; d2.name();
}
 
#endif
 
int main()
{
    test();
}
```

# Non-Copyable
不可复制习惯用法是一种 C++ 设计模式，可防止对象被复制或分配。它通常应用于管理资源的类，例如文件句柄或网络套接字，其中复制对象可能会导致资源泄漏或双重删除等问题。要使类不可复制，您需要删除复制构造函数和复制赋值运算符。这可以在类声明中显式完成，让其他程序员清楚地知道不允许复制。

```cpp
class NonCopyable {
public:
  NonCopyable() = default;
  ~NonCopyable() = default;

  // Delete the copy constructor
  NonCopyable(const NonCopyable&) = delete;

  // Delete the copy assignment operator
  NonCopyable& operator=(const NonCopyable&) = delete;
};

class MyClass : private NonCopyable {
  // MyClass is now non-copyable
};
```

# Erase-remove
擦除-删除习惯用法是一种常见的 C++ 技术，用于有效地从容器中删除元素，特别是从标准序列容器（如 `std::vector` 、 `std::list` 和 `std::deque` ）中删除元素。它利用标准库算法 `std::remove` （或 `std::remove_if` ）和成员函数 `erase()` 。

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> numbers = {1, 3, 2, 4, 3, 5, 3};
    
    // Remove all occurrences of 3 from the vector.
    numbers.erase(std::remove(numbers.begin(), numbers.end(), 3), numbers.end());

    for (int number : numbers) {
        std::cout << number << " ";
    }

    return 0;
}
```

# Copy and Swap
copy-swap 是一种 c++习惯用法，它利用复制构造函数和 swap 函数来创建一个赋值操作符。它遵循一个简单但功能强大的范例:创建右侧对象的临时副本，并与左侧对象交换其内容。

```cpp
class String {
    // ... rest of the class ...

    String(const String& other);
    
    void swap(String& other) {
        using std::swap; // for arguments-dependent lookup (ADL)
        swap(size_, other.size_);
        swap(buffer_, other.buffer_);
    }

    String& operator=(String other) {
        swap(other);
        return *this;
    }
};
```

# Copy-Write
Copy-Write 习惯用法，有时也称为 write-on-copy (CoW)或“惰性复制”习惯用法，是一种用于编程的技术，用于最小化复制大型对象的开销。通过使用对对象的共享引用和仅在需要修改时复制数据，它有助于减少实际复制操作的数量。

```cpp
#include <iostream>
#include <memory>

class MyString {
public:
    MyString(const std::string &str) : data(std::make_shared<std::string>(str)) {}

    // Use the same shared data for copying.
    MyString(const MyString &other) : data(other.data) { 
        std::cout << "Copied using the Copy-Write idiom." << std::endl;
    }

    // Make a copy only if we want to modify the data.
    void write(const std::string &str) {
        // Check if there's more than one reference.
        if(!data.unique()) {
            data = std::make_shared<std::string>(*data);
            std::cout << "Copy is actually made for writing." << std::endl;
        }
        *data = str;
    }

private:
    std::shared_ptr<std::string> data;
};

int main() {
    MyString str1("Hello");
    MyString str2 = str1; // No copy operation, just shared references.

    str1.write("Hello, World!"); // This is where the actual duplication happens.
    return 0;
}
```