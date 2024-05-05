# 进阶
这部分内容为 C++进阶，对初学者来说难度较大，希望大家能够坚持学习下去~~
## 面向对象
首先，什么是面向对象呢？面向对象编程——Object Oriented Programming，简称 OOP，是一种程序设计思想。OOP 把对象作为程序的基本单元，一个对象包含了数据和操作数据的函数。**OOP 强调的是程序如何表示数据，而不关心实现。**
### 类和对象
类用于指定对象的形式，是一种用户自定义的数据类型，它是一种封装了数据和函数的组合。类中的数据称为**成员变量**，函数称为**成员函数**。类可以被看作是一种模板，可以用来创建具有相同属性和行为的多个对象，如一个学生类，它具有某些相似的特征（年龄、身高、体重等）。下面我们用一个例子来说明 C++中类的定义。
#### 定义
```c++
class Student          // 类的声明使用关键字 class
{        
  public:              // 类的成员修饰符：public、private、projected
    int age_;          // 变量
    double height_;
    double weight_;

	// 类内声明的函数，有两种定义方法
	// 第一种函数定义方法：类外定义
	double GetWeight(void) const;
	// 第二种函数定义方法：类内定义（默认为内联函数）
	double GetWeight(void) const { return weight_; }
	double age(void) const { return age_; }
};                     // 分号必不可少

// 类外定义
// 需要为定义的函数加上范围（即要定义的函数隶属于哪个类，本例中属于student）和范围解析运算符（::）
double Student::GetWeight(void) const {
  return weight_;
}

// 上面已经自定义了一个student类，那么下面可以直接使用自定义的类型来定义一个变量
Student xiaoming;
```
习惯上，我们将那些短小的函数放在类声明中，即内联函数，以提高效率，而将其他的函数实现和声明分开（通常声明在头文件中，定义在相应的源文件中），这样可以增强代码的封装性。
还有一点需要说明的是：实例化的对象共用相同的类方法（即它们共享类方法的存储空间，不再随对象的不同而额外分配存储空间），而它们拥有各自独立的数据（即每个实例都会为其数据单独分配内存空间）。
使用我们自定义的类来声明变量就叫“**实例化**”，这个变量就叫该对象的“**实例**”。如下示例，`xiaoming` 这个变量就是 `student` 类的一个实例，它拥有自己的数据而共享 `student` 类方法。
```c++
Student xiaoming;
```
#### 成员修饰符
C++中类内成员修饰符分为三种：`public`、`private`、`projected`（区别主要在于继承部分）
##### public
顾名思义，`public` 修饰的成员是公有的，即**类内类外均可以访问，类外访问时必须借助于实例**，下面以 `weight` 变量为例说明：
```c++
class Student{
  public:
    int age_;
    double height_;
    double weight_;        // 声明变量

	double SetWeight(double weight_);
};

void Student::SetWeight(double weight_){
  weight = weight_;  // 类内访问
}

Student xiaoming;
cout << xiaoming::weight << endl;  // 类外访问
```
##### private
顾名思义，`private` 修饰的成员是私有的，即只有类内成员可以访问，类外访问时会报错。
```c++
class Student{
// 若什么修饰符都不加，则定义的成员默认为private
  private:
    double weight_;       // 声明变量
  public:
    int age_;
    double height_;

	double SetWeight(double weight);
	double GetWeight(void) const ;
};

void Student::SetWeight(double weight){
  weight_ = weight;
}

double Student::GetWeight(void) const {
  return weight_;    // 类内访问
}
```
![[Snipaste_2023-10-19_21-24-32.png]]
公有成员函数是程序和对象私有成员之间的桥梁。我们无法从类外部直接访问私有成员，而只能借助共有函数，有时我们会特别地抽象出更具代表性的公有函数，将这些函数作为“**接口**”提供给用户，以便用户“访问”私有成员。
将实现细节与抽象分开被称为“**封装**”。 `private` 为我们提供了一种封装的方法，利于我们实现“**数据隐藏**”，即将实现的细节隐藏在私有部分。封装的另一个例子是，将类的定义与声明放在不同的文件中。这些都是 OOP 的特性。
#### 类的基本函数
##### 构造函数
上面我们讲了如何声明和定义一个类，那么我们如何构造一个这样的对象呢？C++为我们提供了一种方法--构造函数。构造函数可以让我们构造一个新对象同时将值赋给数据成员。构造函数的名字和类名相同，但和普通函数不同的是它没有返回值，参数有无均可。
```c++
class Student{
  public:
	Student(){
		cout << "This is the constructor." << endl;
	}
};

Student xiaoming;
```
我们构造类的新对象时编译器就会默认调用这个类的构造函数，就像上面我们创建的 `xiaoming`，会默认调用 `student()` 构造函数。同时构造函数为我们提供了一种初始化数据成员的方法，我们可以通过向构造函数传入参数来初始化数据成员，示例如下：
```c++
class Student{
  private:
    int age_;
  public:
    Student(int age){ age_ = age; }
}

Student xiaoming(14);
```
需要说明的是，即使我们不显式地定义构造函数，编译器也会为我们自动生成一个构造函数，缺点是自动生成的构造函数什么也不会干，若我们想初始化数据成员，非默认构造函数是必不可少的。
##### 析构函数
同构造函数类似，在对象被销毁的时候也会自动调用析构函数，同样的编译器也会为我们自动生成类的析构函数，但前提是我们不做定义。我们也可以自定义析构函数，使其按照我们的意愿执行。析构函数名就是在类名前面加上 `~`，它也没有返回值，参数有无均可。
```c++
class Student{
  public:
    ~Student(){}
};
```
我们通常将类的构造函数和析构函数都声明在 `public` 中，这是因为如果声明在 `private` 中我们就无法从类外部访问到这个函数，而这两个函数是创建类对象和销毁类对象的关键，这也使我们无法创建和销毁类对象，所以声明在 `public` 中是必要的。
另外，我们习惯上在构造函数中初始化数据成员，如初始化变量、为指针分配内存空间等操作，而在析构函数中释放我们分配的内存，一方面这样做可以方便我们创建对象时初始化数据，另一方面这样做也可以避免一些潜在的问题，如只声明了指针而忘记分配空间等。
```c++
class Student{
  private:
    int age_;
    string name_;
  public:
    Student(int age, string name) : age_(age), name_(name){}
    ~Student(){}
};
```
建议在编写类的时候就定义构造函数和析构函数，避免使用编译器自动生成的默认构造函数和默认析构函数，这样做有助于我们养成良好的编程习惯，同时避免一些潜在的问题。
#### `this` 指针
我们在编写类成员函数的时候可能会遇到一个问题：当我们想返回引用函数的对象本身时，我们该怎么做？C++为我们提供了一种方法-- `this` 指针。它**指向用来调用成员函数的对象**，同时 `this` 指针本身作为隐参自动传递给成员函数。
```c++
class Student{
  private:
    int age_;
  public:
    Student(int age) : age_(age){}
    const Student& Compare(const Student& other) const;
};

const Student& Student::Compare(const Student& other) const{
	if(other.age > age)
		return other;
	else
		return *this;
}

Student xiaoming(14);
Student xiaohong(15);
Student& t = xiaoming.Compare(xiaohong);
```
在上面的 `Compare` 成员函数中，`this` 指向的就是调用该函数的对象，示例中，`xiaoming` 这个对象调用了该成员函数，所以 `this` 指针就指向 `xiaoming` ，又由于 `Compare` 函数返回的是引用类型，所以我们要对 `this` 指针解引用。利用 `this` 指针的特性，我们也可以对 `Compare` 函数做如下修改（修改后的函数和原函数是等价的）：
```c++
const Student& Student::Compare(const Student& other) const{
	if(other.age > this->age)
		return other;
	else 
		return *this;
}
```
需要注意的是，当我们使用指针来访问类的数据成员时，不能使用 `.` 成员运算符，而应该使用 `->`。就像上修改后的函数一样，`this` 指针指向一个对象，就可以使用 `->` 来访问这个对象的数据成员。
### 继承与派生
C++为我们提供了一种方法，使我们不必重复编写一些重复类，增强了类的复用性，这就是继承。如果一个类的属性与另一个类有高度相似性，那么我们就可以从另一个类继承该类，使这个类拥有父类的属性。
继承代表了 `is a` 关系。例如，哺乳动物是动物，狗是哺乳动物，因此，狗是动物，等等。规定被继承的函数叫基类或父类，派生出的类叫派生类或子类。
```c++
// 基类
class Animal {
};

//派生类
class Dog : public Animal {
};
```
继承表明子类继承了父类的数据，C++对其中的继承方式也作了规定，有三种继承方式：`private`、`public` 和 `projected`。默认情况下，派生类可以访问基类的所有非私有成员函数。另外，**派生类不能直接访问基类的私有部分，而必须通过基类方法实现**。
我们几乎不使用 `protected` 或 `private` 继承，通常使用 `public` 继承。当使用不同类型的继承时，遵循以下几个规则：

-  `public`：当一个类派生自**公有**基类时，基类的**公有**成员也是派生类的**公有**成员，基类的**保护**成员也是派生类的**保护**成员，基类的**私有**成员不能直接被派生类访问，但是可以通过调用基类的**公有**和**保护**成员来访问。在不发生派生的情况下，`projected` 和 `private` 没有区别。
- `projected`：当一个类派生自**保护**基类时，基类的**公有**和**保护**成员将成为派生类的**保护**成员。
- `private`：当一个类派生自**私有**基类时，基类的**公有**和**保护**成员将成为派生类的**私有**成员。
## 重载
### 函数重载
现在我们写一个可以比较两个数大小的函数：
```c++
int Max(int x, int y){ return x > y ? x : y; }
```
经过实践我们很快就会发现这个函数的弊端，这个函数只能比较 `int` 类型的数，若我们想比较 `double` 类型该怎么办呢？C++提供了一种方法--函数重载，即我们可以声明同名不同参的函数，以满足程序的需要。值得注意的是，这在 C 语言中是不被允许的。
```c++
int Max(int x, int y){ return x > y ? x : y; }
double Max(double x, double y){ return x > y ? x : y; }
```
程序在运行的时候会根据参数类型自动推断出该调用哪个函数，拿上面两个例子来说，若我们向 `Max` 函数传入 `int` 类型的变量，程序就会自动调用第一个函数，反之我们传入 `double` 类型的变量，程序就会自动调用第二个函数。另外，函数重载在类成员函数中也是适用的。
### 运算符重载
除函数重载外 C++还为我们提供了运算符的重载，即运算符不再局限于标准类型（简单的四则运算等），我们可以依照我们的意愿自定义运算符的运算规则。
```c++
class Student {
 private:
  int age_;
 public:
  Student() {}
  Student(int age) : age_(age) {}
  bool operator>(const Student& other) {
    if (age_ > other.age_)
      return true;
    else
      return false;
  }
};

 Student xiaoming(14);
 Student xiaohong(15);
 if (xiaohong > xiaoming) cout << "yes" << endl; // yes
```
由于 `Student` 类型不是标准类型，编译器无法判断 `xiaoming` 和 `xiaohong` 的大小关系，想要比较二者大小关系就必须重载 `>` 运算符，上述示例展示了如何重载 `>` 运算符。重载后的运算符使用规则和普通的使用规律相同，只是其内运行逻辑是按照我们的意愿进行的。重载运算符的返回类型和该运算符的标准返回类型相同，就上述示例而言，在标准运算中 `>` 执行结果的 `bool` 类型，那么我们重载时也应返回一个 `bool` 类型，当然这只限于关系运算符。重载非关系运算符时的返回结果可以由我们自定义，但是通常我们会返回和传入参数类型相同的类型。
```c++
class Student {
 public:
  int age_;
  
  Student() {}
  Student(int age) : age_(age) {}
  Student operator+(const Student& other) {
    return Student(this->age_ + other.age_);
  }
};

 Student xiaoming(14);
 Student xiaohong(15);
 Student t = xiaohong + xiaoming;
 cout << t.age_; // 29
```
## 模板
模板的存在是为了减少我们代码的复用性以及增强泛用性。上面我们介绍了函数重载，你可能会想我想比较 `int`、`double` 两种类型的值就要编写两个函数，那我想比较更多类型是不是就必须写更多同名函数？这显然不符合我们的预期，C++为我们提供了一种方法，可以大大减少这种问题的出现，这就是我们即将展示的“模板”。
### 函数模板
模板的声明使用关键字 `template`，后面跟上模板类型，在紧接着就可以直接声明我们想编写的函数了。
```c++
template <typename T>
...function...
```
有了模板我们就可以重写上文 `Max` 函数了，使其变得更加简洁、泛用性更强。
```c++
template <typename T>
T Max(const T& x, const T& y) {
  return x > y ? x : y;
}

int a = 0;
int b = 2;
cout << Max(a, b) << endl; // 2
double c = 0;
double d = 2;
cout << Max(c, d) << endl; // 2
```
同重载函数的调用选择一样，模板类型 `T` 的推导也是根据传入函数的参数类型决定的。如上第一次调用 `Max` 函数，我们传入了 `int` 类型的变量，那么编译器就会自动把模板类型 `T` 推导为 `int`，这样的话，这个模板函数就和我们上文第一次编写的 `Max` 函数相同了。
模板类型关键字使用 `typename` 和 `class` 均可，习惯上，我们使用 `typename` 关键字，使用 `T` 作为模板类型的名字，当然你也可以使用其他名字。不仅于此，你还可以使用多个模板类型。
### 类模板
正如我们定义函数模板一样，我们也可以定义类模板。泛型类声明的一般形式如下所示：
```c++
template <typename type>
class class-name {
	......
}
```
下面展示一种泛型类的编写，它是数据结构中的栈：
```c++
template <class T>
class Stack {
 private:
  vector<T> elems;  // 元素
  
 public:
  void push(T const&);  // 入栈
  void pop();           // 出栈
  T top() const;        // 返回栈顶元素
  bool empty() const {  // 如果为空则返回真。
    return elems.empty();
  }
};
```
与模板函数不同的是，模板类不能自动推导类型，需要我们显式指定类型。以我们上面写的栈类为例，展示如何定义一个元素类型为 `int` 的栈。
```c++
Stack<int> stack;
```
模板给我们带来了极大的便利，它泛型编程提供了一种方式。实际上，STL 库就大量使用了模板，以增强泛用性。
## 异常
我们运行程序时可能会出现我们预料以外的情况，这些情况有时会导致严重的情况，甚至使正在运行的程序终止，而 C++为我们提供了一种异常处理方法以避免这种情况的发生。
### try catch
假如我们现在有一个函数 `div`，它可能会抛出异常（除数为 0），如果我们不加处理的执行程序，那么后果将是我们无法预料的。
```c++
int div(int num1, int num2){
return num1 / num2;
}
```
为了避免这种情况，我们需要在这个函数抛出异常的时候进行处理。C++为我们提供了 `try`、`catch` 关键字用于处理异常，`try` 里面用于放置可能会出现异常的语句，而 `catch` 用于捕获出现的异常，要想使用这种方式处理异常，我们必须告诉编译器将有异常发生，C++允许我们使用 `throw` 来抛出异常以供 `catch` 检测，那么我们就可以写出如下的代码：
```c++
int division(int num1, int num2) {
  if (num2 == 0) throw "The divisor cannot be zero!";
  return num1 / num2;
}

int x, y;
while (cin >> x >> y) {
  try {
    int ans = division(x, y);
    cout << ans << endl;
    return 0;
  } catch (const char* msg) {
    cout << msg << endl;
    continue;
  }
}
```
我们需要注意的是，当代码块抛出异常时就不再执行当前代码块以下的语句，而是跳转到与 `catch` 捕获类型相同的代码块向下执行。如上示例，当程序运行到 `div` 中的 `throw` 时就不会执行当前函数剩余的语句了，而跳转到 `catch` 语句所包含的代码块中，执行该代码块中的语句。
这里只是介绍了 C++抛出接收异常的最基本方式，更高级的运用还需读者自行探索。实际上我们通常通过继承 C++中 `exception` 基类来定义我们自己的异常类，进而实现自定义异常。当然 C++中也有很多内置的标准异常类，它们都继承自 `exception` 基类，如逻辑异常 `logic_error`、运行异常 `runtime_error` 等，这些都可以直接使用。