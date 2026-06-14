
## 1. Access Control Defaults

- If you don’t explicitly specify `private` or `public`,  
    **members declared before `public:` are private by default**.
    

---

## 2. Inline Function in Class Definition

- A function defined **inside** a class declaration is treated as an **inline function** automatically.
    

---

## 3. Inline Function Rule

- Every file that uses an inline function must have access to its **definition**.  
    👉 Therefore, **inline function definitions should be placed in header files**.
    

---

## 4. Class Constructor Example

Constructors initialize objects when they’re created.  
Default arguments are recommended in the prototype.

```C++
// Constructor definition 
Stock::Stock(const string& co, long n, double pr) 
{     
	company = co;      
	if (n < 0) 
	{         
		std::cerr << "Number of shares can't be negative; "<< company << " shares set to 0.\n";         
		shares = 0;     
	} 
	else 
	{         
		shares = n;     
	}      
	share_val = pr;     
	set_tot(); 
}
```

---

## 5. Object Initialization

C++ provides **two ways** to initialize objects using constructors:

`// a. Explicit form Stock food = Stock("World Cabbage", 250, 1.25); `
`// b. Implicit form (recommended) Stock garment("Furry Mason", 50, 2.5); `

They are **equivalent**.

You can also allocate dynamically:

`Stock* pstock = new Stock("Electroshock Games", 18, 19.0);`

---

## 6. Default Constructor

- A **default constructor** is still a constructor (often overloaded).
    
- You **should not rely** on the compiler-generated one.
    
- Always define your own default constructor to ensure safe initialization.
    

---

## 7. Common Misunderstanding

Be careful with constructor syntax differences:

`Stock first("Concrete Conglomerate"); // ✅ calls constructor`
`Stock second();                       // ⚠ declares a function, not an object `
`Stock third;                          // ✅ calls default constructor`

---

## 8. Destructor

- Destructor functions are rarely called directly — they are **automatically executed** when the object’s lifetime ends.
    

---

## 9. Initialization vs Assignment

- **Initializing** an object is **more efficient** than assigning to an existing object after creation.
    

---

## 10. Constant Member Functions

To declare a function that doesn’t modify the object:

`void Stock::show() const;`

**Caution:** Const object can only evoke the const member function.

---

## 11. `this` Pointer

- `this` is an implicit pointer available in all non-static member functions.
    
- It points to the **object that invoked the function**.
    

---

## 12. Constants within Classes

To define constants that belong to the class:

- Use `enum` or `static const`.
    
- Note that `enum` values are not stored as separate integers in objects.
    

---

## 13. Two Common Forms of Object Initialization

`Stock garment("World Cabbage", 250, 1.2); Stock food("Furry Mason", 50, 2.5);`

Both are valid and call the corresponding constructor.

---

## 14. Stack as a Container (Example)

The stack holds multiple data items and provides the following operations:

1. Create an empty stack
    
2. Push (insert an item)
    
3. Pop (remove the top item)
    
4. Check if the stack is full
    
5. Check if the stack is empty
    

---

## 15. Arrays of Objects

Arrays of objects have **special initialization rules** —  
see _C++ Primer Plus_, page **301** for detailed explanation.

# Summary

- 类内定义的函数默认 inline，inline 必须放在头文件。
    
- 默认访问权限是 private（直到遇到 public:）。
    
- 构造函数用于初始化对象，可显式或隐式调用。
    
- 默认构造函数最好手动定义，避免编译器生成的潜在风险。
    
- 析构函数自动执行，不需手动调用。
    
- 初始化比赋值高效。
    
- `const` 成员函数不改变对象状态，const 对象只能调用 const 函数。
    
- `this` 指针指向当前对象，是成员函数默认参数。
    
- 类内常量可用 `enum` 或 `static const` 定义。
    
- 栈容器提供 push、pop、判满、判空等功能。
    
- 对象数组初始化时会自动为每个元素调用构造函数。