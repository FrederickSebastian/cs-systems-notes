
## 概述

适配器 Adapters (也叫包装器, wrapper) 是一种**设计模式**，用于**封装现有的容器、迭代器或函数对象，以提供新的接口或行为**，使得某些组件能够在不同的上下文中使用，或者提供更加通用的接口。在 STL 中，适配器通常用于提供特定的容器接口，如栈（stack）、队列（queue）和优先队列（priority_queue）。

在 C++STL 中适配器分为**容器适配器**、**函数适配器**、和**迭代器适配器**共三类。函数对象适配器就是上面适配器仿函数中介绍的bind、function等这些适配器，用于修改、扩展或重新定制现有的函数对象或函数指针的行为。

---

## 一. 容器适配器

### std::stack
只能访问栈顶

### std::queue
先进先出FIFO, 只能访问队首和队尾, 不能遍历

### std::priority_queue
以下详细介绍`std::priority_queue`: 基于堆(heap)的受限容器适配器, 用于高效维护"当前最重要的元素", 

>**只保证 队首元素是优先级最高的, 不保证其他元素的顺序**

它不是排序容器, 而是**随时取最大/最小值的数据结构**
1. 底层容器一般是`std::vector<T>`, 要求支持random access iterator, push_back & pop_back. 
2. 常用函数
	- top(), 时间复杂度O(1)
	- push(ele), O(log N)
	- pop(), O(log N)
	- 构造, O(N)
3. **常见误区**: priority_queue内置默认的Compare是less, 表示**值小的优先级低**, 故这时pop出来时最大的值.
4. 构造时可以添加第三个参数如greater, 此时最小的值在顶端
5. 自定义比较:
	- `pair` 作为参数时, first将会被比较, 按照first大的优先, 如果first相等, 比较second
	- 自定义类/结构体 + 比较器
	```cpp
	struct Node {
	    int cost;
	    int id;
	};

	struct Cmp {
	    bool operator()(const Node& a, const Node& b) const {
	        return a.cost > b.cost; // cost 小的优先
	    }
	};

	std::priority_queue<Node, std::vector<Node>, Cmp> pq;
	```
	- **compare(a, b) == true 表示a的优先级低于b**
6. 常见误区: 
	- 不能遍历priority_queue
	- 不能sort排序该容器

---

## 二. 函数适配器
在不改变被调用对象本身的前提下, 改变其调用接口或者调用行为

### std::bind
参数绑定适配器: 把一个可调用对象的一部分参数预先绑定, 生成一个新的可调用对象:
```cpp
#include <functional>

int add(int a, int b) {
    return a + b;
}

auto add10 = std::bind(add, 10, std::placeholders::_1);
int x = add10(5); // 15
```

### std::men_fn
成员函数适配器: 把成员函数指针适配为一个普通可调用对象:
```cpp
#include <functional>

struct Foo {
    void print(int x) const { /* ... */ }
};

auto f = std::mem_fn(&Foo::print);  // 适配
Foo obj;
f(obj, 42);        // 等价于 obj.print(42)
```
常见应用: 塞给for_each
`std::for_each(vec.begin(), vec.end(),std::mem_fn(&Foo::print));`

### std::reference_wrapper
把值转换为引用传递. 适配STL. 具体使用时常与bind配合. 详细介绍见最后. 

### std::function
function 是一个泛型函数封装器，可以**封装任意可调用对象**，包括函数指针、函数对象、成员函数指针等。

```cpp
int add(int a, int b) {
    return a + b;
}

int main() {
    std::function<int(int, int)> f = add;
    std::cout << f(2, 3) << std::endl; // 输出 5

    return 0;
}
```
用function封装模板可以改善其在不同场景下的低效性

### std::unary_negate & std::binary_negate
对一元和二元谓词取反
```cpp
bool isEven(int x) {
    return x % 2 == 0;
}

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::for_each(v.begin(), v.end(), [](int x) {
        std::cout << std::unary_negate<bool>(isEven)(x) << ' ';
    });
    // 输出 1 0 1 0 1

    return 0;
}
```


### std::logical_not
对一元谓词取逻辑非:
```cpp
bool isEven(int x) {
    return x % 2 == 0;
}

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::for_each(v.begin(), v.end(), [](int x) {
        std::cout << std::logical_not<bool>(isEven(x)) << ' ';
    });
    // 输出 1 0 1 0 1

    return 0;
}
```


---

## 三. 迭代器适配器
修改迭代器的行为

### Reverse Iterator
反向迭代器用于逆序遍历容器。它通过重新定义递增运算符（++）和递减运算符（--），使得迭代器可以逆序访问容器中的元素。

```cpp
int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};

    // 使用反向迭代器逆序遍历
    for (auto it = v.rbegin(); it != v.rend(); ++it) {
        std::cout << *it << " ";    // 输出 5 4 3 2 1
    }

    return 0;
}
```

### Insert Iterator
插入迭代器用于在容器的特定位置插入新元素。C++ STL 提供了三种插入迭代器：

- std::back_insert_iterator：在容器的尾部插入元素。
- std::front_insert_iterator：在容器的头部插入元素。
- std::insert_iterator：在容器的指定位置插入元素。

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> v = {1, 2, 3};

    // 使用 back_insert_iterator 在尾部插入元素
    std::back_insert_iterator<std::vector<int>> back_it(v);
    *back_it = 4;

    // 使用 front_insert_iterator 在头部插入元素
    std::front_insert_iterator<std::vector<int>> front_it(v);
    *front_it = 0;

    // 使用 insert_iterator 在指定位置插入元素
    std::insert_iterator<std::vector<int>> insert_it(v, v.begin() + 2);
    *insert_it = 5;

    for (int i : v) {
        std::cout << i << " ";
    }
    // 输出 0 1 5 2 3 4

    return 0;
}
```


### Stream Iterator
流迭代器用于从输入流读取数据或将数据写入输出流。C++ STL 提供了两种流迭代器：

- std::istream_iterator：用于从输入流（如 std::cin）读取数据。
- std::ostream_iterator：用于将数据写入输出流（如 std::cout）。

```cpp
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    // 使用 istream_iterator 从 cin 读取数据
    std::istream_iterator<int> in_iter(std::cin);
    std::istream_iterator<int> end_iter;   // 默认构造，用于构造容器时定义范围的结束
    std::vector<int> v(in_iter, end_iter); // 从 in_iter 开始读取数据，直到遇到 end_iter

    // 使用 ostream_iterator 将数据写入 cout
    std::ostream_iterator<int> out_iter(std::cout, " "); // 每个输出的整数之间插入一个空格作为分隔符
    std::copy(v.begin(), v.end(), out_iter); // 将 v 中的元素复制到 std::cout

    return 0;
}
```

>关于流迭代器还有很多技巧, 这里不再赘述.


### Move Iterator
C++11 引入的移动迭代器 std::move_iterator 可将某个范围的元素移动到目标范围，而不需要通过拷贝。
```cpp
#include <iostream>
#include <vector>
#include <iterator>
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    std::vector<int> v2;

    // 使用 move_iterator 将 v 中的元素移动到 v2
    std::move_iterator<std::vector<int>::iterator> move_it(v.begin());
    v2.insert(v2.end(), move_it, move_it + v.size());

    for (int i : v2) {
        std::cout << i << " ";
    }
    // 输出 1 2 3 4 5

    return 0;
}
```

---

## 以下是对std::bind() & std::regerence_wrapper的详细介绍

### 一、`std::bind` 的本质（一句话）

> **`std::bind` 不是“传参”，而是“生成一个带调用规则的函数对象”。**


### 二、`std::bind` 的三类参数（必须记住）

在

`std::bind(f, a1, a2, ..., aN)`

中，每个 `a_i` 只可能是下面三种之一：

#### 1.  普通值（value）

`std::bind(f, 10)`

- **含义**：调用时始终使用 `10`
    
- **行为**：`bind` 时拷贝 / 移动保存
    
- **关键词**：值语义（拍快照）
    

---

#### 2️. 占位符（`std::placeholders::_k`）

`std::bind(f, _2, _1)`

- **含义**：调用时用第 `k` 个实参替换
    
- **行为**：参数重排 / 延迟绑定
    
- **注意**：
    
    - `_k` 只关心“位置”
        
    - 不保存值
        
    - 每次调用都会重新替换
        

---

#### 3️. `std::reference_wrapper`（`std::ref / std::cref`）

`std::bind(f, std::ref(x))`

- **含义**：调用时传 `x` 的引用
    
- **行为**：保存 `&x`，而不是副本
    
- **关键词**：引用语义
    

---

### 三、调用期发生了什么（机械执行规则）

`auto g = std::bind(f, a1, a2, ..., aN); g(b1, b2, ..., bM);`

调用 `g(...)` 时：

1. 从左到右处理 `a1 ... aN`
    
2. 对每个参数：
    
    - 普通值 → 直接用
        
    - `_k` → 替换为 `b_k`
        
    - `std::ref(x)` → 解引用成 `x`
        
3. 用替换后的参数调用 `f(...)`
    

---

### 四、`std::reference_wrapper` 是什么（一句话）

> **一个“能被拷贝的引用”。**

---

### 五、为什么必须有 `reference_wrapper`

### 语言事实

- 引用不能拷贝
    
- 引用不能放进容器
    
- `std::bind` / STL 算法 **必须存对象**
    

### 解决方案

`std::ref(x)  // 存的是 &x，用起来像 x&`

等价心智模型：

`reference_wrapper<T> ≈ T* 但会自动转成 T&`

---

### 六、最重要的对照例子（必背）

### ❌ 默认是值语义

`int x = 10; auto f = std::bind(inc, x);`

等价于：

`inc(10);`

---

### 正确使用引用语义

`auto f = std::bind(inc, std::ref(x));`

等价于：

`inc(x);`

---

### 七、占位符的终极理解（关键句）

> **`_k` 的含义是：  
> “调用我时，把第 k 个实参原封不动地放在这里。”**

---

### 八、综合例（模板级理解）

`int x = 1;  auto f = std::bind(     [](int& a, int b){ a += b; },     std::ref(x),     _1 );  f(5);   // x += 5 f(10);  // x += 10`

最终：

`x == 16`

---

### 九、一个实用判断口诀

> **在 `bind` 里看到一个名字：  
> 没包 `std::ref` → 它是副本  
> 包了 `std::ref` → 它是原对象**

---

### 十、现实建议

- 能用 lambda → **优先 lambda**
    
- 学 `bind` 的目的：
    
    - 读旧代码
        
    - 理解 STL 的“值 / 引用 / 调用适配”设计哲学