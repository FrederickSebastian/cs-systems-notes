# C++ 容器与适配器初始化速查

> 默认使用 C++11 及以上标准；特别标注的写法需要更高版本。

## 1. 先记住这些通式

```cpp
#include <array>
#include <vector>
#include <deque>
#include <list>
#include <forward_list>
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>
#include <stack>
#include <queue>
#include <string>
#include <functional>    // less、greater、hash
#include <iterator>      // begin、end
#include <utility>       // move、pair
#include <ranges>        // C++23 的 from_range

using namespace std;
```

绝大多数标准容器都支持下列初始化方式：

```cpp
Container<T> c;                       // 1. 默认初始化：空容器
Container<T> c(other);                // 2. 拷贝初始化
Container<T> c(std::move(other));     // 3. 移动初始化；需 <utility>
Container<T> c{a, b, c};              // 4. 初始化列表
Container<T> c(first, last);          // 5. 迭代器区间 [first, last)
auto c = Container<T>{a, b, c};       // 6. 用 auto 接收
```

常见规律：

| 初始化需求 | 通常使用 |
|---|---|
| 创建空容器 | `C<T> c;` |
| 写死若干初值 | `C<T> c{...};` |
| 从另一容器复制 | `C<T> c(other);` |
| 从另一容器的部分区间构造 | `C<T> c(first, last);` |
| 创建 `n` 个元素 | 顺序容器常用 `C<T> c(n);` |
| 创建 `n` 个相同元素 | 顺序容器常用 `C<T> c(n, value);` |
| 自定义排序规则 | 有序关联容器、`priority_queue` 在类型中写比较器 |
| 自定义底层容器 | 容器适配器在第二个模板参数中指定 |

> `first, last` 表示左闭右开区间：包含 `first`，不包含 `last`。

---

## 2. 顺序容器

### 2.1 `array`：固定长度数组

```cpp
array<int, 5> a{};              // 5 个元素全部值初始化为 0
array<int, 5> b{1, 2, 3};       // {1, 2, 3, 0, 0}
array<int, 3> c = {1, 2, 3};
array d{1, 2, 3};               // C++17：推导为 array<int, 3>
array<int, 3> e = c;            // 拷贝
```

`array` 的长度是类型的一部分，必须在编译期确定：

```cpp
array<int, 3> a;
array<int, 4> b;                // 与 a 是不同类型
```

注意：

```cpp
array<int, 5> a;                // 局部变量中的基础类型元素值不确定
array<int, 5> b{};              // 推荐：元素全部为 0
```

### 2.2 `vector`：动态连续数组

```cpp
vector<int> a;                  // 空
vector<int> b(5);               // 5 个 0
vector<int> c(5, 8);            // 5 个 8
vector<int> d{1, 2, 3};         // 3 个元素：1, 2, 3
vector<int> e(d.begin(), d.end());
vector<int> f(d);               // 拷贝
vector<int> g(std::move(d));    // 移动
vector h{1, 2, 3};              // C++17：vector<int>
```

最容易混淆的一组：

```cpp
vector<int> a(5, 8);            // {8, 8, 8, 8, 8}
vector<int> b{5, 8};            // {5, 8}
```

### 2.3 `deque`：双端队列

初始化形式与 `vector` 基本相同：

```cpp
deque<int> a;
deque<int> b(5);                // 5 个 0
deque<int> c(5, 8);             // 5 个 8
deque<int> d{1, 2, 3};
deque<int> e(d.begin(), d.end());
deque f{1, 2, 3};               // C++17
```

### 2.4 `list`：双向链表

```cpp
list<int> a;
list<int> b(5);                 // 5 个 0
list<int> c(5, 8);              // 5 个 8
list<int> d{1, 2, 3};
list<int> e(d.begin(), d.end());
list f{1, 2, 3};                // C++17
```

### 2.5 `forward_list`：单向链表

```cpp
forward_list<int> a;
forward_list<int> b(5);         // 5 个 0
forward_list<int> c(5, 8);      // 5 个 8
forward_list<int> d{1, 2, 3};
forward_list<int> e(d.begin(), d.end());
forward_list f{1, 2, 3};        // C++17
```

### 顺序容器初始化通式

除 `array` 外，下面几种容器的构造方式高度一致：

```cpp
vector<T>
deque<T>
list<T>
forward_list<T>
```

可统一记成：

```cpp
C<T> a;                         // 空
C<T> b(n);                      // n 个值初始化的 T
C<T> c(n, value);               // n 个 value
C<T> d{v1, v2, v3};             // 指定元素
C<T> e(first, last);            // 来自迭代器区间
```

---

## 3. 字符串 `string`

`string` 是 `basic_string<char>` 的别名，初始化方式与顺序容器很相似：

```cpp
string a;                       // 空串
string b = "hello";
string c("hello");
string d(5, 'x');               // "xxxxx"
string e{'h', 'e', 'l', 'l', 'o'};
string f(b.begin(), b.end());
string g(b, 1, 3);              // 从 b[1] 开始取 3 个字符："ell"
```

不要写：

```cpp
string s(5, "x");              // 错误：第二个参数应是 char，不是字符串
```

---

## 4. 有序关联容器

底层通常按比较器维护有序结构。

| 容器 | 内容 | 键能否重复 |
|---|---|---|
| `set<T>` | 只有键 | 否 |
| `multiset<T>` | 只有键 | 是 |
| `map<K, V>` | 键值对 | 否 |
| `multimap<K, V>` | 键值对 | 是 |

### 4.1 `set` 与 `multiset`

```cpp
set<int> a;                     // 空，默认升序
set<int> b{3, 1, 2, 2};         // {1, 2, 3}，重复值被忽略
set<int> c(b.begin(), b.end());

multiset<int> d{3, 1, 2, 2};    // {1, 2, 2, 3}，保留重复值
```

降序：

```cpp
set<int, greater<int>> a{3, 1, 2};
multiset<int, greater<int>> b{3, 1, 2, 2};
```

自定义比较器：

```cpp
struct Compare {
    bool operator()(int a, int b) const {
        return a > b;           // 降序
    }
};

set<int, Compare> s{3, 1, 2};
```

### 4.2 `map` 与 `multimap`

每个元素都是 `{key, value}`：

```cpp
map<string, int> a;
map<string, int> b{
    {"Alice", 90},
    {"Bob",   85}
};

map<string, int> c(b.begin(), b.end());

multimap<string, int> d{
    {"Math", 90},
    {"Math", 80}               // 允许相同键
};
```

也可以显式写 `pair`：

```cpp
map<string, int> m{
    make_pair("Alice", 90),
    pair<string, int>{"Bob", 85}
};
```

按键降序：

```cpp
map<int, string, greater<int>> m{
    {1, "one"},
    {2, "two"}
};
```

### 有序关联容器初始化通式

```cpp
set<Key, Compare> s{keys...};
multiset<Key, Compare> ms{keys...};

map<Key, Value, Compare> m{
    {key1, value1},
    {key2, value2}
};
```

默认比较器是 `less<Key>`，因此默认按键升序。

---

## 5. 无序关联容器

底层使用哈希表，不保证遍历顺序。

| 容器 | 内容 | 键能否重复 |
|---|---|---|
| `unordered_set<T>` | 只有键 | 否 |
| `unordered_multiset<T>` | 只有键 | 是 |
| `unordered_map<K, V>` | 键值对 | 否 |
| `unordered_multimap<K, V>` | 键值对 | 是 |

```cpp
unordered_set<int> a;
unordered_set<int> b{3, 1, 2, 2};
unordered_multiset<int> c{3, 1, 2, 2};

unordered_map<string, int> d{
    {"Alice", 90},
    {"Bob",   85}
};

unordered_multimap<string, int> e{
    {"Math", 90},
    {"Math", 80}
};
```

从区间初始化：

```cpp
unordered_set<int> s(source.begin(), source.end());
unordered_map<string, int> m(source.begin(), source.end());
```

自定义类型作为键时，通常要提供哈希函数和相等判断：

```cpp
struct Key {
    int x;
};

struct KeyHash {
    size_t operator()(const Key& k) const {
        return hash<int>{}(k.x);
    }
};

struct KeyEqual {
    bool operator()(const Key& a, const Key& b) const {
        return a.x == b.x;
    }
};

unordered_set<Key, KeyHash, KeyEqual> s;
```

### 无序关联容器初始化通式

```cpp
unordered_set<Key, Hash, KeyEqual> s{keys...};
unordered_map<Key, Value, Hash, KeyEqual> m{
    {key1, value1},
    {key2, value2}
};
```

与有序关联容器的核心区别：

- 有序容器指定 `Compare`；
- 无序容器指定 `Hash` 和 `KeyEqual`；
- 二者都可使用初始化列表、迭代器区间、拷贝和移动构造。

---

## 6. 容器适配器

容器适配器不直接管理一种全新的存储结构，而是限制底层容器的操作接口。

| 适配器 | 默认底层容器 | 可用的常见底层容器 |
|---|---|---|
| `stack<T>` | `deque<T>` | `deque`、`vector`、`list` |
| `queue<T>` | `deque<T>` | `deque`、`list` |
| `priority_queue<T>` | `vector<T>` | `vector`、`deque` |

> 能否作为底层容器，取决于适配器所需操作。例如 `queue` 需要前端删除，因此不能使用没有 `pop_front()` 的 `vector`。

### 6.1 `stack`：栈，后进先出

```cpp
stack<int> a;                   // 空栈，底层默认是 deque<int>

deque<int> d{1, 2, 3};
stack<int> b(d);                // 拷贝底层容器；栈顶为 3
stack<int> c(std::move(d));     // 移动底层容器
```

指定底层容器：

```cpp
stack<int, vector<int>> a;
stack<int, list<int>> b;

vector<int> v{1, 2, 3};
stack<int, vector<int>> c(v);
```

C++23 可直接使用范围构造：

```cpp
stack<int> s(from_range, source);   // C++23
```

### 6.2 `queue`：队列，先进先出

```cpp
queue<int> a;                   // 空队列，底层默认是 deque<int>

deque<int> d{1, 2, 3};
queue<int> b(d);                // 队首为 1，队尾为 3
queue<int> c(std::move(d));
```

指定底层容器：

```cpp
queue<int, list<int>> a;

list<int> values{1, 2, 3};
queue<int, list<int>> b(values);
```

C++23：

```cpp
queue<int> q(from_range, source);   // C++23
```

### 6.3 `priority_queue`：优先队列

默认是大顶堆，最大元素位于 `top()`：

```cpp
priority_queue<int> a;

vector<int> v{3, 1, 4, 2};
priority_queue<int> b(less<int>{}, v);   // 拷贝 v，top() == 4

priority_queue<int> c(v.begin(), v.end());
```

小顶堆，最小元素位于 `top()`：

```cpp
priority_queue<int, vector<int>, greater<int>> minHeap;

vector<int> v{3, 1, 4, 2};
priority_queue<int, vector<int>, greater<int>> minHeap2(
    greater<int>{}, v
);

priority_queue<int, vector<int>, greater<int>> minHeap3(
    v.begin(), v.end()
);
```

自定义比较器：

```cpp
struct Compare {
    bool operator()(int a, int b) const {
        return a > b;           // 小顶堆效果
    }
};

priority_queue<int, vector<int>, Compare> pq;
```

C++23：

```cpp
priority_queue<int> pq(from_range, source);   // C++23
```

### 适配器初始化通式

```cpp
Adapter<T> a;                               // 使用默认底层容器
Adapter<T, Container<T>> b(container);      // 从底层容器拷贝
Adapter<T, Container<T>> c(move(container));// 从底层容器移动
```

`priority_queue` 多一个比较器参数：

```cpp
priority_queue<T, Container<T>, Compare> pq;
```

特别注意：在 C++11～C++20 中，`stack` 和 `queue` 通常不能像 `vector` 那样直接写：

```cpp
stack<int> s{1, 2, 3};          // 通常不成立
queue<int> q{1, 2, 3};          // 通常不成立
```

应先初始化底层容器，再构造适配器，或逐个 `push()`。

---

## 7. 从普通数组初始化

```cpp
int a[] = {1, 2, 3, 4};

vector<int> v(begin(a), end(a));
list<int> l(begin(a), end(a));
set<int> s(begin(a), end(a));
unordered_set<int> us(begin(a), end(a));
priority_queue<int> pq(begin(a), end(a));
```

也可使用指针区间：

```cpp
vector<int> v(a, a + 4);
```

---

## 8. C++17 类型推导（CTAD）

C++17 起，很多情况下可以省略模板实参：

```cpp
vector v{1, 2, 3};              // vector<int>
deque d{1.0, 2.0};              // deque<double>
list l{'a', 'b'};               // list<char>
set s{3, 1, 2};                 // set<int>
map m{
    pair{"Alice", 90},
    pair{"Bob", 85}
};
```

但显式写出类型通常更清楚，也可避免推导结果不符合预期。

---

## 9. 高频易错点

### 圆括号与花括号含义不同

```cpp
vector<int> a(3, 5);            // {5, 5, 5}
vector<int> b{3, 5};            // {3, 5}

list<int> c(3, 5);              // {5, 5, 5}
list<int> d{3, 5};              // {3, 5}
```

### `map` 的每个初值必须是键值对

```cpp
map<string, int> ok{{"A", 1}, {"B", 2}};
// map<string, int> bad{"A", 1, "B", 2};   // 错误
```

### `set` 会自动去重，`multiset` 不会

```cpp
set<int> a{1, 1, 2};            // 1, 2
multiset<int> b{1, 1, 2};       // 1, 1, 2
```

### 无序容器的输出顺序不可依赖

```cpp
unordered_set<int> s{1, 2, 3};  // 遍历顺序不一定是 1, 2, 3
```

### 比较器的含义

```cpp
set<int, less<int>> s;          // 小到大
set<int, greater<int>> s2;      // 大到小

priority_queue<int, vector<int>, less<int>> p1;     // 大顶堆
priority_queue<int, vector<int>, greater<int>> p2;  // 小顶堆
```

`priority_queue` 的比较器直觉容易与 `set` 混淆，直接记结论最稳妥：

- `less<T>`：大顶堆；
- `greater<T>`：小顶堆。

---

## 10. 一页式总表

| 类别 | 容器 | 最常用初始化 |
|---|---|---|
| 固定顺序容器 | `array<T, N>` | `array<T, N> a{v1, v2};` |
| 动态顺序容器 | `vector<T>` | `vector<T> v(n, value);`、`vector<T> v{...};` |
| 双端顺序容器 | `deque<T>` | `deque<T> d(n, value);`、`deque<T> d{...};` |
| 链式顺序容器 | `list<T>` | `list<T> l(n, value);`、`list<T> l{...};` |
| 单链顺序容器 | `forward_list<T>` | `forward_list<T> l(n, value);`、`forward_list<T> l{...};` |
| 字符串 | `string` | `string s = "text";`、`string s(n, ch);` |
| 有序集合 | `set<T>` / `multiset<T>` | `set<T> s{...};` |
| 有序映射 | `map<K,V>` / `multimap<K,V>` | `map<K,V> m{{k1,v1},{k2,v2}};` |
| 哈希集合 | `unordered_set<T>` 等 | `unordered_set<T> s{...};` |
| 哈希映射 | `unordered_map<K,V>` 等 | `unordered_map<K,V> m{{k1,v1},{k2,v2}};` |
| 栈适配器 | `stack<T>` | `stack<T> s;` 或从底层容器构造 |
| 队列适配器 | `queue<T>` | `queue<T> q;` 或从底层容器构造 |
| 优先队列 | `priority_queue<T>` | `priority_queue<T> pq;` 或用迭代器区间构造 |

最终可压缩为三条记忆：

```cpp
C<T> c{elements...};                 // 给出具体元素
C<T> c(first, last);                 // 从一个区间取得元素
C<T> c(n, value);                    // 顺序容器：n 个相同元素
```

而映射容器的元素要写成键值对：

```cpp
Map<Key, Value> m{{key1, value1}, {key2, value2}};
```

容器适配器则优先记住：

```cpp
stack<T> s;
queue<T> q;
priority_queue<T> pq;                // 默认大顶堆
priority_queue<T, vector<T>, greater<T>> minHeap;
```
