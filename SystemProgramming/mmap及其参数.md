`mmap` 是 Linux 中一个非常核心的系统调用，用于**将文件或设备映射到进程的虚拟地址空间**，或者**创建匿名内存区域**。它是实现高效 I/O、共享内存、内存管理等机制的基础。

---

## 一、函数原型

在用户态通过 libc 提供的接口（本质仍是系统调用）：

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
```

---

## 二、参数解析（逐个精确说明）

### 1. `addr`

- 指定映射的**起始地址（建议值）**
    
- 通常传 `NULL`，由内核自动选择
    
- 如果使用 `MAP_FIXED`，则必须使用指定地址（危险）
    

---

### 2. `length`

- 映射区域的长度（字节）
    
- 必须 > 0
    
- 实际会按页大小（通常 4KB）对齐
    

---

### 3. `prot`（内存保护属性）

常用组合：

|标志|含义|
|---|---|
|`PROT_READ`|可读|
|`PROT_WRITE`|可写|
|`PROT_EXEC`|可执行|
|`PROT_NONE`|不可访问|

示例：

```c
PROT_READ | PROT_WRITE
```

---

### 4. `flags`（映射类型）

这是关键参数，决定行为：

#### 常见选项：

|标志|含义|
|---|---|
|`MAP_SHARED`|修改会写回文件（共享）|
|`MAP_PRIVATE`|写时复制（COW），不影响文件|
|`MAP_ANONYMOUS`|匿名映射（不使用文件）|
|`MAP_FIXED`|强制使用指定地址|
|`MAP_POPULATE`|预加载页|

⚠️ `MAP_SHARED` vs `MAP_PRIVATE` 是核心区别。

---

### 5. `fd`

- 文件描述符
    
- 如果使用 `MAP_ANONYMOUS`，这里填 `-1`
    

---

### 6. `offset`

- 文件偏移量
    
- 必须是页大小的整数倍
    

---

## 三、返回值

- 成功：返回映射地址（`void *`）
    
- 失败：返回 `MAP_FAILED`（即 `(void *)-1`），并设置 `errno`
    

---

## 四、典型使用场景

### 1. 文件映射（零拷贝 I/O）

```c
int fd = open("file.txt", O_RDONLY);
char *p = mmap(NULL, 4096, PROT_READ, MAP_PRIVATE, fd, 0);

// 直接访问文件内容
printf("%s\n", p);

munmap(p, 4096);
close(fd);
```

**优势：**

- 避免 `read()` 的用户态/内核态拷贝
    
- 提高性能
    

---

### 2. 匿名映射（代替 malloc）

```c
int *p = mmap(NULL, sizeof(int),
              PROT_READ | PROT_WRITE,
              MAP_PRIVATE | MAP_ANONYMOUS,
              -1, 0);

*p = 42;

munmap(p, sizeof(int));
```

---

### 3. 进程间共享内存

```c
int *p = mmap(NULL, sizeof(int),
              PROT_READ | PROT_WRITE,
              MAP_SHARED | MAP_ANONYMOUS,
              -1, 0);
```

结合 `fork()` 后：

- 父子进程共享同一块内存
    

---

## 五、底层机制（核心理解）

### 1. 虚拟内存映射

`mmap` 本质上是在进程的页表中建立映射：

```
虚拟地址 → 物理页 / 文件页
```

---

### 2. 懒加载（Demand Paging）

- 调用 `mmap` 时**不会立即加载数据**
    
- 第一次访问时触发 **缺页异常（page fault）**
    
- 内核才真正加载数据
    

---

### 3. 写时复制（COW）

对于 `MAP_PRIVATE`：

- 初始共享同一物理页
    
- 写入时复制一份新页
    

---

## 六、与 read/write 的对比

|特性|mmap|read/write|
|---|---|---|
|数据拷贝|无（零拷贝）|有|
|编程复杂度|较高|简单|
|随机访问|高效|较慢|
|大文件处理|优秀|一般|

---

## 七、注意事项（容易踩坑）

1. **访问越界会 SIGSEGV**
    
2. **文件大小不足会导致错误**
    
3. `offset` 必须页对齐
    
4. 映射后文件被删除仍可访问（引用计数机制）
    
5. 需要手动 `munmap`
    

---

## 八、相关系统调用

- `munmap()`：解除映射
    
- `mprotect()`：修改权限
    
- `msync()`：同步到磁盘
    
- `brk()` / `sbrk()`：传统堆管理（对比 mmap）
    

---

## 九、总结（简洁版）

`mmap` 的本质：

> **建立“文件/内存 ↔ 虚拟地址空间”的映射关系，使文件像内存一样访问。**

它是以下技术的基础：

- 零拷贝 I/O
    
- 共享内存
    
- 动态链接库加载
    
- 内存分配器（glibc malloc 内部也用 mmap）
    

