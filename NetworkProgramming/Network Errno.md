| 错误码          | 含义                   | 常见处理方式                |
| ------------ | -------------------- | --------------------- |
| EAGAIN       | 资源暂时不可用，操作现在无法完成     | 等待下一次 I/O 就绪通知        |
| EWOULDBLOCK  | 当前操作会阻塞              | Linux 下通常等同于 `EAGAIN` |
| EINTR        | 系统调用被信号中断            | 视具体调用情况重试             |
| EPIPE        | 管道破裂，写入端已无有效接收方      | 停止写入，关闭或清理连接          |
| ECONNRESET   | 连接被对端重置，通常收到 TCP RST | 关闭并清理连接               |
| ECONNREFUSED | 连接被拒绝                | 检查目标 IP、端口、服务状态       |
| ETIMEDOUT    | 连接或网络操作超时            | 超时处理、按需重试             |
| ENOTCONN     | Socket 尚未建立连接        | 检查连接状态                |
| EINPROGRESS  | 非阻塞连接正在建立            | 等待可写事件，再检查 `SO_ERROR` |
| EALREADY     | 前一次非阻塞连接尝试尚未完成       | 等待连接完成                |
| EISCONN      | Socket 已连接           | 避免重复连接                |
| ENETUNREACH  | 网络不可达                | 检查网络及路由               |
| EHOSTUNREACH | 目标主机不可达              | 检查路由、主机和网络状态          |

# 四个常见的errno

### 1. EAGAIN：暂时无法执行

常见于非阻塞 Socket。

例如：

```
ssize_t n = recv(fd, buf, sizeof(buf), 0);
```

对于非阻塞 Socket：

- 接收缓冲区有数据：返回实际读取字节数。
    
- 对端正常关闭写方向，且缓冲区数据已读完：返回 `0`。
    
- 当前没有数据可读：返回 `-1`，`errno = EAGAIN`。
    
- 其他错误：返回 `-1`，设置对应 `errno`。
    

关键：EAGAIN 不意味着连接断开。

对于 epoll ET（边缘触发）模式，通常需要持续读取，直到遇到 `EAGAIN`。

对于写操作同样适用：`send()` 返回 `EAGAIN` 时，应保留尚未发送的数据，并在合适时机监听 `EPOLLOUT`。

## 2. EWOULDBLOCK 一般同EAGAIN

通常, 在Linux下二者等效. 
```
EAGAIN == EWOULDBLOCK  // true
```

但是POIEX不保证其等效. 

**跨平台写法**

```
if (errno == EAGAIN || errno == EWOULDBLOCK) {
    // 非阻塞 I/O 暂时无法完成
}
```

这是较通用的写法，适用于 Linux、FreeBSD、macOS 等 POSIX 系统。

### 3. EINTR：系统调用被信号中断

例如，线程正在执行：

```
int n = epoll_wait(epfd, events, 1024, -1);
```

如果此时收到一个被捕获的信号，系统调用可能提前返回：

```
n == -1
errno == EINTR
```

关键：EINTR 通常表示操作被信号打断，不代表文件描述符或连接损坏。

注意，某些系统调用可能被内核自动重启（与 `SA_RESTART` 等机制有关），而 `epoll_wait()` 在 Linux 上不会被 `SA_RESTART` 自动重启。

### 4. EPIPE：接收方已经关闭

常见于：

- 管道读端全部关闭，但进程仍向写端写入。
    
- TCP 对端已经关闭接收方向，本端仍尝试发送数据。
    

例如：

```
send(fd, buf, len, 0);
```

可能返回：

```
-1
errno == EPIPE
```

关键：EPIPE 还涉及 SIGPIPE 信号。

默认情况下，触发 `EPIPE` 的写入操作可能同时产生 `SIGPIPE`，其默认动作是终止进程。

Linux Socket 编程中可以使用：

```
send(fd, buf, len, MSG_NOSIGNAL);
```

这样可以阻止这次 `send()` 触发 `SIGPIPE`，再通过 `errno` 处理错误。

也可以全局忽略 `SIGPIPE`：

```
signal(SIGPIPE, SIG_IGN);
```

在服务器开发中，通常更推荐针对 Socket 发送使用 `MSG_NOSIGNAL`，避免影响进程内其他模块的信号处理行为。