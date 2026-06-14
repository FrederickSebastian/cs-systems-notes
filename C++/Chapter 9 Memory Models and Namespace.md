
## 1. Header File Inclusion

- When compiling multiple files:
    
    - If the filename is enclosed in **angle brackets (`<>`)**, the compiler searches the **system’s standard header directories**.
        
    - If it’s enclosed in **double quotes (`""`)**, the compiler first searches the **current working directory** or the **source file’s directory**.
        

---

## 2. Header Guards (`#ifndef`)

- Used to prevent multiple inclusions of the same header:
    
    `#ifndef COORDIN_H_ #define COORDIN_H_ // code here #endif`
    
- If the macro (`COORDIN_H_`) is not defined, the code between `#ifndef` and `#endif` is processed.  
    (_See page 248 in C++ Primer Plus._)
    

---

## 3. Register Variables

- The `register` keyword (from C) **suggests storing a variable in a CPU register**.
    
- In C++11, `register` has **no effect**—it only applies to **automatic variables** (local to a block).
    

---

## 4. Static Variable Initialization

- If you **don’t explicitly initialize** a `static` variable, the compiler automatically sets it to **0**.
    
- Applies to pointers and structs as well.
    
- ⚠️ If you define a struct **outside** any function but **initialize it inside** one, it’s **not static**—its value is random unless explicitly declared `static`.
    

---

## 5. Static Duration Variables and Linkage

All **static-duration variables** exist for the **entire lifetime of the program**.  
**Linkage** determines where a name can be accessed.

|Linkage Type|How to Declare|Meaning|
|---|---|---|
|External Linkage|Declare outside any block|Accessible from all translation units|
|Internal Linkage|Declare outside any block + use `static`|Accessible only within the same file|
|No Linkage|Declare inside a block + use `static`|Exists throughout the program but only visible within that block|

---

## 6. The Meaning of `static`

- **Inside a function:** `static` means **no linkage**, **static duration**.
    
- **Outside a function:** `static` means **internal linkage** (visible only in that file).  
    So the meaning changes based on scope.
    
|**Storage Description**|**Duration**|**Scope**|**Linkage**|**How Declared**|
|---|---|---|---|---|
|**Automatic**|Automatic|Block|None|Declared **in a block**|
|**Register**|Automatic|Block|None|Declared **in a block with the keyword `register`**|
|**Static (no linkage)**|Static|Block|None|Declared **in a block with the keyword `static`**|
|**Static (external linkage)**|Static|File|External|Declared **outside all functions**|
|**Static (internal linkage)**|Static|File|Internal|Declared **outside all functions with the keyword `static`**|

---

## 7. Global Variables and `extern`

To use a global variable in other files:

`// file1.cpp int cats = 20;  // file2.cpp extern int cats;`

- Define it once in a source file.
    
- Declare it elsewhere using `extern`.  
    (_See page 255 for details._)
    

---

## 8. One Definition Rule (ODR)

- Each **variable** or **function** must have **only one definition** across the program.
    
- If a function is used in multiple files, each file must include its **prototype**.
    

---

## 9. Scope-Resolution Operator `::`

- Accesses the **global** version of a variable when a local one hides it:
    
    `cout << "Using the global variable: " << ::warming;`
    

---

## 10. Static and Duplicate Variables Across Files

Without `extern`, you **cannot** define the same variable name in two files:

`// file1.cpp int errors = 20;  // file2.cpp int errors = 5; // ❌ violates ODR`

To allow both:

`// file1.cpp int errors = 20;  // file2.cpp static int errors = 5; // ✅ internal linkage`

---

## 11. CV-Qualifiers

- **CV** means **const** and **volatile**.
    
- See page 260 for details on `volatile`.
    

---

## 12. The `mutable` Keyword

- Allows modification of specific members **even in `const` objects**:
    
    `struct Example {     mutable int counter; };`
    
- Rarely used in practice.
    

---

## 13. Linkage of `const` Variables

- Unlike normal variables, **`const` variables have internal linkage** by default.
    
- To make them external:
    
    `extern const int limit = 100;`
    
- They must be **initialized at declaration**.
    

---

## 14. Function Linkage

- Normal functions have **external linkage** by default.
    
- You can use `extern` to emphasize this, or `static` to make it **internal**.
    
- All functions have **static duration** (they exist for the whole program).
    

---

## 15. One Definition Rule and Inline Functions

- The **ODR** also applies to functions,  
    but **`inline` functions are exceptions** — they can be defined in header files safely.
    

---

## 16. Linking Between C and C++

- Language linkage between C and C++ (e.g., using `extern "C"`) is introduced on page 262.
    

---

## 17. Initialization with `new`

- You can use **C++11 list initialization** with `new`:
    
    `int* pit = new int{6};`
    
- Works similarly for structs and arrays.
    

---

## 18. Placement `new` Operator

- **Placement new** allows construction of objects at a **specific memory location**:
    
    `type_name* p1 = new (pointer) type_name; double* pd2 = new (buffer) double[6];`
    
- ⚠️ Memory allocated by **placement new** **cannot** be freed using `delete`.
    

---

## 19. Namespaces

Create a new namespace using the `namespace` keyword:

`namespace Jack {     double pail;     void fetch();     int pal; };`

- A namespace can be **global** or **nested**, but not defined **inside a code block**.
    
- Names inside a namespace **override global names**.
    

---

## 20. Extending an Existing Namespace

Namespaces are **open** — you can add new entities later:

`namespace Jack {     char* goose(const char*); } namespace Jack {     void fetch() { /* ... */ } }`

- You can declare prototypes and define them later within the same namespace.
    

---

## 21. Using Namespaces

Three main ways to access namespace members:

|Method|Example|Description|
|---|---|---|
|**Scope Resolution**|`Jack::pail = 12.34;`|Fully qualified access|
|**Using Declaration**|`using Jack::fetch;`|Imports a specific name|
|**Using Directive**|`using namespace Jack;`|Imports all names from the namespace|

---

## 22. `using` Conflicts and Scope Rules

Example: 
```C++
namespace Jill {
    double bucket(double n);
    double fetch;
    struct Hill { ... };
}
namespace Jack {
    double fetch;
}
char fetch;  // global variable
int main() {
    using Jill::fetch;   // bring Jill::fetch into local scope
    double fetch;        // ❌ Error: already defined locally
    cin >> fetch;        // uses Jill::fetch
    cin >> ::fetch;      // uses global fetch
    return 0;
}
void func() {
	using namespace Jack;
    using namespace Jill;
    fetch = 1.1;         // ❌ conflict: both Jack and Jill define fetch
    Jack::fetch = 2.0;   // ✅ explicitly access Jack::fetch
    Jill::fetch = 3.1;   // ✅ explicitly access Jill::fetch
}
```


📘 **Note:**  
If a namespace and a local declarative region define the same name:

- A **using declaration** → causes a **conflict** (error).
    
- A **using directive** → the **local version hides** the namespace version.
    

---

## 23. Using Declarations and Overloaded Functions

- When you bring a function into scope using a `using` declaration **without arguments**,  
    **all overloaded versions** of that function are imported.
    

---

✅ **Key Takeaways**

- `static`, `extern`, and linkage rules are fundamental in multi-file projects.
    
- Header guards prevent redefinition errors.
    
- Namespaces keep large projects organized and reduce name collisions.
    
- `using` must be used cautiously — name conflicts are common.
    
- Understand the scope-resolution operator `::` to disambiguate variable or function versions.


#  Common Pitfall（易错点）: `const` Variables in Headers vs Normal Variables (C++)

## 🔗 Linkage Difference

|Variable Type|Default Linkage|Effect When Defined in Header|
|---|---|---|
|**Non-const global variable**|**External**|❌ Causes **multiple definition linker errors** if included in multiple translation units.|
|**Const global variable**|**Internal**|✅ Each translation unit gets its **own independent copy**, so no linker error — but not shared.|

---

## Hidden Issue

- A `const` variable defined in a header **without `extern`** means every translation unit gets a **separate copy**.  
    → Different addresses, not shared globally.
    

---

## ✅ Correct Way to Share One `const` Across Translation Units

`// header.h extern const int g; // declaration  // global.cpp extern const int g = 42; // definition`

---

## Key Rule Summary

|Case|Result|
|---|---|
|Normal global variable in header|❌ Multiple definitions (linker error)|
|Const variable in header (no extern)|⚠ Compiles fine, but creates **multiple copies**|
|Const variable declared `extern` in header and defined once in `.cpp`|✅ Shared pro|