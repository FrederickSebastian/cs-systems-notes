
>`epoll` 本身只解决一件事情:  **告诉你哪些文件描述符就绪了（可读 / 可写 / 异常）**
# 一、核心接口与参数（标注“传入/传出/双向”）

## 1. `epoll_create1`

```c
int epoll_create1(int flags);
```

### 参数

- `flags`（**传入**）
    
    - `0`
        
    - `EPOLL_CLOEXEC`：`exec` 后自动关闭该 epoll fd
        

### 返回值（**传出**）

- 成功：`epfd`（epoll 实例 fd）
    
- 失败：`-1`（检查 `errno`）
    

### 作用

创建**内核事件管理器实例**，后续所有 `epoll_ctl / epoll_wait` 都围绕这个 `epfd` 进行。

---

## 2. `epoll_ctl`

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

### 参数

- `epfd`（**传入**）  
    epoll 实例标识
    
- `op`（**传入**）  
    操作类型：
    
    - `EPOLL_CTL_ADD`：注册 fd
        
    - `EPOLL_CTL_MOD`：修改关注事件
        
    - `EPOLL_CTL_DEL`：删除 fd
        
- `fd`（**传入**）  
    被监控的文件描述符（监听 socket / 已连接 socket）
    
- `event`（**传入**；`DEL` 时可为 `NULL`）  
    指向 `struct epoll_event`，用于**定义监听规则 + 绑定用户数据**
    

---

### `struct epoll_event`

```c
struct epoll_event {
    uint32_t events;   // 事件掩码
    epoll_data_t data; // 用户数据
};
```

#### (1) `events`（**传入**）

关注的事件集合（位掩码）：

- I/O 事件
    
    - `EPOLLIN`：可读（数据到达 / 连接到来）
        
    - `EPOLLOUT`：可写
        
    - `EPOLLERR`：错误
        
    - `EPOLLHUP`：对端关闭
        
- 触发模式
    
    - 默认：LT（水平触发）
        
    - `EPOLLET`：ET（边缘触发）
        
- 其他
    
    - `EPOLLONESHOT`
        

---

#### (2) `data`（**传入 → 在 epoll_wait 中传出**）

```c
typedef union epoll_data {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;
```

用途：

- 在事件返回时携带**上下文**
    
- 常见：
    
    - `data.fd = fd`
        
    - `data.ptr = connection*`
        

---

### 返回值（**传出**）

- 成功：`0`
    
- 失败：`-1`
    

---

## 3. `epoll_wait`

```c
int epoll_wait(int epfd,
               struct epoll_event *events,
               int maxevents,
               int timeout);
```

### 参数

- `epfd`（**传入**）  
    epoll 实例
    
- `events`（**传出**）  
    **输出数组**，存放就绪事件
    
- `maxevents`（**传入**）  
    `events` 数组容量（最多返回多少事件）
    
- `timeout`（**传入**，毫秒）
    
    - `-1`：阻塞
        
    - `0`：立即返回
        
    - `>0`：超时等待
        

---

### 返回值（**传出**）

- `>0`：就绪事件数量
    
- `=0`：超时
    
- `=-1`：错误
    

---

### `events[i]` 的含义（**传出数据结构**）

- `events[i].events`：**实际发生的事件**
    
- `events[i].data`：你在 `epoll_ctl` 中写入的数据
    

---

# 二、这些参数在“建立连接 + 通信”中的作用

以 TCP 服务器为例：

---

## 1. 监听 socket 阶段

```c
listenfd = socket(...);
bind(...);
listen(...);
```

然后：

```c
ev.events = EPOLLIN;
ev.data.fd = listenfd;
epoll_ctl(epfd, EPOLL_CTL_ADD, listenfd, &ev);
```

### 作用

- `fd = listenfd`：监控监听套接字
    
- `events = EPOLLIN`：
    
    - 表示“有新连接到来”（不是数据）
        
- `data.fd`：
    
    - 用于在返回时识别这是监听 fd
        

---

## 2. 新连接到来（accept）

```c
if (events[i].data.fd == listenfd) {
    int connfd = accept(...);
}
```

---

## 3. 注册连接 socket

```c
ev.events = EPOLLIN;      // 关注读事件
ev.data.fd = connfd;

epoll_ctl(epfd, EPOLL_CTL_ADD, connfd, &ev);
```

### 作用

- `fd = connfd`：开始监控该连接
    
- `events`：决定你关心读还是写
    
- `data`：建立“事件 → 连接”的映射
    

---

## 4. 数据通信阶段

当：

```c
epoll_wait(...)
```

返回：

```c
events[i]
```

### 参数作用体现：

- `events[i].events`
    
    - 判断：
        
        - `EPOLLIN` → 调用 `read`
            
        - `EPOLLOUT` → 调用 `write`
            
- `events[i].data`
    
    - 找到对应连接（fd 或结构体）
        

---

## 5. 连接关闭 / 异常

```c
epoll_ctl(epfd, EPOLL_CTL_DEL, connfd, NULL);
close(connfd);
```

### 参数作用

- `fd`：指定要移除的连接
    
- `event`：不再需要
    

---

# 三、整体流程（多路 IO 转接）

## Step 1：创建 epoll

```c
epfd = epoll_create1(0);
```

---

## Step 2：注册监听 socket

```c
epoll_ctl(epfd, ADD, listenfd, &ev);
```

---

## Step 3：进入事件循环

```c
while (1) {
    int n = epoll_wait(epfd, events, ...);

    for (i = 0; i < n; i++) {
```

---

## Step 4：区分事件类型

### 情况 A：监听 fd

```c
if (events[i].data.fd == listenfd)
```

→ `accept` 新连接  
→ `epoll_ctl ADD connfd`

---

### 情况 B：普通连接 fd

```c
else
```

#### 读事件

```c
if (events[i].events & EPOLLIN)
    read(...)
```

#### 写事件

```c
if (events[i].events & EPOLLOUT)
    write(...)
```

---

## Step 5：异常处理

- `read = 0` → 对端关闭
    
- `read < 0`：
    
    - `EAGAIN`：正常（ET 模式）
        
    - 其他：关闭连接
        

---

## Step 6：维护 epoll

- 新连接 → ADD
    
- 状态变化 → MOD
    
- 关闭连接 → DEL
    

---

# 四、压缩总结（关键逻辑）

## 参数本质分工

|参数|本质作用|
|---|---|
|epfd|事件容器|
|fd|被监控对象|
|events（ctl）|关注什么|
|data|绑定上下文|
|events（wait）|返回发生了什么|

---

## 核心机制一句话

> **epoll_ctl 定义“监听规则”，epoll_wait 返回“就绪结果 + 上下文”，从而只处理活跃连接，实现高效多路复用。**