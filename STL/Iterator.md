
## 概述

STL 迭代器的相关功能定义在 `<iterator>` 头文件中。这个头文件提供了**迭代器的定义、迭代器类别以及迭代器辅助函数的声明**。迭代器的基本操作有：

```cpp
std::vector<int>::iterator it; // 声明一个 vector 的迭代器 it
*it;         // 解引用迭代器 it，返回指向迭代器当前元素的引用
it++ 或 ++it // 将迭代器 it 后移一位
it-- 或 --it // 将迭代器 it 前移一位
it1 == it2;  // 比较两个迭代器是否指向同一位置
it1 != it2;  // 比较两个迭代器是否不指向同一位置
```

迭代器可以使用比较运算符（如 <、>、== 等）来对迭代器进行比较，以确定它们在容器中的相对位置。

---

## 分类

**输入迭代器**（Input Iterator）：只能**读取**容器中的元素，支持**单向**遍历且只能递增。
**输出迭代器**（Output Iterator）：只能**写入**容器中的元素，支持**单向**遍历且只能递增。
**前向迭代器**（Forward Iterator）：能够**读写**容器中的元素，支持**单向**遍历且只能递增。
**双向迭代器**（Bidirectional Iterator）：能够**读写**容器中的元素，且可递增和递减，支持**双向**遍历。
**随机访问迭代器**（Random Access Iterator）：提供**最完整的迭代器功能**，能够读写容器中的元素，且支持任意位置的遍历、比较操作和随机访问以及算术运算，如用 it1 - it2 计算两个迭代器之间的距离，用 `it[n]` 访问迭代器指向位置的第 n 个元素。

> **p.s. 比较两个迭代器的相对位置(<, >)，仅限随机访问迭代器**

---

## 迭代器的相关函数

以下的所有函数均适用于内置数组
1. `std::begin()` 和 `std::end()`返回指向容器开始和结束的迭代器, **支持所有容器**, 在C++11中引入:
	`auto it = std::begin(vec);`  
	**p.s. 注意与容器的成员函数vec.begin()的区别**
	
2. `std::rbein()` 与`std::rend()` 与上述类似但返回反向开始和结束的迭代器, 支持**适用于反向迭代的容器**.
3. `std::advance(Iterator& it, Distance n)` 将迭代器向前或向后移动n个位置, **支持所有容器**, 没返回值:
	`std::advance(it, 2); // 移动到第三个元素`
4. `std::distance(Iterator1, Iterator2)`计算两个向前迭代器之间的距离，即它们所指向的元素数量。
5. `std::next`和`std::prev`  返回一个新的**迭代器**，它指向由当前**迭代器 it 向前/向后 n 个位置的元素**。移动的步数，默认为 1，对 next 可以为正数向前或负数向后移动，对 prev 可以为正数向后或负数向前移动，但**原始迭代器 it 位置不变**。它们在 C++11 中被引入，适用于支持双向迭代的容器:
```cpp
auto next_it = std::next(it, 2); // next_it 现在指向第三个元素
std::cout << "Element after next 2 steps: " << *next_it << std::endl; // 输出 3

auto prev_it = std::prev(next_it, -1); // prev_it 现在指向第四个元素
std::cout << "Element before prev -1 step: " << *prev_it << std::endl; // 输出 4
```