
## 1. Function Prototypes

- **Different from ANSI C**, prototypes are **necessary in C++**.  
    They help the compiler verify argument types and return types before function calls.
    

---

## 2. Arrays as Function Arguments

- When you **pass an array to a function**, the array **decays into a pointer**.
    
- As a result:
    
    - You **lose size information** (`sizeof` no longer works properly).
        
    - The function only receives the **address of the first element**.
        

### Tip

You can pass **two pointers** instead of size:

`void show(const double* begin, const double* end);`

This allows iterating using the start and end addresses.

---

## 3. Using `const` with Arrays

- Use `const` properly to **protect arrays** that shouldn’t be modified by the function.
    

Example:

`void display(const int arr[], int n);`

---

## 4. Pointer and Const Rules

- A **common pointer** cannot point to a **`const` variable**.
    
- C++ **disallows assigning** the address of a const variable to a non-const pointer —  
    this applies to **array pointers** too.
    

Example:

`const int n = 10; int* p = &n;      //  Error const int* p2 = &n; //  OK`

---

## 5. Functions and `std::array`

When working with `std::array`, parameter passing differs from raw arrays.

- **Display-only (no modification):**
    
    `void show(std::array<double, 4> da); // argument passed by value`
    
- **Modifiable:**
    
    `void fill(std::array<double, 4>* pa); // pass by address`
    

---

## 6. Recursion

- Recursion (`递归`) should be reviewed carefully.
    
- It allows a function to **call itself**, but must have a **base case** to stop infinite recursion.
    

---

## 7. Pointers to Functions

### Key Concept

- **Function names** in expressions represent **addresses**.
    

`process(think);    // passes address of think() to process() thought(think());  // passes return value of think() to thought()`

---

## 8. Declaration of Function Pointers

- A pointer to a function must include:
    
    - The **return type**
        
    - The **parameter types** (the function’s signature)
        

Example:

`double pam(int);        // function prototype double (*pf)(int);      // pf points to a function taking (int) and returning double`

⚠ **Parentheses matter!**  
Without them:

`double *pf(int); // declares a function returning a pointer to double`

Not the same thing.

---

## 9. Declaration Trick (Important Tip)

- To declare a function pointer easily:
    
    1. Write a normal function prototype.
        
    2. Replace the function name with `(*pf)`.
        

Example:

`double pam(int); double (*pf)(int);`

Now `pf` is a **pointer to a function** that matches the type of `pam`.

---

## 10. Assigning Function Addresses

`double pam(int); double (*pf)(int);  pf = pam;  // pf now points to pam()`

---

## 11. Using a Function Pointer

To **call** the function via pointer:

`cout << (*pf)(20) << " hours" << endl; // or simply: cout << pf(20) << endl;`

---

## 12. Manual vs. Automatic Initialization

- Be aware of the **difference between manual assignment** and **auto initialization** of function pointers.
    
- See details in _C++ Primer Plus_, page 202.
    

---

## 13. Using `typedef` to Simplify Function Pointer Declarations

You can define an alias for a function pointer type:

`typedef const double* (*p_fun)(const double*, int); // p_fun is now a type name`

This simplifies future declarations:

`p_fun process;`

(See _C++ Primer Plus_, page 204.)

---

 **Summary**

- Always declare prototypes in C++.
    
- Passing arrays → decay into pointers (no `sizeof` info).
    
- Protect data with `const`.
    
- Remember: **const pointer rules** apply to both simple and array pointers.
    
- `std::array` should be passed by value or address depending on whether you modify it.
    
- Function pointers must **match signatures exactly**, including return type and parameters.
    
- Use `typedef` to clean up complex pointer-to-function syntax.