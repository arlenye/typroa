ReactiveProgramming



Reactive是异步非阻塞编程 是错误的

​    Reactive 正确的是 同步+异步非阻塞编程

 NIO是同步非阻塞  ，异步非阻塞是AIO



非阻塞从实现上来说，其实是回调

就是当前不阻塞，事后回调

```sequence
load() -> loadConfiguration():
loadConfigurations() -> loadUsers():
loadUsers()  -> loadOrder():

```



CompletableFuture

提供异步操作

提供`Future` 链式操作