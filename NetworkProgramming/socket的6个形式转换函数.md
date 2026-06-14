# 字节序转换(CPU大小端)

| 函数    | 含义           | 全称                    | 作用   |
| ----- | ------------ | --------------------- | ---- |
| htonl | 主机 → 网络（32位） | host to network long  | IP地址 |
| htons | 主机 → 网络（16位） | host to network short | 端口号  |
| ntohl | 网络 → 主机（32位） | network to host long  | IP地址 |
| ntohs | 网络 → 主机（16位） | network to host short | 端口号  |
> 后缀位`l`的代表long类型, 也就是用于转换IP地址的, 后缀为`s`的代表short类型, 用于转换端口号(port)


# IP表示形式转换

## `inet_pton`
internet presentation to numeric, 把字符串IP转换为网络字节序的二进制IP

`int inet_pton(int af, const char *src, void *dst);`
### 参数: 
- `af`: address family, IPv4或者v6
- `src`: 字符串形式的IP地址
- `dst`: **输出参数** , 缓冲区, 用于存储二进制的IP, **实际类型是struct addr_in**
### 返回值

- `1`：成功
- `0`：地址格式错误
- `-1`：错误（如 `af` 不合法）

## `inet_ntop`
internet numeric to presentation, 把二进制IP转换为字符串

`const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);`

### 参数
- af略
- `src`: 传入参数, 二进制IP的地址, **实际上是struct addr_in **
- `dst`: 输出参数, 存储转换后的字符串IP
- `size`: 指定`dst`的缓冲区大小

### 返回值
- 成功返回`dst`
- 失败返回`NULL`