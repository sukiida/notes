# C++标准库

## 12.1.1动态内存与智能指针

程序使用动态内存出于以下三种原因之一：

- 程序不知道自己需要使用多少对象
- 程序不知道自己所需对象的准确类型
- 程序需要在多个对象间共享数据

## 12.1.2直接管理内存（new/delete）

对于类中那些依赖编译器合成的默认构造函数的内置类型成员，如果他们未在类内被初始化，那么它们的值是未定义的。对动态分配的对象进行初始化通常是个好主意。

阻止new抛出异常

```c++
int *p2 = new (nothrow) int; // 如果分配失败，new返回一个空指针
```

delete

```c++
int i, *pi1 = &i, *pi2 = nullptr;
double *pd = new double(13), *pd2 = pd;
delete i;    // 错误，i不是一个指针
delete pi1;  // 未定义，pi1指向局部变量
delete pd;   // 正确
delete pd2;  // 未定义，pd2指向的内存已经被释放
delete pi2;  // 正确，释放一个空指针总是没有错误的
```

delete之将nullptr赋予指针，清楚地指出指针不指向任何对象。**这只能提供有限的保护**，多个指针指向相同内存的情况，nullptr只对其中被赋值的指针起保护作用。

## 12.1.3 shared_ptr和new结合使用

**不要混合使用智能指针和普通指针**

使用make_shared代替new，分配对象的同时将shared_ptr与之绑定，避免无意中将同一块内存绑定到多个独立创建的shared_ptr上。

# 并发API

## std::thread
```c++
void foo() {}
std::thread first(foo);
```
等待线程结束
```c++
first.join();
```
not joinalbe状态，下面三种情况：
1. 默认构造的std::thread对象，没有传入执行函数。
2. 已经运行t.join()或者t.detach()的线程对象。
3. 线程对象已经被移动（通过复制或者构造操作），意味着底层线程的控制权已经被转移给其他的std::thread对象。

不允许处于joinable状态的线程对象析构，此时触发异常。因为隐式join和隐式detach会造成更加不符合直觉的现象。
### 使std::thread对象在所有路径皆不可联接
## std::async

### 优选基于任务而非基于线程的子程序设计

```c++
auto fut = std::async(doAsyncWork);
```
### 如果异步是必要的，则指定std::launche::async
```c++
auto fut1 = std::async(f);
auto fut2 = std::async(std::launch::async, f);
```
默认策略，下面的异步调用可能会成为死循环。
```c++
using namespace std::literals;

void f()
{
    std::this_thread::sleep_for(1s);
}

auto fut = std::async(f);
while (fut.wait_for(100ms) != std::future_status::ready) {
    ...
}
```
## 协程
