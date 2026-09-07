
### 1. 核心区别

```cpp
push_back(T);        // 传入一个已经构造好的 T
emplace_back(args);  // 传入构造 T 所需的参数
```

- `push_back`：将已有对象拷贝/移动进容器。
    
- `emplace_back`：通过完美转发，直接在容器内部构造对象。
    
- `emplace_back` 在传“构造参数”时可能减少临时对象和一次移动。
    

```cpp
vector<Person> v;

v.push_back(Person("Tom", 20)); // 先构造 Person，再移动
v.emplace_back("Tom", 20);      // 直接构造 Person
```

如果已经有对象：

```cpp
Person p("Tom", 20);

v.push_back(std::move(p));
v.emplace_back(std::move(p));
```

二者通常没有明显性能区别。

---

### 2. 为什么 `emplace_back({ ... })` 经常报错？

```cpp
vector<pair<int, int>> v;

v.push_back({1, 2});      // OK
v.emplace_back({1, 2});   // ERROR
v.emplace_back(1, 2);     // OK
```

原因：

`emplace_back` 本质是可变参数模板：

```cpp
template<class... Args>
emplace_back(Args&&... args);
```

调用时需要从实参推导 `Args...`。

而：

```cpp
{1, 2}
```

是 **braced-init-list（花括号初始化列表）**，它本身没有普通表达式类型，因此通常无法参与模板参数推导。

`push_back` 可以，是因为它的参数类型已经确定为容器元素类型 `T`：

```cpp
void push_back(T&&);
```

所以编译器知道 `{1, 2}` 应该用于构造 `T`。

一句话：

> `emplace_back` 支持模板类型推导；真正无法推导的是没有普通类型的 `{...}`。

---

## 3. 重点坑：`emplace_back` 可能出现意外构造行为

必须记住：

> `emplace_back(args...)` 的本质是直接调用元素类型的某个构造函数 `T(args...)`。

例如：

```cpp
vector<vector<int>> v;

v.emplace_back(10, 20);
```

这里不是插入：

```cpp
{10, 20}
```

实际调用的是：

```cpp
vector<int>(10, 20);
```

结果：

```cpp
{20, 20, 20, 20, 20, 20, 20, 20, 20, 20}
```

即创建 **10 个值为 20 的元素**。

因此使用 `emplace_back` 时必须确认：

```cpp
T(args...)
```

到底会匹配哪个构造函数。

尤其当类型存在：

- 多个重载构造函数
    
- `initializer_list` 构造函数
    
- 参数数量相近的构造函数
    

时，更容易出现和预期不同的行为。

---

## 面试版回答

> `push_back` 接收一个已经构造好的对象，然后拷贝或移动到容器中；`emplace_back` 接收构造对象所需的参数，通过完美转发直接在容器内部构造，因此可能减少临时对象。
> 
> 如果对象本身已经存在，`push_back(std::move(obj))` 和 `emplace_back(std::move(obj))` 通常区别不大。
> 
> `emplace_back({1,2})` 经常失败，是因为 `{1,2}` 是 braced-init-list，没有普通表达式类型，无法用于 `Args...` 的模板参数推导。
> 
> 使用 `emplace_back` 还要特别注意构造函数匹配，因为它本质上执行的是 `T(args...)`，可能调用到和预期不同的构造函数。