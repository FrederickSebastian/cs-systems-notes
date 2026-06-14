## 目录

- **第七章：TCP状态与端口复用**

  - [01P-TCP状态-其他状态](#01P-TCP状态-其他状态)
  - [02P-半关闭及shutdown函数](#02P-半关闭及shutdown函数)

---

# 第七章：TCP状态与端口复用

## 01P-TCP状态-其他状态

![图片](images/img_020.png)
netstat -apn | grep client    查看客户端网络连接状态
netstat -apn | grep port  查看端口的网络连接状态
![图片](images/img_053.png)
TCP状态时序图：
结合三次握手、四次挥手 理解记忆。
1. 主动发起连接请求端：CLOSE -- 发送SYN -- SEND_SYN -- 接收 ACK、SYN -- SEND_SYN -- 发送 ACK -- ESTABLISHED（数据通信态）
2. 主动关闭连接请求端： ESTABLISHED（数据通信态） -- 发送 FIN -- FIN_WAIT_1 -- 接收ACK -- FIN_WAIT_2（半关闭）
-- 接收对端发送 FIN -- FIN_WAIT_2（半关闭）-- 回发ACK -- TIME_WAIT（只有主动关闭连接方，会经历该状态）
-- 等 2MSL时长 -- CLOSE
3. 被动接收连接请求端： CLOSE -- LISTEN -- 接收 SYN -- LISTEN -- 发送 ACK、SYN -- SYN_RCVD -- 接收ACK -- ESTABLISHED（数据通信态）
4. 被动关闭连接请求端： ESTABLISHED（数据通信态） -- 接收 FIN -- ESTABLISHED（数据通信态） -- 发送ACK
-- CLOSE_WAIT (说明对端【主动关闭连接端】处于半关闭状态) -- 发送FIN -- LAST_ACK -- 接收ACK -- CLOSE
```c
重点记忆： ESTABLISHED、FIN_WAIT_2 <--> CLOSE_WAIT、TIME_WAIT（2MSL）
```

netstat -apn | grep  端口号
2MSL时长：
一定出现在【主动关闭连接请求端】。 --- 对应 TIME_WAIT 状态。
保证，最后一个 ACK 能成功被对端接收。（等待期间，对端没收到我发的ACK，对端会再次发送FIN请求。）

## 02P-半关闭及shutdown函数

端口复用:
```c
int opt = 1;// 设置端口复用。
setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, (void *)&opt, sizeof(opt));
```

半关闭：
通信双方中，只有一端关闭通信。  --- FIN_WAIT_2
```c
close（cfd）;
shutdown(int fd, int how);
```

how: SHUT_RD关读端
SHUT_WR关写端
SHUT_RDWR 关读写
shutdown在关闭多个文件描述符应用的文件时，采用全关闭方法。close，只关闭一个。