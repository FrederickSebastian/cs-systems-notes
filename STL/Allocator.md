STL 中的空间配置器是用于**动态分配和管理内存**的组件，它在底层提供了对内存的分配和释放操作。

---

1. 内存分配与释放
allocate(size_t n) 为指定数量的对象分配未构造的内存。

```cpp
T* allocate(size_t n) {
    return static_cast<T*>(::operator new(n * sizeof(T)));
}
```

默认情况下 STL 容器使用 std::allocator 作为分配器，但我们可以提供自定义分配器来优化内存管理。

deallocate(T* p, size_t n) 释放先前分配的内存。

```cpp
void deallocate(T* p, size_t n) {
    ::operator delete(p);
}
```

2. 对象构建与销毁
construct(T* p, const T& val) 在已分配的内存上构造对象。

```cpp
void construct(T* p, const T& val) {
    new(p) T(val);
}
```

destroy(T* p) 调用对象的析构函数，销毁对象但不释放内存。

```cpp
void destroy(T* p) {
    p->~T();
}
```

3. 自定义分配器, 封装了new与delete
```cpp
template <typename T>
struct MyAllocator {
    // 分配内存
    T* allocate(size_t n) {
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }

    // 构造对象
    void construct(T* p, const T& val) {
        new(p) T(val);
    }

    // 销毁对象
    void destroy(T* p) {
        p->~T();
    }

    // 释放内存
    void deallocate(T* p, size_t n) {
        ::operator delete(p);
    }
};
```