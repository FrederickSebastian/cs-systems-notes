# Reactor 并发模型与实现概述

本文比较通知 worker 接收连接、主进程分发连接、共享队列配合 eventfd，以及 SO_REUSEPORT 端口复用四种方案。

## 1. Reactor 的基本分工

Reactor 通过 epoll 等机制等待 I/O 就绪，然后调用对应回调。监听 socket 就绪时执行 accept，连接 socket 就绪时执行非阻塞读写。

需要区分：

- `listenfd`：监听 socket，用于接收新连接。
- `connfd`：accept 返回的已连接 socket，用于与某个客户端通信。
- 一个 Reactor 可以交替处理很多连接；多个 worker 在不同 CPU 核上运行，才进一步实现并行。

## 2. 多进程：主进程通知，worker 执行 accept

```text
主进程监听 listenfd
    → 选择 worker，向管道写入通知
    → worker 的 Reactor 检测到管道可读
    → worker 读取通知并执行 accept
    → 将 connfd 加入自己的 Reactor
```

监听 socket 在 fork 前创建，子进程继承它。管道传递的是“去接收连接”的通知，不是连接 fd。

实现要点：

- listenfd 必须非阻塞；worker 可以循环 accept，直到 EAGAIN。
- 通知不会为 worker 预留连接。多个 worker 可能竞争同一待接收队列；有人得到 EAGAIN 是正常情况，不代表同一连接被重复接收。
- accept 不必为了防止重复接收而加锁；锁可以用于协调接收机会、减少竞争。
- LT 下，主进程不消费监听队列，可能反复发送通知，需要合并或协调。
- ET 下，需要保证通知可靠、worker 能处理剩余连接，不能简单地每次通知只 accept 一次。
- 轮询通知只是在分配接收机会，不保证最终连接数量均匀；一个 worker 可能一次接收很多连接。
- 每个 worker 通常在 fork 后创建自己的 epoll 实例。

优点是无须跨进程移交已连接 fd；难点是通知、监听触发与 accept 调度之间的协调。

## 3. 多进程/多线程：主进程 accept，再分发连接

```text
主进程监听 listenfd 并执行 accept
    → 选择 worker
    → 通过 Unix 域 socket 传递连接 fd
    → worker 接收 fd，加入自己的 Reactor
```

### 不能通过普通管道直接发送 fd 整数

fd 是每个进程自己的文件描述符表索引。主进程 accept 得到的 fd 不会自动出现在已经 fork 的子进程中。

```text
主进程 fd 12 → 客户端连接
子进程 fd 12 → 其他文件，或者无效
```

因此，`write(pipefd, &connfd, sizeof(connfd))` 只传递了数字，没有移交连接。

通常使用：

```text
socketpair(AF_UNIX, ...)
    + sendmsg / recvmsg
    + SCM_RIGHTS
```

内核在 worker 的 fd 表中建立对同一打开文件描述的引用；两端的 fd 数字不必相同。

**可以使用多线程实现该模型以避免文件描述符表不一致的问题.**

实现要点：

- 主进程 accept 后按连接数或负载选择 worker。
- worker 不需要 accept，通常可关闭继承的 listenfd。
- 成功发送后，主进程关闭自己的 connfd 副本；worker 最终关闭接收到的副本。
- 发送遇到 EAGAIN 时暂存连接并安排重试，同时限制待分发队列大小。
- 接收端检查附属消息、截断情况和 fd 数量。
- worker 本地创建 Connection 对象，不能把主进程对象的指针直接传过去使用。

优点是连接归属明确、可以逐连接调度；代价是 fd 传递、背压和关闭责任的管理。

## 4. 多线程：共享队列 + eventfd

这是同一进程内的主从 Reactor。线程共享 fd 表，因此可以通过共享队列直接交接 connfd 整数。

```text
主线程 accept 得到 connfd
    → 选择 worker
    → 加锁入队，解锁
    → 向 worker 的 eventfd 写入 1
    → worker 的 epoll_wait 被唤醒
    → 读取 eventfd，取出待办连接
    → 在自己的 Reactor 中注册连接
```

职责划分：

- 队列保存待处理的连接 fd。
- Mutex 保护队列的并发访问。
- eventfd 只负责通知，不保存连接 fd。
- worker 的 Reactor 负责后续连接读写和关闭。

实现要点：

1. 通常每个 worker 配置一个队列、一个 eventfd 和一个 Reactor，以支持定向分配。
2. 先入队，再通知，避免 worker 先醒来却看不到任务。
3. 在锁内将待办队列 swap 到局部队列，解锁后再注册连接，缩短持锁时间。
4. 回调通常先消费 eventfd 通知，再提取待办队列；以队列内容为准，不机械地按通知计数弹出任务。
5. eventfd 通常使用 `EFD_NONBLOCK | EFD_CLOEXEC`，正确处理 EINTR、EAGAIN 等返回情况。
6. 连接交接后，由所属 worker 管理其状态，避免多个线程同时操作同一连接。

eventfd 只有一个可读写 fd，不像普通管道分为读端和写端。

如果所有 worker 共用一个任务队列，被唤醒的 worker 可能取走全部任务，便不再是主线程精确指定连接归属的方式。

### eventfd的普通模式与信号量模式

创建时是否指定 `EFD_SEMAPHORE`，决定读取行为。

| 操作 | 普通模式 | 信号量模式 |
|---|---|---|
| 写入数值 n | 计数器加 n | 计数器加 n |
| 计数器大于 0 时读取 | 返回累计值并清零 | 返回 1，计数器减 1 |
| 计数器为 0 时读取 | 阻塞；非阻塞时返回 EAGAIN | 相同 |
| 典型用途 | 合并通知、批量处理任务 | 每次领取一个许可 |

两种模式的读写数据都使用 8 字节的 `uint64_t`。普通模式适合唤醒 worker 后批量处理队列；信号量模式不保证多个线程公平地各读取一次，同一线程可能连续领取许可。

## 6. SO_REUSEPORT端口复用：内核分配新连接

每个 worker 独立创建监听 socket，设置 SO_REUSEPORT，然后绑定相同的 IP 和端口。每个 worker 还拥有自己的 Reactor。

```text
                  新连接
                     ↓
                  内核分配
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      listenfd A  listenfd B  listenfd C
      Reactor A   Reactor B   Reactor C
      accept/读写 accept/读写 accept/读写
```

### 并发如何产生

1. 内核选择一个监听 socket，将新连接交给它对应的接收队列。
2. 对应 worker 的 epoll 检测到监听就绪，执行 accept。
3. worker 将连接加入自己的 Reactor，负责后续读写。
4. 多个 worker 可以在多个 CPU 核上同时运行；每个 worker 内部通过非阻塞 I/O 处理多个连接。

SO_REUSEPORT 负责分流新连接，worker 负责实际并发执行。已建立连接通常持续由接收它的 worker 管理，不会每次读写都重新分配。

### 最小 C++ 示例

以下是 Linux 下的结构示例。省略初始化错误处理、连接处理函数实现和优雅退出。

```cpp
#include <arpa/inet.h>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <unistd.h>
#include <cerrno>
#include <thread>
#include <vector>

// 需实现：非阻塞读写、输入输出缓冲、事件更新、断开清理。
void handle_connection(int epfd, int fd, unsigned events);

void worker_loop() {
    // 每个 worker 创建不同的监听 socket。
    int listenfd = socket(
        AF_INET, SOCK_STREAM | SOCK_NONBLOCK | SOCK_CLOEXEC, 0);

    int enable = 1;
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEPORT,
               &enable, sizeof(enable)); // 必须在 bind 前设置

    sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_ANY);
    addr.sin_port = htons(8080);

    bind(listenfd, reinterpret_cast<sockaddr*>(&addr), sizeof(addr));
    listen(listenfd, SOMAXCONN);

    int epfd = epoll_create1(EPOLL_CLOEXEC);

    epoll_event ev{};
    ev.events = EPOLLIN; // 默认 LT
    ev.data.fd = listenfd;
    epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &ev);

    epoll_event events[128];
    while (true) {
        int n = epoll_wait(epfd, events, 128, -1);
        if (n < 0) {
            if (errno == EINTR) continue;
            break;
        }

        for (int i = 0; i < n; ++i) {
            int fd = events[i].data.fd;
            if (fd != listenfd) {
                handle_connection(epfd, fd, events[i].events);
                continue;
            }

            while (true) {
                int connfd = accept4(
                    listenfd, nullptr, nullptr,
                    SOCK_NONBLOCK | SOCK_CLOEXEC);

                if (connfd < 0) {
                    if (errno == EINTR) continue;
                    if (errno == EAGAIN || errno == EWOULDBLOCK)
                        break;
                    // 实际项目需分类处理其他 accept 错误。
                    break;
                }

                epoll_event conn_ev{};
                conn_ev.events = EPOLLIN;
                conn_ev.data.fd = connfd;
                if (epoll_ctl(epfd, EPOLL_CTL_ADD,
                              connfd, &conn_ev) < 0) {
                    close(connfd);
                }
            }
        }
    }

    // 实际退出时还需清理该 worker 持有的所有连接。
    close(epfd);
    close(listenfd);
}

int main() {
    std::vector<std::thread> workers;
    for (int i = 0; i < 4; ++i)
        workers.emplace_back(worker_loop);
    for (auto& thread : workers)
        thread.join();
}
```

核心约束：

- 各 worker 分别调用 socket，持有不同的监听 socket；仅共享同一个继承的 listenfd 不等于上述方案。
- 所有参与绑定的 socket 都在 bind 前设置 SO_REUSEPORT。
- Linux 下，参与绑定同一地址的进程需要具有相同的有效 UID。
- 可使用多线程，也可让多个进程分别完成上述初始化。
- 实际程序必须检查 socket、setsockopt、bind、listen、epoll 等调用的返回值。
- 事件回调应避免长时间阻塞；耗时任务会拖慢所属 Reactor。

### 效率与公平性的边界

该方案不需要集中 accept 后再通过队列或 IPC 移交连接，减少了分发锁和专门通知的开销。

默认分配通常基于连接信息的哈希，有利于分散连接，但不保证严格轮询、连接数完全相等或业务负载相等。繁忙长连接仍可能造成负载不均。需要自定义选择策略时，可进一步使用 reuseport BPF。

SO_REUSEPORT 与 SO_REUSEADDR 不是同一个选项；也不同于多个 Reactor 监听同一个 listenfd 并使用 EPOLLEXCLUSIVE 减少唤醒竞争。

## 7. 四种方案对比

| 方案 | 谁 accept | 交接方式 | 主要关注点 |
|---|---|---|---|
| 主进程通知 worker | worker | 管道传通知 | 接收竞争、重复通知、触发模式协调 |
| 主进程接收后分发 | 主进程 | Unix 域 socket + SCM_RIGHTS | fd 传递、背压、关闭责任 |
| 多线程主从 Reactor | 主线程 | 共享队列 + eventfd | 队列同步、唤醒、连接归属 |
| SO_REUSEPORT | 各 worker | 内核直接分配新连接 | 独立监听、分流与实际业务负载的差别 |
