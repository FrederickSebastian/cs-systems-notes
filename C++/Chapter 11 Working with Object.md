
## 1. Operator Overloading Basics

`operatorOP(argument-list);`

- The definition of an overloaded operator is the same as a function, except the function name is replaced with `operatorOP`.
    
- When using the operator, the **left-side object** is the one that invokes the operator, and the **right-side object** (if any) is passed as an argument.
    

---

## 2. Restrictions on Operator Overloading

a. At least **one operand must be a user-defined type**.  
b. You cannot violate the **original operator’s syntax rules**.  
c. You cannot **create new operator symbols**.

---

## 3. `friend` Keyword

- Allows a **non-member function** or **another class** to access private or protected members of a class.
    
- Declared inside a class using the `friend` keyword:
    
    `class MyClass {     friend void showData(MyClass& obj); };`
    
- Common use: overloading `<<` operator for stream output (see p.320).
    

---

## 4. Tip

- If a method needs to compute and return a new object,  
    see if a **constructor** can do it instead.  
    → Safer, cleaner, ensures the new object is initialized properly.
    

---

## 5. Re-overloading Operators

- Operators can be **overloaded multiple times**, just like functions.
    
- Each version must have a **different signature**(parameter type and quantity). 
    

---

## 6. Implicit and Explicit Conversions

- A constructor that takes **one parameter** can be used for **implicit conversions** between that parameter type and the class type.
    

---

## 7. The `explicit` Keyword

- Use `explicit` to **disable automatic (implicit) conversion**.
    
- Explicit conversion is still allowed manually with a cast.
    

---

## 8. Two-Step Conversion

C++ may perform **two conversions** in one call.  
Example:  
If a method expects a `double`, passing an `int` can convert:

`int → double → object`

---

## 9. Conversion Functions

Convert objects **to built-in types**.

### Rules

a. Must be a **class member function**.  
b. Must **not specify a return type**.  
c. Must **take no arguments**.

Example:

`operator double();`

Definition:

`ClassName::operator double() const {     return pounds; }`

---

## 10. `explicit` with Conversion Functions

- The keyword `explicit` can also be applied to **conversion functions**.
    
- It’s generally **recommended** to make conversion functions `explicit` to avoid ambiguous or unwanted conversions.
    

---

## 11. Summary of Conversions

|Conversion Type|Description|
|---|---|
|**Constructor with one parameter**|Converts a common type → class object|
|**Conversion function**|Converts a class object → common type|

---

## 12. Conversion Limits

- C++ only performs conversions **on function arguments**.
    
- You **cannot** use a built-in type to directly call a class method — it must first be explicitly converted.
    

---

## 13. Conversion + Operator Overloading

- Be cautious when mixing **operator overloading** and **conversions** —  
    too many automatic conversions can lead to **ambiguity**.
    
- In some cases, using a **friend function** helps avoid these issues.  
    (See details on p.342.)
    

---

## 14. Overloading the `<<` Operator

To allow `<<` chaining (e.g., `cout << obj1 << obj2;`),  
the overloaded function’s **return type** must be `ostream&`.

`friend ostream& operator<<(ostream& os, const ClassName& obj);`

### Summary:  
- Operator overloading behaves like a normal function and must involve user-defined types.
- `friend` lets non-member functions access private data (commonly for `<<`).  
- Constructors can handle conversions from common types to objects, while conversion functions do the reverse.  
- Use `explicit` to avoid unwanted implicit conversions.  
- When overloading `<<`, return `ostream&` to allow chaining.