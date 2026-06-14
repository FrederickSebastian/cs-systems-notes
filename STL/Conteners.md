### 一. deque

1. std::deque 支持在**两端高效地进行插入和删除**操作，结合栈和队列的特点，提供了灵活的元素管理方式。头文件: `#include <deque>`

2. 常用的成员函数
```cpp
dq.push_back(x) / dq.push_front(x)          // 把 x 插入队尾/队首  O(1)
dq.back() / dq.front()                      // 返回队尾/队首元素  O(1)
dq.pop_back() / dq.pop_front()              // 删除队尾/队首元素  O(1)
dq.size()                                   // 返回 deque 的元素数量 O(1)
dq.empty()                                  // 检查容器是否为空，返回布尔值  O(1)
dq.at(pos)                                  // 返回位置 pos 的元素的引用，进行边界检查  O(1)
dq.swap(std::deque& other)                  // 交换两个 std::deque 的内容 O(1)
dq.reserve(size_t n)                        // 预分配 vector 的内存容量至能容纳 n 个元素  O(1)
dq.insert(pos, count, value)                // 在指定位置 pos 插入 count 个值为 value 的元素  O(N)
dq.erase(iterator it)                       // 删除 deque 中的某一个元素，由迭代器指定元素  O(N)
dq.erase(iterator first, iterator last)     // 删除 deque 中 [first, last) 中的元素  O(N)  
dq.clear()                                  // 清空 deque  O(N)
```

3. 作为双端队列使用的一些用法: 
```cpp
std::deque<int> dq;
dq.push_front(1); // 在队首插入
dq.push_back(2);  // 在队尾插入
dq.pop_front();   // 从队首删除
dq.pop_back();    // 从队尾删除
```

4. 有疑问的地方: 在需要移动元素时，可以使用 std::move 来避免不必要的拷贝操作：
```cpp
std::deque<std::string> dq1 = {"Hello", "World"};
std::deque<std::string> dq2;
dq2.push_back(std::move(dq1[0]));
```

## 二. list & forward_list

1. 前者是一个双向链表, 支持双向迭代器, 后者是单向链表, 只能单向遍历.
2. 需要强调其移动构造:
		`std::list<int> lst(std::move(lst2));`
3. 常用的成员函数(一些常见的略去):
``` cpp
lst.front() / back()                       // 返回头部/ 尾部元素的引用
lst.empty();                               // 检查容器是否为空，返回布尔值  O(1)
lst.insert(iterator pos, const T& value);  // 在指定位置 pos 插入值为 value 的元素  O(1)
lst.erase(iterator it);                    // 删除由迭代器 it 指定的元素  O(1)
lst.erase(iterator first, iterator last);  // 删除范围 [first, last) 中的元素  O(N)
lst.clear();                               // 清空 list  O(N)
lst.swap(std::list& other);                // 交换两个 std::list 的内容  O(1)
lst.sort();                                // 对容器中的元素进行排序  O(N log N)
lst.unique();                              // 删除容器中相邻的重复元素  O(N)
lst.remove(val);                           // 删除容器中所有等于 val 的元素  O(N)
lst.remove_if(predicate);                  // 删除容器中满足条件的元素 O(N)
```


## 三. map

1. map用于存储键值对, 其中每一个元素都是pair, 其中first是键, second 是值. **键是唯一的, 不能重复, 值可以重复**
		`std::map<int, std::string> mp1;`
		map会按照键的顺序从小大大自动排序, 因此**键的类型必须可以比较大小**
2. map基于红黑树实现, 增删查找的平均时间复杂度是O(logN), 但空间占用较大
3. 常用的成员函数:
``` cpp
mp.find(key)                           // 返回键为 key 的映射的迭代器  O(log N)
mp.erase(it)                           // 删除由迭代器对应指定的键和值  O(log N)
mp.erase(key)                          // 根据映射的键删除对应的键和值  O(log N)
mp.erase(first, last)                  // 删除 [first, last) 的 M 个迭代器对应的键和值  O(M * log N)
mp.size()                              // 返回映射的键值对数目  O(1)
mp.clear()                             // 清空 map 之中的所有元素  O(N)
mp.insert(std::make_pair(key, value))  // 构造新的键值对并插入该元素  O(log N)
mp.empty()                       // 如果 map 为空，返回真，否则返回假  O(1)
mp.begin()                       // 返回指向 map 第一个元素的迭代器  O(1)
mp.end()                         // 返回指向 map 最后一个元素的下一个迭代器  O(1)
mp.rbegin()                      // 返回指向 map 最后一个元素的迭代器  O(1)
mp.rend()                        // 返回指向 map 第一个元素的上一个地址  O(1)
mp.count(key)                    // 查看键为 key 的元素是否存在，若存在则返回 1，不存在则返回 0  O(log N)
mp.lower_bound(key)              // 返回首个键不小于 key 的迭代器(下界)，否则返回 end()  O(log N)
mp.upper_bound(key)              // 返回首个键不大于 key 的迭代器，(上界)否则返回 end()  O(log N)
```


3. 访问 map 单个元素可以直接根据**下标操作符 `[]` 根据键访问**对应的值，也可以用 **find()** 指定，其**键和值**可以分别通过其迭代器的 `it->first` 和 `it->second` 来访问：
```cpp
std::map<std::string, std::string> mp;               

mp["abcdefg"] = "qwertyu";            // 添加键值对，键为 "abcdefg"，值为 "qwertyu"

std::map<int, std::string> mp1;
mp1[1] = "hello";
mp1[2] = "world";

std::cout << mp1[1] << " " << mp[2] << std::endl; // 输出结果为 hello world

std::map<std::string, std::string>::iterator it = mp.find("abcdefg");
std::cout << it->first << " " << it->second << std::endl;
```


4. Tips: 逆向迭代器的使用:
```cpp
std::map <int, int> mp;
mp[1] = 2;
mp[2] = 3;
mp[3] = 4;
auto it = mp.rbegin();
while (it != mp.rend()) {
    std::cout << it->first << " " << it->second << std::endl;
    it++; // 反向遍历时迭代器也是从后往前递增
}
```


5. 元素的初始化Caution: 

```cpp
std::map<std::string, std::string> mp;                     

// 方式一使用下标操作符 [] 最简单
mp["学生"] = "学习";
mp["老师"] = "教书";

// 方式二接受一个 std::pair 类型的参数，其中包含键和值
mp.insert(std::make_pair("vegetable","蔬菜"));

// 方式三直接使用 std::pair 的模板参数
mp.insert(std::pair<std::string, std::string> ("fruit","水果"));

// 方式四使用花括号 {} 直接初始化一个键值对，C++11 及以上版本支持的语法
mp.insert({"hahaha","balabala"});

for (const auto& pair : mp) {
    std::cout << pair.first << " " << pair.second << std::endl;
}  // 高效output的tips
```



6. 所有的输入均被排序. 如果需要保持输入顺序, 可以用vector来push_back记录输入顺序.
7. 对于无序的unordered_map, 内部基于哈希表实现, 内部元素无需杂乱, 元素插入顺序与哈希值有关, 增删查找平均复杂度是O(1), 但建立哈希表比较费时.
		p.s. 这里对于unordered_map不做详细介绍, 其整体操作与map类似.


## 四. set

1. set基于红黑树实现, 具有自动排序功能(从小到大), 时间复杂度同上, 空间占用大
2. set中元素不能重复, 排序方式由元素类型的比较操作符(<)决定.
3. 常用成员函数:
``` cpp

//假设 s 为 set 集合的名称，其有 N 个元素，代码解释后包含算法时间复杂度
s.rbegin()                           // 返回逆序迭代器，指向容器元素最后一个位置  O(1)
s.rend()                             // 返回逆序迭代器，指向容器第一个元素前面的位置  O(1)
s.clear()                            // 删除 set 容器中的所有的元素,返回 unsigned int 类型  O(N)
s.empty()                            // 判断 set 容器是否为空，返回布尔值  O(1)
s.insert(value)                      // 插入一个元素 value  O(1)
s.size()                             // 返回当前set容器中的元素个数  O(1)
s.erase(key_value)                   // 删除键值key_value的值  O(1)
s.find(ele)                          // 查找 set 中的某一元素，有则返回该元素对应的迭代器，无则返回 end()  
s.count(ele)                         // 查询 ele 是否出现，返回 0 或 1  O(logN)
s.lower_bound(k)                     // 返回不小于 k 的第一个元素的迭代器  O(logN)
s.upper_bound(k)                     // 返回大于 k 的第一个元素的迭代器  O(logN)
```

3. 改变排序规则: set默认使用less比较器, 从小到大排序, **可以添加greater比较器改变排序规则**
		`std::set <int, greater<int>> s1; // 更改为从大到小排序`
	也可以初始化时自己定义Lambada表达式:
		`set<int, funcion<bool (int, int)>> s([&](int i, int j) {return i > j;})`
4. 对于undered_set, 内部基于哈希表实现, 其他的特性同上.

## 五. string

1. 常用的成员函数: 
 ```cpp
str.insert(pos, s)      // 在指定位置 pos 插入另一个字符串 s  O(n)
str.erase(pos, len)     // 删除从位置 pos 开始的长度为 len 的子字符串  O(n)，其中 n 是删除的子字符串长度
str.find(s)             // 查找子字符串 s 的位置，返回找到的位置的索引，无则返回 std::string::npos  O(n)
str.rfind(s)            // 从后向前查找子字符串 s 的位置，返回找到的位置的索引，无则返回 std::string::npos  O(n)
str.find_first_of(s)    // 查找子字符串 s 中任意字符第一次出现的位置，返回找到的位置的索引，无则返回 std::string::npos  O(n)
str.find_last_of(s)     // 查找子字符串 s 中任意字符最后一次出现的位置，返回找到的位置的索引，无则返回 std::string::npos  O(n)
str.compare(s)          // 比较两个字符串，返回一个整数，表示比较结果  O(n)
str.c_str()             // 返回一个指向 C - style 字符串的指针  O(1)
str.data()              // 返回一个指向字符串数据的指针  O(1)
str[0]                  // 通过下标访问字符串中的字符  O(1)
str.at(0)               // 通过下标访问字符串中的字符，进行边界检查  O(1)
```

2. p.s. string的size()和length()是一样的, 均返回字符数


### 一些其他的需要注意的东西
1. 对于at, 与下标操作符最大的区别是: at()会自动进行边界检查:
 ``` cpp
 std::vector<int> v = {10, 20, 30};

// 安全访问
int value = v.at(1); // 返回 20

// 越界访问
try {
    int x = v.at(5); // 索引 5 越界
} catch (const std::out_of_range& e) {
    std::cout << "错误: " << e.what() << std::endl;
}
 ```

2. 对于二者的更多的区别点: 
	1. 当出现越界行为时, at( )会抛出out_of_range异常, 后者会出现未定义行为
	2. 性能: at略慢
	3. 在关联容器中的特殊性: 
	- **`m[key]`**: 如果 `key` 不存在，它会调用该类型的默认构造函数，往 map 里插入一个新的键值对.
    -  **`m.at(key)`**: 如果 `key` 不存在，它**不会插入**任何东西，而是直接抛出异常。
	这使得 `at()` 成为在 `const` 环境下查找元素的唯一选择（因为 `[]` 可能会修改容器，所以不能用于 `const map`）。

3. 在 C++11 后可以**使用 emplace_back 代替 push_back**，都是往容器末尾添加元素但底层实现机制不同。
     - **push_back()** 会先创建这个元素，然后再将这个元素拷贝（事后会自行销毁）或移动到容器中。
     - **emplace_back()** 则直接在容器尾部使用完美转发创建这个元素，在构造复杂对象时效率更优。