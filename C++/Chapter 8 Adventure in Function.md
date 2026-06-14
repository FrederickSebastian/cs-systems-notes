
## 1. Inline Functions

- See details in _C++ Primer Plus_, page 209.
    
- Inline functions are expanded in place at compile time, avoiding function call overhead for small, frequently used functions.
    

---

## 2. Reference Basics

- You **cannot declare a reference first and assign it later** like a pointer.
    
    `int rats = 101; int& rodents = rats; // rodents is a reference`
    

---

## 3. Passing by Value vs. Passing by Reference

- Passing **by value** copies the data.
    
- Passing **by reference** allows direct access to the original data — works similarly to a pointer but is syntactically cleaner.
    

---

## 4. Function Parameter Declarations

The only outward difference between passing styles lies in the **parameter declaration**:

`void swapr(int& a, int& b);   // pass by reference void swapv(int* a, int* b);   // pass by pointer`

- When passing pointers, you must use the `*` dereference operator.
    
- References handle this automatically.
    

---

## 5. Restriction on Reference Arguments

- Passing by reference is **more restrictive**:
    
    `double z = cube(x + 2.0); // valid for value argument`
    
    But not valid if the function parameter is a reference, because `x + 2.0` is not a variable (not an lvalue).
    

---

## 6. Temporary Variables for References

- When an actual argument doesn’t match a reference argument, C++ can **generate a temporary variable**.
    
- This behaves like pass-by-value — the original data won’t change.
    

---

## 7. Const References and Temporary Variables

- C++ only allows this for **const references**.
    
- Temporary variables are generated in two cases:
    
    1. The argument is the correct type but not an **lvalue**.
        
    2. The argument is of a different type that can be **converted** to the correct one.
        

 Always declare **reference parameters as `const`** whenever appropriate —  
it allows creation of temporaries and protects original data.

---

## 8. Rvalue References

- **Rvalue references (`&&`)** are introduced in Chapter 18.
    
- Used mainly for **move semantics** and **perfect forwarding**.
    

---

## 9. Returning References (Warning)

- Avoid returning a reference to a **local variable**, since it will cease to exist after the function ends.
    

❌ Dangerous example:

`const free_throws& clone2(free_throws& ft) {     free_throws newguy;  // created locally     newguy = ft;     return newguy;       // returns reference to destroyed object }`

---

## 10. Inheritance

- The language feature that allows one class to pass features to another is called **inheritance**.  
    (Introduced on page 223, discussed in Chapter 13.)
    

---

## 11. Class I/O Formatting Functions

- Functions such as `setf()`, `width()`, `precision()`, etc., are part of I/O stream manipulators.  
    See _C++ Primer Plus_, page 244.
    

---

## 12. Default Arguments

- Default arguments provide **fallback values** when arguments are omitted.
    
- Defined only in the **function prototype**, not in the definition.
    
    `char* left(const char* str, int n = 1);`
    

---

## 13. Order Rule for Default Arguments

- Defaults must be added **from right to left**:
    
    - You can’t give a default to an argument unless all arguments **to its right** also have defaults.
        
    - (See page 224 for details.)
        

---

## 14. Function Overloading

- Allows multiple functions with the **same name** but **different parameter types or counts**.
    
- The compiler selects the correct one based on the **function signature** (types and const qualifiers of parameters).
    
- Return type alone does _not_ distinguish overloaded functions.
    

Example:

`void show(int); void show(double); void show(const string&);`

---

## 15. Function Templates

- Templates make functions type-independent.
    

Example:

`template <typename AnyType> void Swap(AnyType& a, AnyType& b) {     AnyType temp;     temp = a;     a = b;     b = temp; }`

- When calling `Swap(x, y)`, the compiler deduces the types and generates a specialized version automatically.
    

---

## 16. Explicit Specializations

- **Explicit specializations** take priority over general templates but are lower priority than regular functions.
    
    `template <> void Swap<job>(job&, job&);`
    
    (See page 234 for details.)
    

---

## 17. Instantiations and Specializations

- **Instantiation**: compiler automatically generates a specific version of a template.
    
- **Specialization**: programmer provides a custom version for specific types.  
    (See page 236 for full explanation.)
    

---

## 18. `decltype`

- Syntax:
    
    `decltype(expr) varName;`
    
- Purpose: deduce the **exact type** of an expression at compile time.  
    (See page 242 in _C++ Primer Plus_.)
    

---

## 19. Const as a Function Signature Qualifier

- In function overloading, C++ differentiates between **const** and **non-const** member functions.  
    This means `const` can also be part of a function’s signature.
    

Example:

`class Sample {     int get() const;  // const version     int get();        // non-const version };`

---

✅ **Summary**

- Inline and reference features improve speed and safety.
    
- Always use `const` references when possible.
    
- Understand the restrictions of returning references.
    
- Default arguments and overloading increase flexibility.
    
- Templates and `decltype` make C++ functions more generic and powerful.
    
- `const` is part of a function’s identity — affects overload resolution.

## Summary
> 
> **1. Function Template**
> 
> `template <typename T> ReturnType func(T param) { /* ... */ }`
> 
> **Purpose:** Write one generic function that works with multiple data types.
> 
> ---
> 
> **2. Instantiation (实例化)**  
> The process of generating a **concrete function** from a template by replacing the template parameter with an actual type.
> 
> - **a. Implicit Instantiation** – happens automatically when calling the template:
>     
>     `func(42); // compiler creates func<int>(int)`
>     
> - **b. Explicit Instantiation** – forces the compiler to generate the code manually:
>     
>     `template void func<int>(int); // explicit instantiation`
>     
> 
> **Purpose:**
> 
> - Reduce compile time by pre-generating template code in one translation unit.
>     
> - Prevent linker errors when the definition is inside a `.cpp` file.
>     
> 
> ---
> 
> **3. Specialization (具体化)**  
> Provide a **custom implementation** of a template for a specific type.
> 
> `template<> void func<int>(int param) { /* custom code */ }`
> 
> **Purpose:**
> 
> - Override the generic behavior for special cases.
>     
> - Optimize performance or logic for particular types.
>     
> 
> ---
> 
> **4. Explicit Instantiation Summary**
> 
> - **Improves compile efficiency:** prevents multiple translation units (`.cpp` files) from each regenerating the same template version.
>     
> - **Prevents linker errors:** templates are only instantiated when used. If a template is declared but never called in a `.cpp` file, it might not get instantiated; explicit instantiation ensures the function is generated and linked properly.
>     
> 
> ---
> 
> **5. Key Difference**
> 
> - **Explicit instantiation** does **not** change program logic — it’s for **efficiency and reliability**.
>     
> - **Specialization** creates a **new, customized implementation** for a particular type, often altering logic or optimizing behavior.
>




```mermaid
flowchart TD
    A["你调用了一个函数名 func(...)"] --> B[" 编译器收集所有候选函数：
 普通函数
 模板函数（泛型模板）
 模板函数的显式具体化版本
 模板函数的实例化版本（隐式/显式）"]

    B --> C[" 判断是否存在完全匹配的普通函数？
(允许顶层 const 忽略、数组/函数退化等)"]

    C --> D["选择普通函数并结束"]
    C --> E[" 在模板相关候选中选择最优：
 如果有显式具体化版本，优先它
 否则按匹配程度选特化版本"]

    E --> F[" 决定用哪个模板版本实例化代码：
 如果已显式实例化过，直接用
 否则触发隐式实例化"]



