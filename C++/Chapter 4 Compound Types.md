

## 1. `sizeof` Operator

- `sizeof` is **an operator**, not a function.
    
- It returns the **size in bytes** of a type or data object.
    

---

## 2. Array Initialization

- Arrays can **only be initialized at definition**.  
    You **cannot reinitialize** an array later.
    
- C++11 allows **brace initialization**:
    
    `int arr[] = {1, 2, 3};`
    

---

## 3. `cin` and Input Handling

- `cin` works like `scanf`, using **whitespace, tabs, and newlines** to separate strings.
    
- The `istream` class (base of `cin`) provides **line-oriented functions**:
    
    - `getline()`
        
    - `get()`
        

---

## 4. `cin.getline()` vs `cin.get()`

|Function|Behavior|
|---|---|
|`cin.getline(array, size)`|Reads input **until newline**, **discards** newline|
|`cin.get(array, size)`|Reads input **until newline**, but **keeps** newline in buffer|

### Notes

- `cin.get()` with **no arguments** reads a **single character**, including newline.
    
- You can **chain** the calls:
    
    `cin.get(name, Arsize).get();` 
    
- It’s generally **better to use `get()`** instead of `getline()`.
    

---

## 5. Mixing String and Numeric Input

- Mixing numeric and string input causes issues:  
    the newline after a number is **left in the input buffer**.
    
- Always use `cin.get()` to **discard the newline** before reading strings.
    

---

## 6. The `string` Class

### Basics

- Include the header:
    
    `#include <string>`
    
- Initialization is similar to C-strings.
    
- Supports:
    
    - Input/output with `cin` and `cout`
        
    - Array-like access (`str[i]`)
        
    - Automatic memory management
        

### Features

- `'\0'` automatically appended at end
    
- Supports **list initialization** (C++11)
    
- Supports **assignment and concatenation**:
    
    `string str1, str2; str1 = str2; str1 += " world"; string str3 = str1 + str2;`
    
- Read entire line:
    
    `getline(cin, name_string);`
    

 **String objects are safer and more convenient than character arrays.**

---

## 7. String Utilities

- Get string length:
    
    `int length = str_name.size();`
    
- I/O summary:
    
    - Use `getline(cin, str)` for full-line input.
        
    - `cin >> str` skips whitespace.
        
    - `cout << str` prints normally.
        

---

## 8. Raw Strings

- Raw strings interpret characters **literally** (no escape sequences).
    
    `cout << R"(Jim "King" Tutt uses \n instead of endl.)";`
    
- `R"()"` marks a raw string literal.
    

---

## 9. Structures (`struct`)

- C++ allows **omitting the `struct` keyword** when declaring variables:
    
    `struct Point { int x, y; }; Point p1; // valid`
    
- C++11 supports **list initialization** for structs:
    
    `Point p1{3, 4};`
    
- A `string` object can be a struct member.
    

---

## 10. Pointers

### Initialization Warning

Never use an uninitialized pointer:

`long* fellow; *fellow = 223323; //  undefined behavior`

Always initialize before dereferencing:

`long value = 223323; long* fellow = &value; // `

---

## 11. Dynamic Memory Allocation

### Allocate with `new`

`typeName* pointer = new typeName;`

- Creates a new object in **heap memory**.
    
- Can only access it through the pointer.
    

### Free with `delete`

`delete pointer;`

- Frees the allocated memory but **does not delete the pointer itself**.
    
- Always use `new` and `delete` in **pairs**.
    

---

## 12. Dynamic Arrays

`typeName* pointer = new typeName[num_elements]; delete[] pointer;  // note the brackets`

- Allocates and frees dynamic arrays.
    
- `sizeof` on arrays vs. pointers:
    
    - Array → size of entire array
        
    - Pointer → size of the pointer only
        

---

## 13. Array Addresses

- `&array` gives the address of the **entire block**.
    
- `array` itself decays to the **address of the first element**.
    
- Example:
    
    `cout << charArray << endl;`
    
    Prints all characters until `'\0'`.
    

---

## 14. Using `new` with Strings

`ps = new char[strlen(otherarray) + 1];`

Allocates memory dynamically based on another string.

---

## 15. `vector` Template Class

- Dynamic array; automatically resizes as needed.
    
- Include header:
    
    `#include <vector> using namespace std;`
    
- Syntax:
    
    `vector<typeName> objName(n_elem);`
    
- `n_elem` can be a variable or constant.
    
- Less efficient than arrays, but much safer and easier to use.
    

---

## 16. `array` Template Class

- Fixed-size array, safer than built-in arrays.
    
- Uses **static memory allocation**.
    
    `#include <array> array<typeName, n_elem> arrName;`
    
- `n_elem` must be a **constant**.
    

---

   **In short:**

- Use **`string`** instead of char arrays.
    
- Use **`vector`** for dynamic arrays.
    
- Use **`array`** for fixed-size collections.
    
- Always **initialize pointers** before use.
    
- Remember to **delete what you new**.


>在第四章中，系统学习了数据类型
 >概览：
> 1. I/O:
        `cin, cout, cin.getline(array, arsize), cin.get(name, arsize), cin.get();
    2. `string class, initialization, declare, 赋值，融合（concatenation）
    3. `new: typeName * pointerName = new pointerName;
    4. ` delete: delete [] pointerName
    5. `vector: vector<typeName> objName(n_elem)  
    6. `array class: array<typeName, n_elem> arrName