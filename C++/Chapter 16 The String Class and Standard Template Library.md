

---


## 一、string类

1. C++ 提供了功能强大的 `string` 类，包含丰富的成员方法（详见 _p.531_）。  
    它是一个动态字符数组的封装，支持自动扩容、拼接、查找、比较、迭代等操作。
    

---

## 二、智能指针（Smart Pointers）

2. C++11 引入了两种常用的智能指针：  
    `std::unique_ptr` 与 `std::shared_ptr`。  
    将 `new` 返回的地址赋给它们，离开作用域时内存自动释放，无需手动 `delete`。
    
    使用前需：
    
    - 头文件：`<memory>`
        
    - 命名空间：`std`
        
    
    创建方式：
    
    ```cpp
    unique_ptr<double> pdu(new double);
    ```
    
3. 由于两者的构造函数为 `explicit`，**不能隐式转换**普通指针：
    
    ```cpp
    double* p_reg = new double;
    shared_ptr<double> pd = shared_ptr<double>(p_reg);
    ```
    
4. 注意：**不要**将非堆内存（如字符串常量）赋给智能指针。
    
5. 二者区别：
    
    - **`unique_ptr`**
        
        - 独占所有权，不能复制。
            
        - 可使用 `std::move()` 转移所有权：
            
            ```cpp
            ps2 = std::move(ps1);
            ```
            
    - **`shared_ptr`**
        
        - 允许多个指针共享同一块内存。
            
        - 使用**引用计数机制**自动管理生命周期。
            

---

## 三、STL 概述

6. STL（Standard Template Library）由四大核心组件构成：
    
    - **容器（Containers）**：存放数据（类似数组）
        
    - **迭代器（Iterators）**：访问容器中元素（广义指针）
        
    - **算法（Algorithms）**：通用操作，如 `sort()`、`copy()`、`for_each()`
        
    - **函数对象（Function Objects）**：像函数一样使用的对象
        

---

## 四、STL 常用方法

7. 常用容器方法：
    
    - `size()`：返回元素数量
        
    - `swap()`：交换两个容器内容
        
    - `begin()` / `end()`：返回首迭代器与尾后迭代器
        
    - `push_back()`：向末尾添加元素
    - `emplace_back()` 则直接在容器尾部使用完美转发创建这个元素，在构造复杂对象时效率更优。用法同 `push_back()` 
    - `emlace_front` 用法相同, 在元素首部添加
        
    - `erase()`：删除指定范围的元素
        
        ```cpp
        scores.erase(scores.begin(), scores.begin() + 3);  // 删除下标0–2
        ```
        
    - `insert()`：插入新元素
        
        ```cpp
        scores.insert(scores.begin(), newScores.begin(), newScores.end());
        ```
        
8. 定义迭代器：
    
    ```cpp
    vector<double>::iterator pd = scores.begin();
    ```
    
    建议优先使用 STL 算法（如 `for_each()`），而非手动迭代。
    **Tips: 用auto简化: `auto pd = score.begin();` **
    

---

## 五、STL 常用算法

9. 三个代表性算法：
    
    - **`for_each()`**：对区间内每个元素执行指定函数
        
        ```cpp
        for_each(books.begin(), books.end(), ShowReview);
        ```
        
        不修改容器元素。
        
    - **`sort()`**：升序排列
        
        ```cpp
        sort(v.begin(), v.end());
        ```
        
        若自定义类型需重载 `<` 运算符。
        
    - **`random_shuffle()`**：随机打乱元素顺序（要求随机访问迭代器）。
    *但是已在C++17中弃用*
        
9.  如果想要改变容器中的元素，加上引用即可：
    

```cpp
for (auto x : books)
    Show(x);

for (auto& x : books)
    change(x);  // 可修改元素
```
10. Tips: 迭代器是广义指针，ar是数组名，所以可以这样用：
	- `for_each(ar, ar + n, ShowReview);`
---

## 六、STL 迭代器类型

11. 五种标准迭代器（Iterator Categories）：
    

- **输入迭代器**：单向，只读输入（从容器读出数据）。
    
- **输出迭代器**：单向，只写输出（将数据写入容器）。
    
- **正向迭代器**：可多次遍历，支持读写。
    
- **双向迭代器**：支持 `++` 和 `--`。
    
- **随机访问迭代器**：支持随机跳转（`+`、`-`、下标访问）。
    

 这些不是类型，而是一组**行为要求（concepts）**。

---

## 七、STL 概念体系

12. STL定义了若干通用术语：
    

- **概念 (Concept)**：描述算法或类型应满足的一系列要求。
    
- **模型 (Model)**：概念的具体实现。
    
- **改进 (Refinement)**：概念的继承关系。
    
- **适配器 (Adapter)**：将其他接口适配为 STL 接口的类或函数（如 `stack`）。
    

---

## 八、STL 算法与 I/O 迭代器

13. STL算法可直接作用于普通数组指针（被视作迭代器）。
    
14. `copy()` 示例：
    

```cpp
copy(v1.begin(), v1.end(), v2.begin());
```

注意目标区必须足够大。

15. I/O 迭代器：
    

```cpp
#include <iterator>

ostream_iterator<int, char> out_iter(cout, " ");  //第一个参数是被发送给输出的类型，第二个是输出实用的类型
*out_iter++ = 15;  // 将15发送到输出流中，并为下一次输出做好准备
copy(dice.begin(), dice.end(), out_iter);
```

从输入流读取：

```cpp
copy(istream_iterator<int, char>(cin),
     istream_iterator<int, char>(),
     dice.begin());
```

16. 三种插入迭代器：
    

```cpp
front_insert_iterator<list<int>> f_iter(lst);
back_insert_iterator<vector<int>> b_iter(v);
insert_iterator<vector<int>> i_iter(v, v.begin());
```

可自动扩容，区别于 `copy()`。

---

## 九、STL 容器

17. STL 区分 **容器概念** 与 **容器类型**：
    

- 概念是抽象分类。
    
- 类型是可实例化的模板。
    

18. 常见序列容器（Sequential Containers）：
    

|容器|特点|适用场景|
|---|---|---|
|`vector`|动态数组，连续存储|频繁随机访问、尾部插入删除|
|`deque`|双端队列|头尾两端插入删除|
|`list`|双向链表|任意位置插入删除|
|`forward_list`|单向链表|内存紧凑、单向遍历|
|`array`|固定长度静态数组封装|编译期已知长度，高性能|
|`string`|动态字符数组|文本处理|
|`span` (C++20)|数组“视图”，不拥有数据|安全访问已有数组片段|

---

## 十、关联容器（Associative Containers）

19. 关联容器通过**键（key）管理数据，底层基于红黑树或哈希表。**
    

|类型|特点|适合场景|
|---|---|---|
|`set`|存储唯一键，自动排序|去重、快速查找|
|`map`|键值对映射，按键排序|字典、索引表|
|`multiset` / `multimap`|允许重复键|频繁插入且有重复项|

---

## 十一、函数对象与算法扩展

20. **函数对象（函数符）**：能像函数一样调用的对象。  
    包括函数名、函数指针、以及重载了 `()` 运算符的对象。  
    STL 定义了一元、二元函数符，以及谓词（返回 bool）。
    
21. **`transform()` 函数**：
    

- 单区间版本：
    
    ```cpp
    transform(src.begin(), src.end(), dest.begin(), op);
    ```
    
- 双区间版本：
    
    ```cpp
    transform(a.begin(), a.end(), b.begin(), dest.begin(), op2);
    ```
    

22. STL 为所有 **算术、逻辑、关系运算符** 都提供了对应函数符。
    
23. 自适应函数符与函数适配器（function adapters）  
    能将函数符与算法结合，形成可组合的泛型函数。
    

---

## 十二、算法与头文件分组

25. STL 算法主要分为四类：
    

- **非修改序列操作**：`find()`, `for_each()`
    
- **修改序列操作**：`copy()`, `transform()`
    
- **排序与相关操作**：`sort()`, `unique()`, `merge()`
    
- **数值运算操作**：`accumulate()`（头文件 `<numeric>`）
    

前三组在 `<algorithm>`，最后一组在 `<numeric>`。  
编译器不会检查迭代器类型是否匹配，错误会导致大量编译错误。

---

## 十三、其他重要特性

26. `string` 虽不属于 STL 容器，但完全支持 STL 接口。
    
27. `valarray` 与 `array`：
    
    - `valarray`：面向数值计算
        
    - `array`：静态数组的安全替代
        
28. `initializer_list`：用于直接初始化容器：
    

```cpp
std::vector<double> pay{2.1, 31.1, 56.3};
```

包含头文件 `<initializer_list>`，其迭代器为 `const`，不可修改值。

---

##  Summary

- `string` 类是动态字符容器，STL 容器的特化形式。
    
- 智能指针通过 RAII 管理动态内存，`unique_ptr` 独占，`shared_ptr` 共享。
    
- STL 由容器、迭代器、算法、函数对象组成，是 C++ 泛型编程核心。
    
- 五类迭代器定义了访问能力层级。
    
- 容器分为序列容器与关联容器；算法依赖迭代器而非容器类型。
    
- `copy()`、`transform()`、`for_each()` 是最通用的算法模型。
    
- `initializer_list` 提供现代化的统一初始化方式。
    

---

以上是这一章的详细笔记，以下是概述：

--- 

##  STL 的总体结构

STL (Standard Template Library) 其实是个**泛型编程框架**，不是某个类库集合。  
它由 **六大核心组件** 构成，核心思想是——**容器 + 算法 + 迭代器 + 函数对象 + 适配器 + 分配器**。

|组件|职责|举例|
|---|---|---|
|**容器 (Container)**|存储数据|vector, list, map|
|**算法 (Algorithm)**|处理数据|sort, copy, find, transform|
|**迭代器 (Iterator)**|连接算法与容器的“桥梁”|begin(), end(), ++it|
|**函数对象 (Functor)**|让算法可定制行为|greater(), custom lambda|
|**适配器 (Adapter)**|改变接口以适应使用场景|stack, queue, function adapter|
|**分配器 (Allocator)**|管理内存分配（一般默认）|std::allocator|

STL 的哲学是：

> “算法不知道数据来自哪里，容器不知道数据被如何使用。”

算法只依赖“迭代器”，迭代器抽象出访问方式，容器具体存放数据。这样，任何算法都能和任何容器组合。

---

##  一、容器：存东西的盒子

容器分三类：

### 1️⃣ 序列容器（按顺序存）

> 元素按插入顺序排列。

|类型|底层结构|特点|适用场景|
|---|---|---|---|
|**vector**|动态数组|随机访问快，尾部操作快|默认首选|
|**deque**|分段数组|双端插删快|两头操作都频繁|
|**list**|双向链表|任意位置插删快|中间操作多|
|**forward_list**|单链表|轻量单向操作|低内存或嵌入式|
|**array**|静态数组封装|固定大小|性能敏感场景|
|**string**|特化的动态数组|文本操作|字符串|

---

### 2️⃣ 关联容器（按键存）

> 自动根据 key 排序（红黑树）。

|类型|特点|适用场景|
|---|---|---|
|**set**|元素唯一，自动升序|去重、自动排序|
|**multiset**|可重复元素|多值集合|
|**map**|键值对，键唯一|字典、索引表|
|**multimap**|键可重复|多值映射|

---

### 3️⃣ 无序容器（哈希表实现）

> C++11 之后引入。

|类型|特点|场景|
|---|---|---|
|**unordered_set / map**|基于哈希表，不排序|更快查找（平均 O(1)）|
|**unordered_multiset / multimap**|可重复键|同上|

---

## 🧭 二、迭代器：算法与容器的桥梁

迭代器像“智能指针”，提供统一的访问方式。不同容器支持不同迭代器类型。

|迭代器类型|功能|常见容器|
|---|---|---|
|输入迭代器|只读单向|istream_iterator|
|输出迭代器|只写单向|ostream_iterator|
|正向迭代器|多次单向读写|forward_list|
|双向迭代器|可 ++ / --|list, set, map|
|随机访问迭代器|可随机跳转|vector, deque, array|

**关键逻辑：**

> 算法只依赖迭代器类型，而不是容器类型。

比如 `sort()` 要求随机访问迭代器，所以你不能对 `list` 用它（list 只有双向迭代器）。

---

## ⚙️ 三、算法：不关心容器的逻辑模块

算法是 STL 的灵魂。所有算法都通过迭代器访问数据。

📦 分类大致如下：

|分类|举例|功能|
|---|---|---|
|**非修改性**|find, for_each, count|不改变容器|
|**修改性**|copy, transform, remove|改变或复制数据|
|**排序及相关**|sort, unique, merge, random_shuffle|排序、重排|
|**数值运算**|accumulate, inner_product|数学处理（）|

👉 小技巧：

- 大多数算法返回“迭代器”，不是修改容器的“成员函数”。
    
- `copy` 不扩容，`back_inserter` 可自动扩容。
    
- `for_each` 不修改元素（除非引用传参）。
    
- `transform` 用于“映射式”操作。
    

---

## 🧩 四、函数对象（仿函数）

函数对象 = 可以被当成函数用的对象。  
好处：能携带状态、内联优化、可复用。

```cpp
struct Add { int operator()(int a, int b){ return a + b; } };
Add add;
cout << add(3, 4); // 输出7
```

STL 还提供现成的：

- `plus<T>`, `minus<T>`, `greater<T>`, `equal_to<T>` …
    
- 谓词（返回 bool）常配合算法使用，如 `sort(v.begin(), v.end(), greater<int>())`
    

---

## 🪄 五、适配器（Adapter）

适配器是接口转换器，让旧接口兼容 STL。

### 容器适配器

|类型|基于容器|功能|
|---|---|---|
|**stack**|deque（默认）|后进先出|
|**queue**|deque|先进先出|
|**priority_queue**|vector + 堆算法|优先队列|

### 迭代器适配器

- `insert_iterator`、`back_insert_iterator`、`front_insert_iterator`
    
- 输入输出适配：`istream_iterator` / `ostream_iterator`
    

### 函数适配器

- 用来修饰函数对象，比如 `bind2nd`（已被 lambda 替代）。
    

---

##  六、分配器（Allocator）

负责内存分配、释放。一般用默认的 `std::allocator`。  

>各种STL容器模板都接受一个可选的模板参数, 该参数指定使用哪个分配器来管理内存. 例如, vector模板开头如下: 
>	`template <class T, class Allocator = allocator<T> >`
>		`class vector ...`
>如果省略该模板参数的值, 则容器将默认用`allocator<T>`类.

了解即可. 

---

## 七、STL 的思维方式：泛型 + 抽象 + 组合

STL 本质不是“记 API”，而是 **抽象数据 + 操作分离**。

### 举个通透的例子：

```cpp
vector<int> v = {1, 2, 3, 4, 5};
transform(v.begin(), v.end(), v.begin(), [](int x){ return x * 2; });
copy(v.begin(), v.end(), ostream_iterator<int>(cout, " "));
```

 你不需要知道 `transform` 怎么实现，不关心 `vector` 怎么存。  
一切靠迭代器 + 函数组合。

---

## 🧩 八、如何从“知道”变成“体系”

现在你可以这样整理知识框架：

```
STL
│
├─ 容器
│   ├─ 序列容器
│   ├─ 关联容器
│   └─ 无序容器
│
├─ 迭代器（五种）
│
├─ 算法
│   ├─ 非修改
│   ├─ 修改
│   ├─ 排序
│   └─ 数值
│
├─ 函数对象
│
├─ 适配器
│   ├─ 容器适配器
│   ├─ 函数适配器
│   └─ 迭代器适配器
│
└─ 分配器
```

---

## 🧭 最后，实用学习路线（建议）

1. **打好迭代器和算法的连接感**：  
    多写 `for_each`, `transform`, `copy`, `accumulate` 练组合用法。
    
2. **掌握容器特性**：  
    对比 `vector/list/deque/map` 的底层结构和复杂度。
    
3. **写几个综合练习**：
    
    - 用 `map<string,int>` 统计单词频率
        
    - 用 `list` 模拟任务队列
        
    - 用 `transform` + `lambda` 实现批量转换
        
4. **读 `<algorithm>` 头文件源码（或cppreference）**  
    你会突然理解 STL 的“接口一致性”哲学。
    

---
