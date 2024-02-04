# 基础概念
## 并行和并发

- 并行：指两个或多个独立的操作**同时进行**。关键是有处理多个任务的能力，不一定要同时。
- 并发：指**一个时间段内**执行多个操作。关键是有**同时**处理多个任务的能力。
- 它们最关键的点就是：是否是『**同时**』。
## 互斥和同步
- 互斥：同一个资源同一时间只有一个访问者可以进行访问，其他访问者需要等前一个访问者访问结束才可以开始访问该资源。但互斥无法限制访问者对资源的访问顺序，即访问是无序的。
- 同步：分布在不同进程之间的若干程序片断，它们的运行必须严格按照规定的某种先后次序来运行，这种先后次序依赖于要完成的特定的任务。所以同步就是在互斥的基础上，通过其它机制实现访问者对资源的有序访问。
- 同步是一种更为复杂的互斥，而互斥是一种特殊的同步。
## 同步和异步
- 同步：同步就是顺序执行，执行完一个再执行下一个，需要等待、协调运行。
- 异步：异步和同步是相对的，异步就是彼此独立,在等待某事件的过程中继续做自己的事，不需要等待这一事件完成后再工作。
# 线程管理
## 线程的两种阻塞方法
- `detach` ：启动的线程自主在后台运行，当前的代码继续往下执行，不等待新线程结束。
- `join` ：等待启动的线程完成，才会继续往下执行。确保主线程在子线程退出之后才退出。
- 当线程启动后，一定要在和线程相关联的 `std:: thread` 对象销毁前，对线程运用 `join` 或者 `detach` 方法。
- **只要创建了线程对象**（前提是，实例化 `std:: thread` 对象时传递了“函数名/可调用对象”作为参数），线程就开始执行。所以不应该在创建了线程后马上 `join`, 这样会马上阻塞主线程，创建了线程和没有创建一样，应该在晚一点的位置调用 `join`
## 线程参数的传递
- **默认的会将传递的参数以拷贝的方式复制到线程空间，即使参数的类型是引用。** 若想传一个真正引用的参数，应调用 `std::ref`，例如：`thread t(func,std::ref(node))`。
- 使用类的成员函数作为线程函数：
```cpp
class _tagNode{

public:
	void do_some_work(int a);
};
_tagNode node;

thread t(&_tagNode::do_some_work, &node,20);
```

# 优化

资源获取即初始化 (RAII)：将资源的获取和释放与对象的生命周期绑定在一起。通过在对象的构造函数中获取资源，并在析构函数中释放资源，可以确保资源在对象不再使用时被正确释放，即使发生异常或提前退出作用域的情况下也是如此。

```cpp
class thread_guard
{
	thread &t;
public :
	explicit thread_guard(thread& _t) :
		t(_t){}

	~thread_guard()
	{
		if (t.joinable())
			t.join();
	}

	thread_guard(const thread_guard&) = delete;
	thread_guard& operator=(const thread_guard&) = delete;
};

void func(){

	thread t([]{
		cout << "Hello thread" <<endl ;
	});

	thread_guard g(t);
}
```
# 死锁
1. 互斥（资源同一时刻只能被一个进程使用）
2. 请求并保持（进程在请资源时，不释放自己已经占有的资源）
3. 不剥夺（进程已经获得的资源，在进程使用完前，不能强制剥夺）
4. 循环等待（进程间形成环状的资源循环等待关系）
# 互斥锁

- `std::mutex` 独占的互斥锁，不能递归使用
- `std::timed_mutex` 带超时的互斥锁，不能递归使用
- `std::recursive_mutex` 递归互斥锁，不带超时功能
- `std::recursive_timed_mutex` 带超时功能的递归互斥锁

## std::mutex

- `lock()` 用于给临界区加锁，并且只能有一个线程获得锁的所有权，它有阻塞线程的作用。
- `try_lock()` 和 `lock()` 区别在于前者不会阻塞线程，而后者会阻塞线程：
> 如果互斥锁是未锁定状态，得到了互斥锁所有权并加锁成功，函数返回 true
> 如果互斥锁是锁定状态，无法得到互斥锁所有权加锁失败，函数返回 false

## 线程同步

1. 找到多个线程操作的共享资源（全局变量、堆内存、类成员变量等），也可以称之为临界资源
2. 找到和共享资源有关的上下文代码，也就是临界区（下图中的黄色代码部分）
3. 在临界区的上边调用互斥锁类的 `lock ()` 方法
4. 在临界区的下边调用互斥锁的 `unlock ()` 方法

![[Pasted image 20230712220329.png]]

示例：
```cpp
#include<iostream>
#include<exception>
#include<mutex>
#include<thread>
#include<chrono>

std::mutex g_mutex;
int g = 0;
void func(void)
{
    for(int i= 0 ; i < 3; i++)
    {
        g_mutex.lock();
        g++;
        std::cout << "ID" << std::this_thread::get_id() << ":" << g << std::endl;
        g_mutex.unlock();
        std::this_thread::sleep_for(std::chrono::milliseconds(1));
    }
}

int main(int agrc, char** argv)
{
    std::thread t1{func};//防止编译器误识别为函数
    std::thread t2{func};
    t1.join();
    t2.join();

    return 0;
}
```
输出：
```cpp
ID139945513928448:1
ID139945505535744:2
ID139945513928448:3
ID139945505535744:4
ID139945505535744:5
ID139945513928448:6
```

## std::lock_guard

`lock_guard` 是 C++11 新增的一个模板类，使用这个类，可以简化互斥锁 `lock()` 和 `unlock()` 的写法，同时也更安全。`lock_guard` 在使用上面提供的这个构造函数构造对象时，会自动锁定互斥量，而在退出作用域后进行析构时就会自动解锁，从而保证了互斥量的正确操作，避免忘记 `unlock ()` 操作而导致线程死锁。`lock_guard` 使用了 ***RAII*** 技术，就是在类构造函数中分配资源，在析构函数中释放资源，保证资源出了作用域就释放。但是这种方式也有弊端，由于临界区变得很大而导致效率变得很低，多个线程是线性的执行临界区代码的，因此临界区越大程序效率越低

```cpp
// 类的定义，定义于头文件 <mutex>
template< class Mutex >
class lock_guard;

// 常用构造函数
explicit lock_guard ( mutex_type& m );

#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
	//用此语句替换了m.lock()；lock_guard传入一个参数时，该参数为互斥量，此时调用了lock_guard的构造函数，申请锁定m
    lock_guard<mutex> g1(m);
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
}//此时不需要写m.unlock(),g1出了作用域被释放，自动调用析构函数，于是m被解锁

void proc2(int a)
{
	//通过使用{}来调整作用域范围，可使得m在合适的地方被解锁
    {
        lock_guard<mutex> g2(m);
        cout << "proc2函数正在改写a" << endl;
        cout << "原始a为" << a << endl;
        cout << "现在a为" << a + 1 << endl;
    }
    cout << "作用域外的内容3" << endl;
    cout << "作用域外的内容4" << endl;
    cout << "作用域外的内容5" << endl;
}
int main()
{
    int a = 0;
    thread t1(proc1, a);
    thread t2(proc2, a);
    t1.join();
    t2.join();
    return 0;
}
```
也可以传入两个参数，第一个参数为 `adopt_lock` 标识时，表示构造函数中不再进行互斥量锁定，因此**此时需要提前手动锁定**。
```cpp
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;//实例化m对象，不要理解为定义变量
void proc1(int a)
{
    m.lock();//手动锁定
    lock_guard<mutex> g1(m,adopt_lock);
    cout << "proc1函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 2 << endl;
}//自动解锁

void proc2(int a)
{
    lock_guard<mutex> g2(m);//自动锁定
    cout << "proc2函数正在改写a" << endl;
    cout << "原始a为" << a << endl;
    cout << "现在a为" << a + 1 << endl;
}//自动解锁
int main()
{
    int a = 0;
    thread t1(proc1, a);
    thread t2(proc2, a);
    t1.join();
    t2.join();
    return 0;
}
```
## unique_lock
`std::unique_lock` 类似于 `lock_guard`,只是 `std:: unique_lock` 用法更加丰富，同时支持 `std:: lock_guard` 的原有功能。使用 `std:: lock_guard` 后不能手动 `lock()` 与手动 `unlock()`;使用 `std:: unique_lock` 后可以手动 `lock()` 与手动 `unlock()`; `std:: unique_lock` 的第二个参数，除了可以是 `adopt_lock`,还可以是 `try_to_lock` 与 `defer_lock`.

`try_to_lock`: 尝试去锁定，**得保证锁处于 `unlock` 的状态**,然后尝试现在能不能获得锁；尝试用 `mutx` 的 `lock()` 去锁定这个 `mutex`，但如果没有锁定成功，会立即返回，**不会阻塞**在那里，并继续往下执行；

`defer_lock`: 始化了一个没有加锁的 `mutex`;
```cpp
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;
mutex m;
void proc1(int a)
{
	unique_lock<mutex> g1(m, defer_lock);//始化了一个没有加锁的mutex
	cout << "xxxxxxxx" << endl;
	g1.lock();//手动加锁，注意，不是m.lock();注意，不是m.lock(),m已经被g1接管了;
	cout << "proc1函数正在改写a" << endl;
	cout << "原始a为" << a << endl;
	cout << "现在a为" << a + 2 << endl;
	g1.unlock();//临时解锁
	cout << "xxxxx" << endl;
	g1.lock();
	cout << "xxxxxx" << endl;
}//自动解锁

void proc2(int a)
{
	unique_lock<mutex> g2(m, try_to_lock);//尝试加锁一次，但如果没有锁定成功，会立即返回，不会阻塞在那里，且不会再次尝试锁操作。
	if (g2.owns_lock()) {//锁成功
		cout << "proc2函数正在改写a" << endl;
		cout << "原始a为" << a << endl;
		cout << "现在a为" << a + 1 << endl;
	}
	else {//锁失败则执行这段语句
		cout << "" << endl;
	}
}//自动解锁

int main()
{
	int a = 0;
	thread t1(proc1, a);
	t1.join();
	//thread t2(proc2, a);
	//t2.join();
	return 0;
}
```
## std::recursive_mutex
与 `std::mutex` 相比，`std::recursive_mutex` 允许同一个线程多次获取该锁，而不会导致死锁。

递归互斥锁的主要特点是允许同一个线程多次对它进行加锁，而不会因为线程自身的重复加锁而造成死锁。每次成功加锁后，锁的计数会增加，只有当每次加锁操作都对应着相同数量的解锁操作时，锁才会真正释放。

```cpp
#include <iostream>
#include <thread>
#include <mutex>

Std:: recursive_mutex mtx;  // 递归互斥锁

Void recursiveFunction (int depth) {
    std::lock_guard<std::recursive_mutex> lock (mtx);  // 使用 lock_guard 进行自动加锁和解锁

    If (depth > 0) {
        Std:: cout << "Depth: " << depth << std:: endl;
        RecursiveFunction (depth - 1);
    }
}

Int main () {
    Std:: thread t 1 (recursiveFunction, 3);
    Std:: thread t 2 (recursiveFunction, 2);

    T 1.Join ();
    T 2.Join ();

    Return 0;
}
```
## std::timed_mutex
超时独占互斥锁，主要是在获取互斥锁资源时增加了超时等待功能，因为不知道获取锁资源需要等待多长时间，为了保证不一直等待下去，设置了一个超时时长，超时后线程就可以解除阻塞去做其他事情了。
```cpp
Void lock ();
Bool try_lock ();
Void unlock ();

// std:: timed_mutex 比 std:: _mutex 多出的两个成员函数
template <class Rep, class Period>
  bool try_lock_for (const chrono::duration<Rep,Period>& rel_time);

template <class Clock, class Duration>
  bool try_lock_until (const chrono::time_point<Clock,Duration>& abs_time);
```
示例：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

timed_mutex g_mutex;

void work()
{
    chrono::seconds timeout(1);
    while (true)
    {
        // 通过阻塞一定的时长来争取得到互斥锁所有权
        if (g_mutex.try_lock_for(timeout))
        {
            cout << "当前线程ID: " << this_thread::get_id() 
                << ", 得到互斥锁所有权..." << endl;
            // 模拟处理任务用了一定的时长
            this_thread::sleep_for(chrono::seconds(10));
            // 互斥锁解锁
            g_mutex.unlock();
            break;
        }
        else
        {
            cout << "当前线程ID: " << this_thread::get_id() 
                << ", 没有得到互斥锁所有权..." << endl;
            // 模拟处理其他任务用了一定的时长
            this_thread::sleep_for(chrono::milliseconds(50));
        }
    }
}

int main()
{
    thread t1(work);
    thread t2(work);

    t1.join();
    t2.join();

    return 0;
}
```
# condition_variable
`#include<condition_variable>`

如何使用？`std:: condition_variable` 类搭配 `std:: mutex` 类来使用，`std:: condition_variable` 对象(`std:: condition_variable cond;`)的作用不是用来管理互斥量的，它的作用是用来**同步线程**，它的用法相当于编程中常见的 `flag` 标志（A、B 两个人约定 `flag=true` 为行动号角，默认 `flag` 为 `false`,A 不断的检查 `flag` 的值,只要 B 将 `flag` 修改为 `true`，A 就开始行动）。类比到 `std:: condition_variable`，A、B 两个人约定 `notify_one` 为行动号角，A 就等着（调用 `wait()`,阻塞）,只要 B 一调用 `notify_one`，A 就开始行动（不再阻塞）。

`wait` 函数需要传入一个 `std::mutex`（一般会传入 `std:: unique_lock` 对象）,即上述的 `locker`。`wait` 函数会自动调用 `locker.unlock()` 释放锁（因为需要释放锁，所以要传入 `mutex`）并阻塞当前线程，本线程释放锁使得其他的线程得以继续竞争锁。一旦当前线程获得 `notify` (通常是另外某个线程调用 `notify` 唤醒了当前线程)，`wait()` 函数此时再自动调用 `locker.lock()` 上锁。

- `cond.notify_one()`: 随机唤醒一个等待的线程
- `cond.notify_all()`: 唤醒所有等待的线程
# 异步线程
`#include<future>`
## async 与 future
`std::async` 是一个函数模板，用来启动一个异步任务，它返回一个 `std::future` 类模板对象，`future` 对象起到了**占位**的作用（记住这点就可以了），占位是什么意思？就是说该变量现在无值，但将来会有值。刚实例化的 `future` 是没有储存值的，但在调用 `std::future` 对象的 `get()` 成员函数时，主线程会被阻塞直到异步线程执行结束，并把返回结果传递给 `std::future`，即通过 `FutureObject.get()` 获取函数返回值。
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include<future>
#include<Windows.h>
using namespace std;
double t1(const double a, const double b)
{
 double c = a + b;
 Sleep(3000);//假设t1函数是个复杂的计算过程，需要消耗3秒
 return c;
}

int main() 
{
 double a = 2.3;
 double b = 6.7;
 future<double> fu = async(t1, a, b);//创建异步线程线程，并将线程的执行结果用fu占位；
 cout << "正在进行计算" << endl;
 cout << "计算结果马上就准备好，请您耐心等待" << endl;
 cout << "计算结果：" << fu.get() << endl;//阻塞主线程，直至异步线程return
        //cout << "计算结果：" << fu.get() << endl;//取消该语句注释后运行会报错，因为future对象的get()方法只能调用一次。
 return 0;
}
```
## shared_future
`std:: future` 与 `std:: shard_future` 的用途都是为了**占位**，但是两者有些许差别。`std:: future` 的 `get()` 成员函数是转移数据所有权; `std:: shared_future` 的 `get()` 成员函数是复制数据。因此： **`future` 对象的 `get()` 只能调用一次**；无法实现多个线程等待同一个异步线程，一旦其中一个线程获取了异步线程的返回值，其他线程就无法再次获取。`std:: shared_future` 对象的 `get()` 可以调用多次；可以实现多个线程等待同一个异步线程，每个线程都可以获取异步线程的返回值。
# Linux
在 Linux 上运行是必须链接上对应的 pthread 库
```cmake
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} "-pthread")
```
# 生产者-消费者模式
生产者用于生产数据，生产一个就往共享数据区存一个，如果共享数据区已满的话，生产者就暂停生产，等待消费者的通知后再启动。消费者用于消费数据，一个一个的从共享数据区取，如果共享数据区为空的话，消费者就暂停取数据，等待生产者的通知后再启动。
```cpp
#include <chrono>
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <queue>
#include <thread>

using namespace std::chrono_literals;

const int MAX_LEGENTH = 5;

std::queue<double> queue_buffer;
std::mutex mu;
std::condition_variable producer_con;
std::condition_variable consumer_con;

void Consumer()
{
  std::unique_lock<std::mutex> consumer_lock(mu);

  if (queue_buffer.empty()) {
    std::cout << "queue is empty, consumer is waiting for producer."
              << std::endl;
    producer_con.wait(consumer_lock);
  }

  while (!queue_buffer.empty()) {
    std::cout << "consuming ..." << std::endl;
    std::this_thread::sleep_for(3s);
    queue_buffer.pop();
  }
  consumer_con.notify_one();
}
void Producer()
{
  std::unique_lock<std::mutex> producer_lock(mu);
  if (queue_buffer.size() == MAX_LEGENTH) {
    std::cout << "queue is full, producer is waiting for consumer."
              << std::endl;
    producer_con.wait(producer_lock);
  }

  while (queue_buffer.size() < MAX_LEGENTH) {
    std::cout << "producing ..." << std::endl;
    std::this_thread::sleep_for(3s);
    queue_buffer.push(6);
  }
  producer_con.notify_one();
}

int main()
{
  std::thread producer_t(Producer);
  std::thread consumer_t(Consumer);

  producer_t.join();
  consumer_t.join();
  return 0;
}
```
# 线程池
ThreadPool.hpp
```cpp
#pragma once
#ifndef THREAD_POOL_H
#define THREAD_POOL_H

#include <condition_variable>
#include <functional>
#include <future>
#include <memory>
#include <mutex>
#include <queue>
#include <stdexcept>
#include <thread>
#include <vector>

class ThreadPool
{
public:
  ThreadPool(size_t);
  ~ThreadPool();

  template <class F, class... Args>
  auto enqueue(F && f, Args &&... args) -> std::future<decltype(f(args...))>;

private:
  // need to keep track of threads so we can join them
  std::vector<std::thread> workers;
  // the task queue
  std::queue<std::function<void()> > tasks;

  // synchronization
  std::mutex queue_mutex;
  std::condition_variable condition;
  bool stop;
};

// the constructor just launches some amount of workers
inline ThreadPool::ThreadPool(size_t threads) : stop(false)
{
  for (size_t i = 0; i < threads; ++i)
    workers.emplace_back([this] {
      for (;;) {
        std::function<void()> task;

        {
          std::unique_lock<std::mutex> lock(this->queue_mutex);
          this->condition.wait(
            lock, [this] { return this->stop || !this->tasks.empty(); });
          if (this->stop && this->tasks.empty()) return;
          task = std::move(this->tasks.front());
          this->tasks.pop();
        }

        task();
      }
    });
}

// add new work item to the pool
template <class F, class... Args>
auto ThreadPool::enqueue(F && f, Args &&... args)
  -> std::future<decltype(f(args...))>
{
  using return_type = decltype(f(args...));

  auto task = std::make_shared<std::packaged_task<return_type()> >(
    std::bind(std::forward<F>(f), std::forward<Args>(args)...));

  std::future<return_type> res = task->get_future();
  {
    std::unique_lock<std::mutex> lock(queue_mutex);

    // don't allow enqueueing after stopping the pool
    if (stop) throw std::runtime_error("enqueue on stopped ThreadPool");

    tasks.emplace([task]() { (*task)(); });
  }
  condition.notify_one();
  return res;
}

// the destructor joins all threads
inline ThreadPool::~ThreadPool()
{
  {
    std::unique_lock<std::mutex> lock(queue_mutex);
    stop = true;
  }
  condition.notify_all();
  for (std::thread & worker : workers) worker.join();
}

#endif
```
使用示例：(example.cpp)
```cpp
#include <chrono>
#include <iostream>
#include <vector>

#include "ThreadPool.hpp"

int main()
{
  ThreadPool pool(4);
  std::vector<std::future<int> > results;

  for (int i = 0; i < 8; ++i) {
    results.emplace_back(pool.enqueue([i] {
      std::cout << "hello " << i << std::endl;
      std::this_thread::sleep_for(std::chrono::seconds(1));
      std::cout << "world " << i << std::endl;
      return i * i;
    }));
  }

  for (auto && result : results) std::cout << result.get() << ' ';
  std::cout << std::endl;

  return 0;
}
```
# 参考资料

- [苏丙榅](https://subingwen.cn/cpp/mutex/)
- https://zhuanlan.zhihu.com/p/194198073