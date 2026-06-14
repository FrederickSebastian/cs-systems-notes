|项目|TCP Server|TCP Client|UDP Server|UDP Client|
|---|---|---|---|---|
|创建 socket|`socket(SOCK_STREAM)`|同左|`socket(SOCK_DGRAM)`|同左|
|是否必须 bind|✔（必须）|✘（通常不需要）|✔（必须）|✘（通常不需要）|
|bind 是否需要 IP|可选（常用 `0.0.0.0`）|—|可选（常用 `0.0.0.0`）|—|
|bind 是否需要端口|✔|—|✔|—|
|是否需要 listen|✔|✘|✘|✘|
|是否需要 accept|✔|✘|✘|✘|
|是否需要 connect|✘|✔（必须指定 IP+端口）|✘（默认）|✘（默认）|
|发送函数|`send()`|`send()`|`sendto()`|`sendto()`|
|接收函数|`recv()`|`recv()`|`recvfrom()`|`recvfrom()`|
|是否要写对方 IP|connect 时写一次|✔（connect）|每次 recvfrom 得到|每次 sendto 指定|
|是否要写对方端口|connect 时写一次|✔|每次 recvfrom 得到|每次 sendto 指定|