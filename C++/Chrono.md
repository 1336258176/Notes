- 主要用于处理日期和时间
- 主要包含三种类型的类：**时间间隔 duration**、**时钟 clocks**、**时间点 time point**

# 时间间隔 duration

## 原型

```cpp
template<
    class Rep,
    class Period = std::ratio<1>
> class duration;
```
- `Rep`：这是一个数值类型，表示时钟数（周期）的类型（默认为整形）。若 Rep 是浮点数，则 duration 能使用小数描述时钟周期的数目。
- `Period`：表示时钟的周期，它的原型如下：
```cpp
template<
	std::intmax_t Num,
	std::intmax_t Denom = 1
> class ratio;
```
ratio 类表示每个时钟周期的秒数，其中第一个模板参数 Num 代表分子，Denom 代表分母，该分母值默认为 1，因此，ratio 代表的是一个分子除以分母的数值，比如：ratio<2> 代表一个时钟周期是 2 秒

为了方便使用，在标准库中定义了一些常用的时间间隔，比如：时、分、秒、毫秒、微秒、纳秒，它们都位于 chrono 命名空间下，定义如下：

| 名称 | 类型                        |
| :-: | :-: |
| 纳秒 | `std::chrono::nanoseconds`  |
| 微秒 | `std::chrono::microseconds` |
| 毫秒 | `std::chrono::milliseconds` |
| 秒   | `std::chrono::seconds`      |
| 分钟 | `std::chrono::minutes`      |
| 小时 | `std::chrono::hours`        |

## 成员函数

duration 类还提供了获取时间间隔的时钟周期数的方法 count ()，函数原型如下：
```cpp
constexpr rep count() const;
```

## 示例

```cpp
#include<iostream>
#include<chrono>

int main(int agrc, char** argv)
{
    int a = 10;
    int b = 90;

    std::chrono::time_point<std::chrono::steady_clock> start_time;
    int c = a * b;
    auto time = static_cast<std::chrono::duration<double,std::milli>>(std::chrono::steady_clock::now() - start_time).count();
    std::cout << time << std::endl;

    return 0;
}
```

# 时间点 time point

它被实现成如同存储一个 Duration 类型的自 Clock 的纪元起始开始的时间间隔的值，通过这个类最终可以得到时间中的某一个时间点。

## 原型

```cpp
template<
    class Clock,
    class Duration = typename Clock::duration
> class time_point;
```

- Clock ：此时间点在此时钟上的计量
- Duration：用于计量从纪元起时间的 `std::chrono::duration` 类型

## 成员函数

- `time_since_epoch()` ：用来获得 1970 年 1 月 1 日到 time_point 对象中记录的时间经过的时间间隔（duration）
```cpp
duration time_since_epoch() const:
```

## 示例

此程序用于记录 CPP 乘法的操作时间

```cpp
#include<iostream>
#include<chrono>

int main(int agrc, char** argv)
{
    int a = 10;
    int b = 90;

    std::chrono::time_point<std::chrono::steady_clock> start_time;
    int c = a * b;
    auto time = static_cast<std::chrono::duration<double,std::milli>>(std::chrono::steady_clock::now() - start_time).count();
    std::cout << time << std::endl;

    return 0;
}
```

# 时钟 clock

`Chrono` 库中提供了获取当前的系统时间的时钟类，包含的时钟一共有三种：

- `System_clock`：系统的时钟，系统的时钟可以修改，甚至可以网络对时，因此使用系统时间计算时间差可能不准。
- `Steady_clock`：是固定的时钟，相当于秒表。开始计时后，时间只会增长并且不能修改，**适合用于记录程序耗时**。
- `High_resolution_clock`：`high_resolution_clock` 在不同标准库实现之间实现并不一致，尽量不要使用。通常它只是 `std::chrono::steady_clock` 或 `std::chrono::system_clock` 的别名。

# 转换函数

## duration_cast

原型：（返回整数时长）
```cpp
template <class ToDuration, class Rep, class Period>
  constexpr ToDuration duration_cast (const duration<Rep,Period>& dtn);
```

示例：
```cpp
#include <iostream>
#include <chrono>
using namespace std;
using namespace std::chrono;

void f()
{
    cout << "print 1000 stars ...." << endl;
    for (int i = 0; i < 1000; ++i)
    {
        cout << "*";
    }
    cout << endl;
}

int main()
{
    auto t1 = steady_clock::now();
    f();
    auto t2 = steady_clock::now();

    // 整数时长：时钟周期纳秒转毫秒，要求 duration_cast
    auto int_ms = duration_cast<chrono::milliseconds>(t2 - t1);

    // 小数时长：不要求 duration_cast
    duration<double, ratio<1, 1000>> fp_ms = t2 - t1;

    cout << "f() took " << fp_ms.count() << " ms, "
        << "or " << int_ms.count() << " whole milliseconds\n";
}
```

## time_point_cast

原型：
```cpp
template <class ToDuration, class Clock, class Duration>
time_point<Clock, ToDuration> time_point_cast(const time_point<Clock, Duration> &t);
```

示例：
```cpp
#include <chrono>
#include <iostream>
using namespace std;

using Clock = chrono::high_resolution_clock;
using Ms = chrono::milliseconds;
using Sec = chrono::seconds;
template<class Duration>
using TimePoint = chrono::time_point<Clock, Duration>;

void print_ms(const TimePoint<Ms>& time_point)
{
    std::cout << time_point.time_since_epoch().count() << " ms\n";
}

int main()
{
    TimePoint<Sec> time_point_sec(Sec(6));
    // 无精度损失, 可以进行隐式类型转换
    TimePoint<Ms> time_point_ms(time_point_sec);
    print_ms(time_point_ms);    // 6000 ms

    time_point_ms = TimePoint<Ms>(Ms(6789));
    // error，会损失精度，不允许进行隐式的类型转换
    TimePoint<Sec> sec(time_point_ms);

    // 显示类型转换,会损失精度。6789 truncated to 6000
    time_point_sec = std::chrono::time_point_cast<Sec>(time_point_ms);
    print_ms(time_point_sec); // 6000 ms
}
```

# 参考资料

- [处理日期和时间的 chrono 库](https://subingwen.cn/cpp/chrono/)