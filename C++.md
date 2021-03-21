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
