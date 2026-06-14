### Socket 建连四大函数参数速记

---

## 1. `socket()`

```c
int socket(int domain, int type, int protocol);
```

### 参数：

|参数|含义|本程序中|
|---|---|---|
|domain|通信协议族|`AF_INET` (IPv4)|
|type|socket类型|`SOCK_STREAM` (TCP)|
|protocol|具体协议|`0`（自动选TCP）|

---

### 记忆：

> “创建什么类型的通信端点”

即：

- 用哪种网络？
    
- 用TCP还是UDP？
    

---

---

## 2. `bind()`

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

### 参数：

|参数|含义|本程序中|
|---|---|---|
|sockfd|哪个socket绑定|`lfd`|
|addr|绑定到哪个本地地址|`&servAddr`|
|addrlen|地址结构长度|`sizeof(servAddr)`|

---

### 记忆：

> “把哪个socket绑到哪个服务器地址上”

---

---

## 3. `listen()`

```c
int listen(int sockfd, int backlog);
```

### 参数：

|参数|含义|本程序中|
|---|---|---|
|sockfd|哪个socket开始监听|`lfd`|
|backlog|最大等待连接数|`128`|

---

### 记忆：

> “让哪个socket开始排队接客”

---

---

## 4. `accept()`

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

### 参数：

|参数|含义|本程序中|
|---|---|---|
|sockfd|从哪个监听socket接收连接|`lfd`|
|addr|保存客户端地址|`&clinAddr`|
|addrlen|客户端地址长度（传入传出）|`&clinLen`|

---

### 返回值：

```c
cfd
```

新的通信socket。

---

### 记忆：

> “从哪个监听口取一个客户端，并把对方地址告诉我”

---

# 一句话总口诀

```text
socket 定类型
bind 绑自己
listen 开监听
accept 接别人
```

---

# 最核心区别

### bind 的 addr：

服务器自己的地址

### accept 的 addr：

客户端的地址

---

这个区别必须牢牢记住。