## 目录

- **第十三章：UDP通信**

  - [01P-TCP和UDP通信优缺点](#01P-TCP和UDP通信优缺点)
  - [02P-UDP通信server和client流程](#02P-UDP通信server和client流程)
  - [03P-recvfrom和sendto函数](#03P-recvfrom和sendto函数)
  - [04P-UDP实现的并发服务器和客户端](#04P-UDP实现的并发服务器和客户端)
  - [05P-借助TCP的CS模型，改写UDP的CS模型](#05P-借助TCP的CS模型，改写UDP的CS模型)

---

# 第十三章：UDP通信

## 01P-TCP和UDP通信优缺点

TCP通信和UDP通信各自的优缺点：
TCP：面向连接的，可靠数据包传输。对于不稳定的网络层，采取完全弥补的通信方式。 丢包重传。
优点：
稳定。
数据流量稳定、速度稳定、顺序
缺点：
传输速度慢。相率低。开销大。
使用场景：数据的完整型要求较高，不追求效率。
大数据传输、文件传输。
UDP：无连接的，不可靠的数据报传递。对于不稳定的网络层，采取完全不弥补的通信方式。 默认还原网络状况
优点：
传输速度块。相率高。开销小。
缺点：
不稳定。
数据流量。速度。顺序。
使用场景：对时效性要求较高场合。稳定性其次。
游戏、视频会议、视频电话。腾讯、华为、阿里  ---  应用层数据校验协议，弥补udp的不足。

## 02P-UDP通信server和client流程

UDP实现的 C/S 模型：
recv()/send() 只能用于 TCP 通信。 替代 read、write
```c
accpet(); ---- Connect(); ---被舍弃
```

server：
```c
lfd = socket(AF_INET, STREAM, 0);SOCK_DGRAM --- 报式协议。
bind();
listen();  --- 可有可无
while（1）{
    read(cfd, buf, sizeof) --- 被替换 --- recvfrom（） --- 涵盖accept传出地址结构。
    小-- 大
    write();--- 被替换 --- sendto（）---- connect
}
close();
```

client：
```c
connfd = socket(AF_INET, SOCK_DGRAM, 0);
sendto（'服务器的地址结构'， 地址结构大小）
recvfrom（）
写到屏幕
close();
```

## 03P-recvfrom和sendto函数

```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,struct sockaddr *src_addr, socklen_t *addrlen);
```

sockfd： 套接字
buf：缓冲区地址
len：缓冲区大小
flags： 0
```c
src_addr：（struct sockaddr *）&addr 传出。 对端地址结构
```

addrlen：传入传出。
返回值： 成功接收数据字节数。 失败：-1 errn。 0： 对端关闭。
```c
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,const struct sockaddr *dest_addr, socklen_t addrlen);
```

sockfd： 套接字
buf：存储数据的缓冲区
len：数据长度
flags： 0
```c
src_addr：（struct sockaddr *）&addr 传入。 目标地址结构
```

addrlen：地址结构长度。
返回值：成功写出数据字节数。 失败 -1， errno

## 04P-UDP实现的并发服务器和客户端

直接上代码，啃，啃就完事儿，这是服务器代码
```c
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <ctype.h>
#define SERV_PORT 8000
int main(void)
{
    struct sockaddr_in serv_addr, clie_addr;
    socklen_t clie_addr_len;
    int sockfd;
    char buf[BUFSIZ];
    char str[INET_ADDRSTRLEN];
    int i, n;
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    bzero(&serv_addr, sizeof(serv_addr));
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
    serv_addr.sin_port = htons(SERV_PORT);
    bind(sockfd, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    printf("Accepting connections ...\n");
    while (1) {
        clie_addr_len = sizeof(clie_addr);
        n = recvfrom(sockfd, buf, BUFSIZ,0, (struct sockaddr *)&clie_addr, &clie_addr_len);
        if (n == -1)
            perror("recvfrom error");
        printf("received from %s at PORT %d\n",
            inet_ntop(AF_INET, &clie_addr.sin_addr, str, sizeof(str)),
            ntohs(clie_addr.sin_port));
        for (i = 0; i < n; i++)
            buf[i] = toupper(buf[i]);
        n = sendto(sockfd, buf, n, 0, (struct sockaddr *)&clie_addr, sizeof(clie_addr));
        if (n == -1)
            perror("sendto error");
    }
    close(sockfd);
    return 0;
}
```

下面是客户端代码：
```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <ctype.h>
#define SERV_PORT 8000
int main(int argc, char *argv[])
{
    struct sockaddr_in servaddr;
    int sockfd, n;
    char buf[BUFSIZ];
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    inet_pton(AF_INET, "127.0.0.1", &servaddr.sin_addr);
    servaddr.sin_port = htons(SERV_PORT);
    bind(sockfd, (struct sockaddr *)&servaddr, sizeof(servaddr));
    while (fgets(buf, BUFSIZ, stdin) != NULL) {
        n = sendto(sockfd, buf, strlen(buf), 0, (struct sockaddr *)&servaddr, sizeof(servaddr));
        if (n == -1)
            perror("sendto error");
        n = recvfrom(sockfd, buf, BUFSIZ, 0, NULL, 0);         //NULL:不关心对端信息
        if (n == -1)
            perror("recvfrom error");
        write(STDOUT_FILENO, buf, n);
    }
    close(sockfd);
    return 0;
}
```

## 05P-借助TCP的CS模型，改写UDP的CS模型

看懂前面的，问题就不大了。可以再看一下视频复习复习