

## 一、友元类与友元成员

1. 将一个类声明为友元类，则该类的**所有成员函数**都可以访问原始类的私有成员与保护成员。
    
2. 语法示例：
    
    `friend class Remote;`
    
3. 若仅希望另一个类的**特定成员函数**成为友元方法，需要注意类声明顺序：
    
    - **声明阶段**：若使用对方类的指针或引用，需**前向声明**。
        
    - **定义阶段**：若访问对方类的成员，必须在前面有完整类定义。
        

---

## 二、异常处理（Exception Handling）

4. C++ 异常机制提供了将控制权从程序一部分转移到另一部分的途径。  
    三要素：
    
    - 引发异常（`throw`）
        
    - 捕获异常（`catch`）
        
    - 保护代码块（`try`）
        
5. 异常处理流程：
    
    - `try` 块中放置可能出错的代码。
        
    - `throw` 抛出异常对象。
        
    - `catch` 块捕获并处理异常。
        
6. `throw` 类似于 `return`，但会终止函数并沿调用栈向上查找匹配的 `catch`。
    
    `throw value;  // 抛出异常 catch(Type e) { ... }  // 捕获异常`
    
7. 当异常沿调用链传播时，C++ 会自动销毁已构造的局部对象（自动调用析构函数），确保资源释放安全。
    
8. 若存在异常类继承体系，应当**先捕获派生类异常，再捕获基类异常**。  
    → 从“最具体”到“最通用”的顺序。
    

---

## 三、异常兜底技巧（Exception Safety Tips）

9. a. 捕获未知异常：
    
```cpp
try 
{     
riskyFunc(); 
} 
catch (...) 
{     
std::cout << "未知异常被捕获" << std::endl; 
}
```


**即用省略号来表示未知的异常类型**
    b. 捕获所有未处理异常（全局兜底）：


```cpp
#include <exception> 
#include <iostream>  
void myTerminate() {     
	std::cout << "未捕获异常！程序终止！" << std::endl;     
	std::abort(); 
	}  
int main() 
{     
	std::set_terminate(myTerminate);     
	throw 1;  // 无catch，将触发terminate 
}
```

    ⚠️ 注意：`std::set_terminate()` 设置的终止处理函数只能用于兜底，  
    一旦触发，程序将终止，无法恢复。仅用于调试或日志用途。
    

---

## 四、RTTI（Run-Time Type Information）

10. RTTI 让程序在**运行时识别对象的真实类型**，主要用于多态体系。  
    仅在类中含有虚函数（多态类）时才有效。  
    常用操作：
    
    - `typeid(obj).name()`
        
    - `dynamic_cast<T*>(ptr)`
        

---

## 五、类型转换运算符

11. C++ 提供四种类型安全的转换运算符：
    

- `static_cast<T>(expr)`：**编译期类型转换**（最常用）
    
- `dynamic_cast<T>(expr)`：运行期检查多态类型安全转换
    
- `const_cast<T>(expr)`：去除/添加 `const` 限定
    
- `reinterpret_cast<T>(expr)`：底层强制转换（危险）
    

示例：

`double x = 3.14; int n = static_cast<int>(x);  // 安全转换 double → int`

---

## 六. Exception类

12. 头文件`<excetion>`定义了exception类, 可以将其用作基类. 
13. 该类包含一个`what()`的虚拟成员函数, 返回一个字符串, 内容随实现而异. 可以在exception的派生类中重新定义它. 其返回类型为`const char *`
14. 头文件`stdexcept`类定义了几个常用的异常类. 该文件定义了`logic_error`和`runtime_error`. 由这两个类派生出来的如下类:
	- `domain_error`定义域错误
	- `invalid_argument`
	- `length_error`
	- `out_of_bounds`
	每个类都有一个类似于`logic_error`的构造函数, 可以提供一份`what()`返回的字符串, 我们还有:
	- `range_error`
	- `overflow_error`
	- `underflow_error`
	他们是后者派生出来的. 我们可以从这两个类中派生出来适合我们的类.

15. `bad_alloc`异常在头文件`new`中, 用来处理new的内存分配问题


## Summary

- 友元类可访问另一个类的私有与保护成员；声明友元需注意类顺序。
    
- C++ 异常机制通过 `try`、`throw`、`catch` 实现控制流转移。
    
- 捕获顺序：先派生类再基类；`catch(...)` 可兜底未知异常。
    
- `std::set_terminate()` 用于全局异常兜底（慎用）。
    
- RTTI 用于运行期类型识别，仅适用于多态类。
    
- 推荐使用 `static_cast` 等安全转换运算符，避免传统 C 风格强制转换。
    

---

``` C++
#include <iostream>
#include <stdexcept>  // 各种标准异常类
#include <new>        // std::bad_alloc
#include <vector>

using namespace std;

int main() {
    try {
        cout << " 测试 new 分配失败..." << endl;
        double* bigArray = new double[1000000000000000ULL];
        delete[] bigArray;
    } catch (const bad_alloc& e) {
        cout << "捕获异常: bad_alloc -> " << e.what() << endl;
    }

    try {
        cout << "\n 测试数组越界..." << endl;
        vector<int> v = {1, 2, 3};
        cout << v.at(5);
    } catch (const out_of_range& e) {
        cout << "捕获异常: out_of_range -> " << e.what() << endl;
    }

    try {
        cout << "\n 测试逻辑错误..." << endl;
        int x = -5;
        if (x < 0)
            throw domain_error("负数没有平方根");
    } catch (const domain_error& e) {
        cout << "捕获异常: domain_error -> " << e.what() << endl;
    }

    try {
        cout << "\n 测试非法参数..." << endl;
        int n = 0;
        if (n == 0)
            throw invalid_argument("除数不能为0");
    } catch (const invalid_argument& e) {
        cout << "捕获异常: invalid_argument -> " << e.what() << endl;
    }

    try {
        cout << "\n 测试通用捕获..." << endl;
        throw runtime_error("未知运行时错误");
    } catch (const exception& e) {
        cout << "捕获 std::exception 派生类 -> " << e.what() << endl;
    } catch (...) {
        cout << "捕获所有异常" << endl;
    }

    cout << "\n 程序正常结束。" << endl;
    return 0;
}
```

 