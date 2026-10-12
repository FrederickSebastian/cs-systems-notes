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
		map默认按照键从小到大自动排序, **比较器需要满足严格弱序**；默认使用 `std::less<Key>`，也可以自定义比较器。
2. map基于红黑树实现, 增删查找的平均时间复杂度是O(logN), 但空间占用较大
3. 常用的成员函数:
``` cpp
mp.find(key)                           // 返回键为 key 的映射的迭代器  O(log N)
mp.erase(it)                           // 删除由迭代器对应指定的键和值  均摊 O(1)
mp.erase(key)                          // 根据映射的键删除对应的键和值  O(log N)
mp.erase(first, last)                  // 删除 [first, last) 的 M 个迭代器对应的键和值  O(log N + M)
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
mp.upper_bound(key)              // 返回首个键大于 key 的迭代器(上界)，否则返回 end()  O(log N)
```


3. 访问 map 单个元素可以直接根据**下标操作符 `[]` 根据键访问**对应的值，也可以用 **find()** 指定，其**键和值**可以分别通过其迭代器的 `it->first` 和 `it->second` 来访问：
```cpp
std::map<std::string, std::string> mp;               

mp["abcdefg"] = "qwertyu";            // 添加键值对，键为 "abcdefg"，值为 "qwertyu"

std::map<int, std::string> mp1;
mp1[1] = "hello";
mp1[2] = "world";

std::cout << mp1[1] << " " << mp1[2] << std::endl; // 输出结果为 hello world

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
7. 不需要按键排序时，可以考虑 `unordered_map`，详见第五节。


## 四. set

1. set基于红黑树实现, 具有自动排序功能(从小到大), 时间复杂度同上, 空间占用大
2. set中元素不能重复, 排序方式由元素类型的比较操作符(<)决定.
3. 常用成员函数:
``` cpp

//假设 s 为 set 集合的名称，其有 N 个元素，代码解释后包含算法时间复杂度
s.rbegin()                           // 返回逆序迭代器，指向容器元素最后一个位置  O(1)
s.rend()                             // 返回逆序迭代器，指向容器第一个元素前面的位置  O(1)
s.clear()                            // 删除 set 容器中的所有元素，返回 void  O(N)
s.empty()                            // 判断 set 容器是否为空，返回布尔值  O(1)
s.insert(value)                      // 插入一个元素 value  O(log N)
s.size()                             // 返回当前set容器中的元素个数  O(1)
s.erase(key_value)                   // 删除指定元素，返回删除数量（0 或 1） O(log N)
s.find(ele)                          // 查找元素，成功返回对应迭代器，否则返回 end()  O(log N)
s.count(ele)                         // 查询 ele 是否出现，返回 0 或 1  O(logN)
s.lower_bound(k)                     // 返回不小于 k 的第一个元素的迭代器  O(logN)
s.upper_bound(k)                     // 返回大于 k 的第一个元素的迭代器  O(logN)
```

3. 改变排序规则: set默认使用less比较器, 从小到大排序, **可以添加greater比较器改变排序规则**
		`std::set<int, std::greater<int>> s1; // 更改为从大到小排序，需 <functional>`
	也可以初始化时自己定义Lambada表达式:
		`std::set<int, std::function<bool(int, int)>> s([](int i, int j) { return i > j; }); // 需 <functional>`


## 五. unordered_map

1. `std::unordered_map` 是 **C++11 起提供的无序键值对容器**，头文件为 `#include <unordered_map>`。键唯一，值可以重复，每个元素的类型为 `std::pair<const Key, T>`，可修改值，但不能直接修改键。
2. 内部基于哈希表，**不保证遍历顺序，也不保持插入顺序**。单元素插入、按键查找和删除平均为 **O(1)**，最坏为 **O(N)**，不能将平均复杂度理解为每次操作的保证。
3. 常用成员函数（`N` 为当前元素数量；哈希、比较等单次操作按 O(1) 计）：

```cpp
um[key]                       // 返回值的引用；键不存在时插入值初始化的值，int 为 0
um.at(key)                    // 返回值的引用；键不存在时抛出 std::out_of_range，不插入
um.find(key)                  // 成功返回迭代器，否则返回 end()
um.count(key)                 // 键存在返回 1，否则返回 0
um.contains(key)              // 返回键是否存在，C++20
um.insert({key, value})       // 插入键值对，键已存在时不覆盖原值
um.emplace(key, value)        // 尝试原地构造并插入，键已存在时不覆盖原值
um.try_emplace(key, value)    // C++17；键已存在时不构造映射值
um.insert_or_assign(key, value) // C++17；键不存在则插入，存在则更新值
um.erase(key)                 // 返回删除数量（0 或 1）
um.erase(it)                  // 删除 it 指向的元素，返回下一个迭代器；it 不能是 end()
// 以上操作均平均 O(1)，最坏 O(N)
um.size() / um.empty()        // 元素数量 / 是否为空，O(1)
um.begin() / um.end()         // 正向遍历的起点 / 尾后迭代器，O(1)
um.clear()                    // 清空所有元素，O(N)
```

4. 初始化、插入与查询示例（C++11）：

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> um{{"Alice", 90}, {"Bob", 85}};

    um["Alice"] = 95;                    // 更新已有键的值
    ++um["Carol"];                        // 不存在时先插入 0，再递增为 1
    auto result = um.insert({"Alice", 100});
    std::cout << result.second << '\n';   // 0：插入失败，Alice 的值仍为 95

    auto it = um.find("Bob");
    if (it != um.end()) {                 // 解引用之前先检查是否找到
        std::cout << it->first << " " << it->second << '\n';
    }

    for (const auto& entry : um) {
        std::cout << entry.first << " " << entry.second << '\n';
    }                                    // 输出顺序不可依赖
}
```

5. **仅检查键是否存在时，使用 `find()`、`count()` 或 C++20 的 `contains()`**。不要用 `um[key]` 判断存在性：它可能插入元素，且值为 0 不代表键不存在。`const unordered_map` 不支持 `[]`，但支持 `at()`、`find()`、`count()` 等查询。


## 六. unordered_set

1. `std::unordered_set` 是 **C++11 起提供的无序集合**，头文件为 `#include <unordered_set>`。只存储键（元素本身就是键），**元素不能重复**，适合去重和快速判断某个值是否出现。
2. 同样基于哈希表，不保证遍历顺序。单元素插入、查找和删除平均 **O(1)**，最坏 **O(N)**。不能通过迭代器直接修改元素；需要改变元素时，可以先删除旧值，再插入新值。
3. 常用成员函数：

```cpp
us.insert(value)              // 返回 pair<iterator, bool>；second 表示是否插入成功
us.emplace(value)             // 尝试原地构造并插入，同样返回 pair<iterator, bool>
us.find(value)                // 成功返回迭代器，否则返回 end()
us.count(value)               // 存在返回 1，否则返回 0
us.contains(value)            // 返回元素是否存在，C++20
us.erase(value)               // 返回删除数量（0 或 1）
us.erase(it)                  // 删除 it 指向的元素，返回下一个迭代器；it 不能是 end()
// 以上操作均平均 O(1)，最坏 O(N)
us.size() / us.empty()        // 元素数量 / 是否为空，O(1)
us.begin() / us.end()         // 正向遍历的起点 / 尾后迭代器，O(1)
us.clear()                    // 清空所有元素，O(N)
```

4. 去重、查询及遍历时删除示例（C++11）：

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> us{3, 1, 2, 2};
    std::cout << us.size() << '\n';        // 3：重复的 2 只保留一个

    auto result = us.insert(3);
    std::cout << result.second << '\n';    // 0：3 已存在

    if (us.find(2) != us.end()) {
        std::cout << "2 exists\n";
    }

    for (auto it = us.begin(); it != us.end(); ) {
        if (*it % 2 == 0) {
            it = us.erase(it);            // 接收下一个迭代器，避免使用已失效的 it
        } else {
            ++it;
        }
    }
}
```

5. `unordered_set` **没有 `[]` 和 `at()`**，不能按下标访问元素。需要允许重复元素时，使用 `unordered_multiset`；键值对允许重复键时，使用 `unordered_multimap`。

### 两种无序容器的共同注意事项

1. **桶与负载因子**：元素按哈希值分配到桶（bucket）中，哈希冲突时还需要通过相等判断区分键。负载因子为 `size() / bucket_count()`。

```cpp
um.bucket_count()             // 当前桶的数量，O(1)
um.load_factor()              // 当前负载因子，O(1)
um.max_load_factor()          // 最大负载因子阈值，O(1)
um.max_load_factor(0.7f)      // 设置正数阈值；更低的阈值通常需要更多桶
um.reserve(1000)              // 按当前阈值为总共约 1000 个元素准备桶，并非插入元素
um.rehash(1000)               // 请求至少 1000 个桶，还需满足当前元素数量及负载要求
// reserve / rehash：平均 O(N)，最坏 O(N²)；unordered_set 也支持这些接口
```

   已知大致元素数量时，可以先设置 `max_load_factor()`，再调用 `reserve()`，减少后续插入时重新哈希的次数。`reserve(n)` 按元素数量估算桶数，`rehash(n)` 直接按桶数提出要求。

2. **迭代器失效**：重新哈希（rehash）会使所有迭代器失效，包括插入触发的自动扩容。重新哈希不使已有元素的引用和指针失效；删除元素则使指向被删元素的迭代器、引用和指针失效。调用可能重新哈希的操作后，应重新获取迭代器。
3. **键的要求**：默认使用 `std::hash<Key>` 和 `std::equal_to<Key>`，不需要键支持大小比较。自定义类型一般需要提供哈希函数和相等判断，**相等的键必须得到相同的哈希值**，不同键允许哈希冲突。自定义类型示例见 [[CPP容器与适配器初始化速查#5. 无序关联容器]]。
4. 与有序容器的选择：

| 对比项 | `map` / `set` | `unordered_map` / `unordered_set` |
|---|---|---|
| 底层结构 | 通常为红黑树 | 哈希表 |
| 遍历顺序 | 按比较器排序 | 不保证顺序 |
| 按键查找、普通单元素插入、按键删除 | O(log N) | 平均 O(1)，最坏 O(N) |
| 键的要求 | 满足比较器的严格弱序 | 可哈希且可判断相等 |
| 有序查找接口 | 支持 `lower_bound()` / `upper_bound()` | 不支持这两个成员函数 |
| 反向遍历接口 | 支持 `rbegin()` / `rend()` | 不支持，仅提供前向迭代器 |

   需要排序、按键范围查询时选择 `map` / `set`；只需快速查找、统计或去重时，可以考虑无序容器。无序容器并不保证在所有数据规模和哈希分布下都更快。

参考：[C++ 标准草案：无序关联容器要求](https://eel.is/c++draft/unord.req)、[`unordered_map`](https://eel.is/c++draft/unord.map)、[`unordered_set`](https://eel.is/c++draft/unord.set)。


## 七. string

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
	`const map` 和 `const unordered_map` 不能使用 `[]`，但可以使用 `at()`、`find()`、`count()`，以及 C++20 的 `contains()` 查询元素。

3. 在 C++11 后可以**使用 emplace_back 代替 push_back**，都是往容器末尾添加元素但底层实现机制不同。
     - **push_back()** 会先创建这个元素，然后再将这个元素拷贝（事后会自行销毁）或移动到容器中。
     - **emplace_back()** 则直接在容器尾部使用完美转发创建这个元素，在构造复杂对象时效率更优。
