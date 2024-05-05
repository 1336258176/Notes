# 拷贝构造函数

## 用途

**拷贝构造函数**是一种特殊的构造函数，它在创建对象时，是使用同一类中之前创建的对象来初始化新创建的对象。拷贝构造函数通常用于：

1. 通过使用另一个同类型的对象来初始化新创建的对象。
    
2. 复制对象把它作为参数传递给函数。
    
3. 复制对象，并从函数返回这个对象。

## 声明
```cpp
classname (const classname &obj) { 
// 构造函数的主体 
}
```

## 示例

```cpp
class A
{
    private:
        int a;
    public:
        A(){}
        ~A(){}
        A(const A& x){
            std::cout << "kao bei" << std::endl;
        }
};

void func(A a)
{
    std::cout << "hello" << std::endl;
}

int main(int argc, char** argv)
{
    A j;
    A k = j;   //调用拷贝构造函数，对应1
    A o(j);    //调用拷贝构造函数，对应1
    func(j);   //调用拷贝构造函数，对应2

    return 0;
}
```

# 静态成员

## 静态成员变量

### 说明
- 当我们声明类的成员为静态时，这意味着无论创建多少个类的对象，**静态成员都只有一个副本**。
- 静态成员在类的所有对象中是**共享**的。
- 如果不存在其他的初始化语句，在创建第一个对象时，所有的静态数据都会被初始化为零。我们不能把静态成员的初始化放置在类的定义中，但是可以在类的外部通过使用范围解析运算符 **::** 来重新声明静态变量从而对它进行初始化，如下面的实例所示。

### 示例

```cpp
#include <iostream>
 
using namespace std;
 
class Box
{
   public:
      static int objectCount;
      // 构造函数定义
      Box(double l=2.0, double b=2.0, double h=2.0)
      {
         cout <<"Constructor called." << endl;
         length = l;
         breadth = b;
         height = h;
         // 每次创建对象时增加 1
         objectCount++;
      }
      double Volume()
      {
         return length * breadth * height;
      }
   private:
      double length;     // 长度
      double breadth;    // 宽度
      double height;     // 高度
};
 
// 初始化类 Box 的静态成员
int Box::objectCount = 0;
 
int main(void)
{
   Box Box1(3.3, 1.2, 1.5);    // 声明 box1
   Box Box2(8.5, 6.0, 2.0);    // 声明 box2
 
   // 输出对象的总数
   cout << "Total objects: " << Box::objectCount << endl;
 
   return 0;
}
```

## 静态成员函数

### 说明
- 如果把函数成员声明为静态的，就可以把函数与类的任何特定对象独立开来。**静态成员函数即使在类对象不存在的情况下也能被调用**，静态函数只要使用类名加范围解析运算符 **::** 就可以访问。
- 静态成员函数只能访问静态成员数据、其他静态成员函数和类外部的其他函数。
- 静态成员函数有一个类范围，他们不能访问类的 this 指针。您可以使用静态成员函数来判断类的某些对象是否已被创建。

### 示例

```cpp
#include <iostream>
 
using namespace std;
 
class Box
{
   public:
      static int objectCount;
      // 构造函数定义
      Box(double l=2.0, double b=2.0, double h=2.0)
      {
         cout <<"Constructor called." << endl;
         length = l;
         breadth = b;
         height = h;
         // 每次创建对象时增加 1
         objectCount++;
      }
      double Volume()
      {
         return length * breadth * height;
      }
      static int getCount()
      {
         return objectCount;
      }
   private:
      double length;     // 长度
      double breadth;    // 宽度
      double height;     // 高度
};
 
// 初始化类 Box 的静态成员
int Box::objectCount = 0;
 
int main(void)
{
  
   // 在创建对象之前输出对象的总数
   cout << "Inital Stage Count: " << Box::getCount() << endl;
 
   Box Box1(3.3, 1.2, 1.5);    // 声明 box1
   Box Box2(8.5, 6.0, 2.0);    // 声明 box2
 
   // 在创建对象之后输出对象的总数
   cout << "Final Stage Count: " << Box::getCount() << endl;
 
   return 0;
}
```