

---


## 一、C++ I/O 流模型

1. C++ I/O 基于**流（stream）模型**：数据在程序与外部设备之间以流的形式传输。
    
    - **输入流（istream）**：从设备 → 程序
        
    - **输出流（ostream）**：程序 → 设备
        
    - **双向流（iostream）**：输入输出并存
        
2. 头文件：
    
    - `<iostream>`：标准输入输出
        
    - `<fstream>`：文件输入输出
        
    - `<sstream>`：字符串流
        
3. 标准流对象：
    
    | 名称 | 类型 | 作用 |
    |---|---|---|
    |`cin`|istream|标准输入|
    |`cout`|ostream|标准输出|
    |`cerr`|ostream|非缓冲错误输出|
    |`clog`|ostream|缓冲错误输出|
    

---

## 二、文件流（File Streams）

4. 文件I/O类：
    
    - `ifstream`：从文件读取（输入流）
        
    - `ofstream`：写入文件（输出流）
        
    - `fstream`：可同时读写
        
5. 打开文件：
    
    ```cpp
    #include <fstream>
    using namespace std;
    
    ofstream fout("data.txt");       // 写文件
    ifstream fin("data.txt");        // 读文件
    fstream fio("data.txt", ios::in | ios::out); // 读写模式
    ```
    
6. 打开模式（`ios` 标志）：
    
    |模式|含义|
    |---|---|
    |`ios::in`|读|
    |`ios::out`|写|
    |`ios::app`|追加|
    |`ios::ate`|打开后定位到文件末尾|
    |`ios::trunc`|打开时清空内容|
    |`ios::binary`|二进制模式|

	Caution: `ios_base::app` 只允许将数据添加到文件尾部, `ios_base::ate` 仅仅定位到文件尾部, 但是可以自由移动. 
4. 检查是否打开成功：
    
    ```cpp
    if (!fin.is_open()) {
        cerr << "无法打开文件" << endl;
    }
    ```
    
5. 关闭文件：
    
    ```cpp
    fin.close();
    fout.close();
    ```
    

4. 二进制模式下写入数据用成员函数 `write()` . 需要注意的是, 必须将地址强制转换为指向char的指针, 要获取字节数, 使用`sizeof` 运算符. 
		`fout.write( (char * ) & p, sizeof p);` 
5. 要从二进制文件中读取信息, 使用`ifstream` 对应的`read()` 方法
		`fin.read((char *) & p, sizeof p);`
6. Caution: string对象本身没有包含字符串, 而是一个指向字符串内存的指针. 因此不能将string对象存入到文件中.
7. `seekg()` 方法将输入指针移动到指定的文件位置(用于ifstream对象):
		`fin.seekg(30, ios_base::beg); // 30 bytes beyond the beginning`
		`fin.seekg(-1, ios_base::cur);  // back up one byte`
		`fin.seekg(0, ios_base::end);  // go to the end of the file`
8. `seekp()`方法用于ofstream对象,将指针移动至距离开头特定字节的位置. 只接受一个参数
9. 如果要检查当前文件指针的位置, 对于输入流可以使用`tellg()` 方法, 对于输出流可以用 `tellp()`方法.
10. fstream对象包含了两个缓冲区, 一个用于输出, 另一个用于输入. 并可以同步协调的处理输入于输出的指针. 
11. 别忘了, 在file IO中, `flush` 与 `endl` 仍然可以用来刷新缓冲区.
12. 内核格式化参见page638.
---

## 三、文件读写操作

9. 逐行读写：
    
    ```cpp
    string line;
    while (getline(fin, line))
        cout << line << endl;
    fout << "Hello File!" << endl;
    ```
    
10. 按字符读写：
    

```cpp
char ch;
while (fin.get(ch)) cout << ch;
fout.put('A');
```

11. 读写二进制文件：
    

```cpp
fout.write((char*)&data, sizeof(data));
fin.read((char*)&data, sizeof(data));
```

12. 定位文件指针：
    

```cpp
fin.seekg(0, ios::beg);  // 移到文件开头
fout.seekp(0, ios::end); // 移到文件末尾
long pos = fin.tellg();  // 获取当前位置
```

13. 文件I/O与缓冲：
    
    - 标准输出流一般为**行缓冲**（遇到 `\n` 自动刷新）。
        
    - 文件流为**全缓冲**，需显式调用 `flush()` 或 `close()` 才保证写入完成。
        

---

## 四、字符串流（String Streams）

14. 字符串流可在内存中进行格式化I/O操作。
    
    ```cpp
    #include <sstream>
    using namespace std;
    
    stringstream ss;
    ss << "Pi = " << 3.1415;
    string result = ss.str();     // 输出字符串
    ss.str("42");
    int n;
    ss >> n;  // 从字符串读出整数
    ```
    
15. 类型：
    

- `istringstream`：只读
    
- `ostringstream`：只写
    
- `stringstream`：读写
    

16. 常用于：
    

- 字符串与数字间的转换
    
- 格式化日志输出
    
- 解析文本数据
    

---

## 五、I/O 格式控制

17. `<iomanip>` 提供格式控制符：
    

- `setw(n)`：宽度
    
- `setprecision(n)`：小数精度
    
- `setfill(ch)`：填充字符
    
- `fixed` / `scientific`：定点或科学计数法
    

```cpp
cout << setw(8) << setfill('0') << 42; // 输出 00000042
```

18. `cout` 也支持流操作符链式调用：
    

```cpp
cout << fixed << setprecision(3) << 3.14159;
```

---

## 六、模板高级应用

19. **模板函数特化（Function Specialization）**
    

```cpp
template <typename T>
void Swap(T& a, T& b) { ... }

template <> void Swap<Job>(Job& a, Job& b) { ... } // 特化版本
```

20. **模板类特化（Class Specialization）**
    

```cpp
template <class T>
class Container { ... };

template <> class Container<int> { ... }; // 特化int版本
```

21. **部分特化（Partial Specialization）**
    

```cpp
template <class T1, class T2>
class Pair { ... };

template <class T>
class Pair<T, int> { ... }; // 部分特化
```

22. **模板与继承结合：**  
    模板类可被普通类或其他模板类继承。
    

```cpp
template<class T>
class Base { ... };

template<class T>
class Derived : public Base<T> { ... };
```

23. **模板与友元：**
    

- 模板类可声明特定模板函数为友元。
    
- 也可声明整个模板为友元（即所有实例共享访问权）。
    

---

## 七、异常与I/O结合

24. 文件操作中可使用异常方式处理错误：
    

```cpp
fin.exceptions(ifstream::failbit | ifstream::badbit);
try {
    fin.open("data.txt");
} catch (ios_base::failure& e) {
    cerr << "文件错误: " << e.what() << endl;
}
```

---

## 八、 使用cout进行输出

25. flush与endl控制符本质上是刷新缓冲区。后者提供一个换行符。
26. `ios_base` 类存储了描述格式状态的信息。
27. `width()` 方法返回以前的字段宽度(类型为int), 设置下一次的字段宽度. 仅对下一次的项目有影响
28. 改变显示时的计数系统：dec, hex and oct（十进制，十六进制与八进制）
		`cout << hex;`
29. 调整字段宽度：使用`width`**成员函数**
		`int w = cout.width(30); // 先保存以前的，再将其调整为30`
		右对齐是默认的。
30. 改变填充字符：`fill`成员函数
		`cout.fill('*');`
31. 设置浮点数的显示有效数字：`precision`成员函数
		`cout.precision(2);`
32. `setf()`成员函数进行格式化输出
	- 显示末位的0：`cout.setf(ios_base::showpoint)` 
		
	- `setf()`有两个原型，参见page606. 其返回值为`fmtflgs` 可以使用它保存以前的值
		
	- 消除`setf()`的效果：使用`unsetf`
		`cout.unsetf(ios_base::showpoint);`
	- 使用参数`ios_base::fixed` 来返回到默认模式 
25. `setw()` 设置下一次输出时的字段宽度, 仅仅影响下一次. 不过它不在 `<iostream>` 里，而是在 **`<iomanip>`** 里。
26. `setprecision()` 仅仅设置下一次输出时的精度.
27. `setfill()` 同上. 都需要这个头文件.
## 九、使用cin进行输入

32. 三种计数系统同样适用于`cin`
33. 三种流状态：`eofbit` , `badbit` , `failbit`
34. 使用`cin.clear()`来清除他们
35. 单字符输入:
	- `get(char &);` 读取下一个字符, 不论是空格还是换行符. 
	- `get(void)` 读取空白,用返回值将输入传递给程序
36. getline的两个版本
	-  `istream& getline(char* buffer, streamsize count)`  
		**作用：** 从输入流读取一整行，直到遇到 **换行符为止（换行符会被丢弃）**，将读取的内容放进 C 字符数组。第二个参数是指定的读取的数量.
	-  `istream& getline(char* buffer, streamsize count, char delim)`
		**作用：** 自定义分隔符版本。读取到 `delim` 之前的内容(实际delim是一个分界符)，存入 buffer。
37. get的字符串输入版本的函数原型与上述getline类似, 不过get() 将分界符留在输入队列中, getline不保留, 直接丢弃.
38. `cin.peek()` 用于查看下一个输入的字符, 但不抽取输入流中的字符. 对应的有 `putback()` 方法将字符放回输入流中. 
## 八、现代C++的I/O安全建议

25. 推荐使用 `std::string` + `stringstream` 取代 C 风格 I/O。
    
26. 文件I/O必须检查 `.is_open()` 与 `.good()` 状态。
    
27. 若用到模板 + 文件I/O，推荐显式指定模板类型，避免隐式推导带来的歧义。
    
28. 对二进制文件操作时，应始终使用 `ios::binary` 打开模式，防止换行符转换。
    
29. RAII 原则同样适用于文件流：定义在局部作用域中，自动关闭文件。
    

---

## 🧩 Summary

- I/O流是C++最基础的输入输出机制，分为输入流、输出流与双向流。
    
- 文件I/O通过 `<fstream>` 实现，支持多种打开模式与二进制操作。
    
- 字符串流 `<sstream>` 用于在内存中进行格式化输入输出。
    
- `<iomanip>` 提供强大的输出格式控制能力。
    
- 模板支持**特化**与**部分特化**，能结合继承与友元实现复杂泛型结构。
    
- 异常与流操作结合可实现健壮的文件处理。
    
- 遵循 RAII 原则，智能指针与文件流共同保障资源安全释放。