# typedef
***

## Contents
- [typedef](#typedef)
  - [Contents](#contents)
  - [描述](#描述)
  - [主要应用形式](#主要应用形式)
    - [1.为基本数据类型定义新的类型名](#1为基本数据类型定义新的类型名)
    - [2.为自定义数据类型（结构体、公用体和枚举类型）定义简洁的类型名称](#2为自定义数据类型结构体公用体和枚举类型定义简洁的类型名称)
    - [3.为数组定义简洁的类型名称](#3为数组定义简洁的类型名称)
    - [4.为指针定义简洁的名称](#4为指针定义简洁的名称)
  - [`typedef` 与 `#define` 的区别](#typedef-与-define-的区别)
  - [用typedef定义函数指针](#用typedef定义函数指针)
  - [参考资料](#参考资料)

## 描述
在现实生活中，信息的概念可能是长度，数量和面积等。在C语言中，信息被抽象为 `int`、`float` 和 `double` 等基本数据类型。从基本数据类型名称上，不能够看出其所代表的物理属性，并且 `int`、`float` 和 `double` 为系统关键字，不可以修改。为了解决用户自定义数据类型名称的需求，C语言中引入类型重定义语句 `typedef`，可以为数据类型定义新的类型名称，从而丰富数据类型所包含的属性信息。

## 主要应用形式

### 1.为基本数据类型定义新的类型名
语法描述：`typedef data_type name`，示例：
```cpp
typedef unsigned double length;
unsigned double a;
length b;
```
如上，第二三行代码是等价的。

### 2.为自定义数据类型（结构体、公用体和枚举类型）定义简洁的类型名称
如结构体 `struct`，在较老的C++版本中，每次定义变量时都需要保留 `struct` 关键字，如下：
```cpp
struct Point
{
    double x;
    double y;
    double z;
};
struct Point point_1;
```
这就稍显繁琐，于是我们可以这样：
```cpp
typedef struct Point
{
    double x;
    double y;
    double z;
}Point;
Point point_2;
```

### 3.为数组定义简洁的类型名称
```cpp
typedef int int_array_10[10];
typedef int int_array_20[20];

int_array_10 a;
int a[10];

int_array_20 b;
int b[20];
```

### 4.为指针定义简洁的名称
普通指针：
```cpp
typedef char* STRING;
char* ch;
STRING ch;
```
函数指针：`typedef return_type (*name)(data_type_1,...)`
```cpp
typedef int (*myfunc)(int, int)
myfunc MAX;
int max(int a, int b);
MAX = max;
```
其中myfunc代表指向函数的指针类型的新名称

## `typedef` 与 `#define` 的区别
  1. `#define` 是预处理指令，在编译预处理时进行简单的替换，不作正确性检查，不关含义是否正确照样带入，只有在编译已被展开的源程序时才会发现可能的错误并报错。例如： `#define PI 3.1415926` 程序中的：`area=PI*r*r` 会替换为 `3.1415926*r*r` 如果你把 `#define` 语句中的数字9写成字母g预处理也照样带入。
  2. `typedef int * int_ptr_td` 与 `#define int_ptr_df int *` 作用都是用 `int_ptr` 代表 `int *` ,但是二者不同，正如前面所说 ，`#define` 在预处理 时进行简单的替换，而 `typedef` 不是简单替换 ，而是采用如同定义变量的方法那样来声明一种类型。
  3. 也许您已经注意到 `#define` 不是语句 不要在行末加分号，否则会连分号一块置换。

区别如下：
```cpp
typedef int* int_ptr_td;
#define int_ptr_df int*
int_ptr_td a, b;        //等同于 int* a; int* b; a、b都是指针
int_ptr_df c, d;        //等同于 int* c, d; c为指针，d为 int 变量
```
同时还可以用以下方法加以区别：
```cpp
typedef int* pint;
#define PINT int*
const pint p ;//等同于 int* const p ,p不可更改，但p指向的内容可更改
const PINT p ;//等同于 const int* p ,p可更改，但是p指向的内容不可更改。
```
`pint` 是一种指针类型 `const pint p` 就是把指针给锁住了 p不可更改
而 `const PINT p` 是 `const int * p` 锁的是指针p所指的对象。

## 用typedef定义函数指针
- `typedef 返回类型(*新类型)(参数表)`
```cpp
typedef char (*PTRFUN)(int);
PTRFUN pFun;
char glFun(int a){ return;}
voidmain()
{
pFun = glFun;
(*pFun)(2);
}
```
- `typedef 返回类型(类名::*新类型)(参数表)`
```cpp
class CA
{
public:
    char lcFun(int a){ return; }
};
CA ca;
typedef char (CA::*PTRFUN)(int);
PTRFUN pFun;
void main()
{
pFun = CA::lcFun;
ca.(*pFun)(2);
}
```
在这里，指针的定义与使用都加上了“类限制”或“对象”，用来指明指针指向的函数是那个类的。这里的类对象也可以是使用 `new` 得到的。比如：
```cpp
CA *pca =new CA;
pca->(*pFun)(2);
delete pca;
```
而且这个类对象指针可以是类内部成员变量，你甚至可以使用 `this` 指针。比如：
```cpp
//类CA有成员变量PTRFUN m_pfun;
void CA::lcFun2()
{
(this->*m_pFun)(2);
}
```
一句话，使用类成员函数指针必须有 `->*` 或 `.*` 的调用。

## 参考资料
- [C++ typedef用法详解](https://www.cnblogs.com/seventhsaint/archive/2012/11/18/2805660.html)