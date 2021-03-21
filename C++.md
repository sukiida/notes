# 并发API
## 优选基于任务而非基于线程的子程序设计
```c++
auto fut = std::async(doAsyncWork);
```
