# Pointer
***

## Contents
- [Pointer](#pointer)
  - [Contents](#contents)
  - [前言](#前言)
  - [函数指针](#函数指针)
  - [参考资料](#参考资料)

## 前言
指针是C++面型对象中比较难的一部分，所以更有必要记录一下其中的难点、易混点，希望能更好地帮助大家。

## 函数指针
函数指针就是指向函数的指针，函数指针的声明类似函数的声明，只不过将函数名变成了 `(*func_pointer)` 形式如下：
```
data_type (* func_pointer)(data_type arg1, data_type arg2,..., data_type argn);
```
示例：
```cpp
void (*test_p)(int);                //声明指针，函数指针两边的括号必不可少

void test(int a)                    //函数定义
{
    std::cout << ++a << std::endl;
}

test_p = test;                      //函数的函数名实际上就是一个指针，函数名指向该函数的代码在内存中的首地址。
(*test_p)(9);                       //调用函数指针
```
输出结果为10

最近在写OpenCV程序的时候，想实现一个调色器，但发现 `createTrackbar` 无法写入类中，原因是类成员函数无法作为该函数形参中回调函数的实参，如下：
```cpp
class a
{
public:
    int m = 0;
    int count = 100;
    void callback(int pos, void *){}
    void make_window()
    {
        cv::createTrackbar("test","test",&m,count,this->callback);
    }
};
```
这样写是错误的，原因就出在 `this->callback` 上，要想理解具体原因，我们首先要看看 `createTrackbar` 及对应回调函数的原型，如下：
```cpp
CV_EXPORTS int createTrackbar(const String& trackbarname, const String& winname,
                              int* value, int count,
                              TrackbarCallback onChange = 0,
                              void* userdata = 0);

typedef void (*TrackbarCallback)(int pos, void* userdata);
```
众所周知，要想使用类成员函数，必须加上范围解析运算符 `::` 而函数原型声明中并没有相应的模板，所以我们可以将其看成OpenCV开发人员的疏忽。同时我发现一个ROS的话题订阅函数，同样需要输入一个函数作为回调函数，它却能写入类中，函数原型如下：
```cpp
template<class M, class T>
Subscriber subscribe(const std::string& topic, uint32_t queue_ size,
                    void(T::*fp)( const boost::shared_.ptr<M const>&), T* obj,
                    const TransportHints& transport_hints = TransportHints())
```
在这里，指针的定义与使用都加上了“类限制”或“对象”，用来指明指针指向的函数是那个类的。可见，与 `createTrackbar` 不同的是，`subscribe` 中回调函数形参有类限制，这样就避免了实参与形参不一致的问题。

## 参考资料
- [C++ 指针](https://www.runoob.com/cplusplus/cpp-pointers.html)
- [C/C++ 函数指针使用总结](https://www.cnblogs.com/lvchaoshun/p/7806248.html)