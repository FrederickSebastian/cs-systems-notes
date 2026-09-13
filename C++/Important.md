1. 编译阶段报错一般是语法问题, 空指针的解引用是运行问题, 所以空指针的解引用会导致运行崩溃, 而该问题不会在编写或者编译阶段报错. 
2. 类的成员函数的地址不在类的内存中, 而是在栈帧中, 编译时已经确定. 所以对空类(空指针)解引用其函数, 不一定会出现崩溃. 
3. \*this指针本质上是一个形参, 它保存在栈中. (有时候会保存在寄存器中)
4. 类的成员变量按照类内的声明顺序初始化, 与初始化列表无关. 
5. 临时对象具有常性, 可以用const指针或者引用指向它. 
6. 静态成员变量不存在于对象的内存中
7. 内部类默认是外部类的友元
8. `string`类在库中实现时, 默认构造会生成一个包含`\0`的串.
9. `strlen`是在运行时确定的, `sizeof`是在编译时确定的
10. `vector`使用`insert`插入或者`erase`以后, 该迭代器会失效

## 有关const修饰指针与模板特化的问题

1. 考察const时, 按照从内向外读的方法: 
	- `const int * p`, 此处`p`是指向`const int`的指针. 
	- `int * const p`, 此处是`p`是一个const指针, 指向`int`.

接下来考察模板特化中的const问题: 
有模板
```C++
templete<T>
void function(const T & left, const T & right);

```

显示特化如下:

```C++
templete<>
void function<double>(const double * & left, const double * & right);
```

这里将`double *`看作一个整体, 赋给`T`. const是修饰整个`T`, 所以不能修改指针指向的内容. 这里的实例化是错误的. 正确的实例化如下: 

```C++
template<>
bool function<double*>(double* const& left,
                   double* const& right)

```


> 如果难以区分, 还可以按照const在`*`的左边或右边来区分const修饰的是谁.


# shared_ptr的循环引用问题

考察以下代码
```C++
#include <iostream>
#include <memory>
using namespace std;

class B;

class A {
public:
    shared_ptr<B> pb;

    A()  { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
};

class B {
public:
    shared_ptr<A> pa;

    B()  { cout << "B()" << endl; }
    ~B() { cout << "~B()" << endl; }
};

int main() {
    auto a = make_shared<A>();
    auto b = make_shared<B>();

    a->pb = b;
    b->pa = a;

    cout << "A use_count = " << a.use_count() << endl;
    cout << "B use_count = " << b.use_count() << endl;

    return 0;
}
```

以上代码中, A对象嵌套了指向B的指针, B也嵌套了指向A的指针. 然后两个指针分别指向A与B. **多个对象通过shared_ptr相互引用, 形成强引用闭环, 析构时引用计数无法归零, 导致内存泄漏.**
### 解决方案

使用`weak_ptr`, 该指针为弱引用. 不增加`shared_ptr`的引用计数. 不延长对象的生命周期. 

**使用weak_ptr时要考虑所有权语义.** 例如在双向链表中, `next`指针拥有下一个节点, 应当使用`shared_ptr`. `prev`指针本身只是指向上一个节点, 应当使用`weak_ptr`

 >正向关系: `shared_ptr`. 反向关系: `weak_ptr`. 