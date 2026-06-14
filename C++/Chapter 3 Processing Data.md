## 1. The Essence of OOP

- The essence of **Object-Oriented Programming (OOP)** is **designing and extending your own data types**.
    

---

## 2. Variable Naming Rules

### Basic Rules

- Only **letters**, **digits**, and **underscores (`_`)** are allowed.
    
- The **first character** cannot be a digit.
    
- Names starting with **double underscores (`__`)** or **underscore + uppercase letter** are **reserved** for compiler and implementation use.
    

### Naming Convention Tips

- Keep naming **consistent and descriptive**.
    
- Embedding **type information** in variable names can improve readability, e.g.:
    
    `int countInt; double rateDouble;`
    

---

## 3. Initialization (C++11 Style)

- Modern initialization:
    
    `int rheas = {12}; // or equivalently int rheas{12};`
    
- Empty braces `int x{}` initialize to **zero**.
    
- Prefer brace-initialization for safety — it prevents narrowing conversions.
    

---

## 4. Integer Literals & Output

- Regardless of how an integer literal is defined (octal, hex, decimal),  
    `cout` **always prints in decimal by default**.
    
- To change the output base:
    
    `cout << oct;   // switch to octal cout << hex;   // switch to hexadecimal cout << dec;   // switch back to decimal`
    

---

## 5. Integer Suffixes

- Append suffixes to control the type explicitly:
    
    - `L` → `long`
        
    - `UL` → `unsigned long`
        
    
    `long num = 42L; unsigned long big = 123UL;`
    

---

## 6. `char` and Character Encoding

- `char` can be treated as a **small integer type**, even smaller than `short`.
    
- A **character literal** like `'A'` actually represents **its ASCII code**.
    

### Extended Character Types

|Type|Description|Prefix|Stream|
|---|---|---|---|
|`wchar_t`|Wide character|`L`|`wcin`, `wcout`|
|`char16_t`|UTF-16 character|`u`|—|
|`char32_t`|UTF-32 character|`U`|—|

Example:

`wchar_t wc = L'中'; char16_t c16 = u'你'; char32_t c32 = U'好';`

---

## 7. Constants

- Prefer using **`const`** over `#define` for symbolic constants:
    
    `const double PI = 3.1415926;`
    

---

## 8. Type Casting

### C-Style Casting

`(typeName) value;`

### C++-Style Casting

`static_cast<typeName>(value);`

⚠ **Note:** Type casting only affects the **expression** it’s used in —  
the **variable’s original type doesn’t change**.

### Example

`int a = 5, b = 2; double result = static_cast<double>(a) / b; // correct: 2.5`

---

## 9. `auto` Type Deduction (C++11)

- Automatically deduces type from initializer:
    
    `auto x = 0.0; // double auto y = 0;   // int`
    
- Especially useful with STL containers and iterators.
    

---

## 10. Type Limits (`<climits>`)

- Include:
    
    `#include <climits>`
    
- Provides symbolic constants for numeric limits:
    
    - `INT_MAX`, `INT_MIN`
        
    - `LONG_MAX`, `LONG_MIN`
        
    - `CHAR_MAX`, `CHAR_MIN`
        

---

## 11. Data Type Minimum Sizes (Standard Guarantees)

|Type|Minimum Width|Notes|
|---|---|---|
|`short`|≥ 16 bits||
|`int`|≥ size of `short`||
|`long`|≥ 32 bits|≥ size of `int`|
|`long long`|≥ 64 bits|≥ size of `long`|

Typical 16-bit integer range: **−32768 ~ 32767**

---

## 12. Floating-Point Types

### Scientific (E) Notation

Useful for very large or very small numbers:

`float num = 3.14e5;  // 3.14 × 10^5`

### Precision & Size

|Type|Precision (Decimal Digits)|Bit Width|Description|
|---|---|---|---|
|`float`|~6 digits|32 bits|single precision|
|`double`|≥15 digits|64 bits|double precision|

---

 **Key Takeaways**

- Stick to **consistent naming** and **modern initialization** (`{}`).
    
- Prefer **`const`**, **`auto`**, and **C++-style casts**.
    
- Know your **data type sizes** and **precision limits**.
    
- Use `<climits>` and `<cfloat>` when exact type ranges matter.