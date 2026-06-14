

## 一、自动类型推断

1. `auto`：让编译器根据初始化表达式自动推断类型。
    
    ```cpp
    auto x = 10;       // int  
    auto y = 3.14;     // double  
    auto s = "Hi"s;    // std::string (C++14起)
    ```
    
     推荐：减少重复书写复杂类型，尤其适合迭代器、模板。
    
2. `decltype(expr)`：获取表达式类型（但不求值）。
    
    ```cpp
    decltype(x + y) z;  // 推断z类型与x+y相同
    ```
    Tips: 在定义模板时很有用, 可以在实例化时确定类型
    ```C++
    template <typename T, typename U>
    void ef(T t, U u)
    {
	    decltype(T * U) tu;
    }
    ```

1. C++11 创建新的类型别名的语法:
	`using  itType = std::vector<std::string>::iterator;`
	尽可能多使用`using`语法而不是C的`typedef`.

---

## 二、范围for与初始化改进

3. 范围for循环（Range-based for）：
    
    ```cpp
    for (auto& val : vec) 
        cout << val << " ";
    ```
    
4. 统一初始化（C++11）：
    
    ```cpp
    int a{10};
    vector<int> v{1, 2, 3, 4};
    ```
    

---

## 三、右值引用与移动语义

5. 右值引用：`T&&`，用于接受临时对象。
    **Caution: 不能获取右值引用的地址** 
    ```cpp
    string s1 = "hello";
    string s2 = std::move(s1);  // s1资源转移给s2
    ```
    
    - 提升性能，避免拷贝。
        
    - 头文件`utility`中的函数 `std::move()` 表示“我不再需要这对象的资源”, 可以强制使用移动语义: 
	    `four = std::move(one);  // forced move assignment`
        
6. 移动构造与移动赋值：
    
    ```cpp
    MyClass(MyClass&& other);  
    MyClass& operator=(MyClass&& other);
    ```
    需要注意的是在移动以后要把原指针设为nullptr. 

---

## 四、Lambda表达式

7. Lambda是匿名函数，可在局部定义并立即使用：
    
    ```cpp
    auto f = [](int x, int y) { return x + y; };
    cout << f(3, 4);  // 输出7
    ```
    
8. 捕获列表：
    
    ```cpp
    int a = 10, b = 20;
    auto f = [=]() { return a + b; };     // 捕获外部变量值
    auto g = [&]() { b++; };              // 捕获引用
    auto h = [a](int x){ return a + x; }; // 捕获部分
    ```
    
9. Lambda常用于STL算法：
    
    ```cpp
    sort(v.begin(), v.end(), [](int a, int b){ return a > b; });
    ```
    
7. **Caution:** 仅当lambda表达式**完全**由**一条返回语句**组成时, 自动类型推断才会生效, 否则要使用返回类型后置语法: 
	`[](double x)->double{int y = x; return x - y;}`

8. 可以给Lambda指定名称, 用auto类型推断, 以便多次使用:
	`auto mod3 = [](int x){return x % 3 = 0;}`

9. 修改中括号`[]`中的内容可以控制lambda访问动态变量:
	- `[apple]` 按值访问变量apple.
	- `[&pig]` 按引用访问变量pig
	- `[&]` 按引用访问作用域内所有的动态变量
	- `[=]` 按值访问作用域内所有动态变量
	-  还可以混合使用他们, 例如 `[&, ted]` 表示按引用访问除ted以外的所有变量, 然后按值访问ted.

---

## 五、智能指针回顾

10. `unique_ptr` 独占资源，`shared_ptr` 共享资源，`weak_ptr` 弱引用防循环。
    
    ```cpp
    auto p = make_shared<int>(42);
    weak_ptr<int> wp = p;
    ```
    

---

## 六、可变参数模板（Variadic Templates）

11. 可处理任意数量参数的模板：
    

```cpp
template<typename... Args>
void print(Args... args) {
    (cout << ... << args) << endl;  // C++17折叠表达式
}
print("A", 2, 3.5); // 输出 A23.5
```

---

## 七、constexpr 与 编译期计算

12. `constexpr`：让表达式在编译期求值。
    

```cpp
constexpr int sq(int x) { return x * x; }
int arr[sq(3)];  // 合法，编译期计算
```

---

## 八、智能枚举与结构绑定

13. 强类型枚举（C++11）：
    

```cpp
enum class Color { Red, Green, Blue };
Color c = Color::Red;
```

14. 结构化绑定（C++17）：
    

```cpp
auto [x, y] = make_pair(3, 4);
```

---

## 九、标准库扩展

15. 新增容器与工具：
    

- `unordered_map`, `unordered_set`（哈希表）
    
- `tuple` 元组：`auto [x, y, z] = t;`
    
- `chrono`：时间/日期库
    
- `thread` / `mutex`：多线程支持
    
- `filesystem` (C++17)：文件操作接口
    

16. 随机数库：
    

```cpp
#include <random>
std::mt19937 gen(std::random_device{}());
std::uniform_int_distribution<> dis(1, 6);
int dice = dis(gen);
```

---

## 十、异常与安全改进

17. `noexcept`：标记函数不会抛异常。
    

```cpp
void func() noexcept;
```

18. `nullptr`：替代旧的 `NULL`，类型安全。
    
19. `static_assert`：编译期断言。
    

```cpp
static_assert(sizeof(int) == 4, "int size error");
```

---

## 十一. 类的构造函数的改进

1. 委托构造函数允许在另一个构造函数里使用已有的构造函数

2. 继承构造函数能让派生类继承基类的构造函数但仅限于该名称空间.

---


##  Summary

- **auto / decltype**：类型自动推断。
    
- **move / &&**：实现高效的移动语义。
    
- **Lambda**：让函数式编程在C++中成为主流。
    
- **constexpr**：把更多计算提前到编译期完成。
    
- **强枚举、结构绑定、可变参数模板**：增强语言表达力。
    
- **unordered_map、thread、filesystem** 等库使C++更现代化。
    
- **RAII + 智能指针 + noexcept** → 安全高效C++核心组合。