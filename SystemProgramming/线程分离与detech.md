
# 一、线程两种状态

- **Joinable（默认）**
    
    - 必须 `pthread_join`
        
    - 否则资源泄漏
        
- **Detached**
    
    - 自动回收资源
        
    - 不能 `join`
        

---

# 二、pthread_detach

```c
int pthread_detach(pthread_t thread);
```

**作用：**

- 将线程设为分离状态
    
- 线程结束后自动释放资源
    

**特点：**

- 不可再 `join`
    
- 适合无需返回结果的后台任务
    

---

# 三、pthread_join

```c
int pthread_join(pthread_t thread, void **retval);
```

**作用：**

- 等待线程结束（阻塞）
    
- 回收资源
    
- 获取返回值
    

---

# 四、pthread_cancel

```c
int pthread_cancel(pthread_t thread);
```

**作用：**

- 向线程发送“终止请求”
    

**特点：**

- 不保证立即终止（依赖取消点）(通常是在进入内核的时候)
    
- 不回收资源
    
- 通常需要配合 `join`
    

---

# 五、三者关系（核心）

- `cancel`：请求线程结束
    
- `join`：等待 + 回收
    
- `detach`：自动回收
    

---

# 六、典型组合

```c
pthread_cancel(tid);
pthread_join(tid, NULL);
```

含义：

1. 请求线程退出
    
2. 等待其结束
    
3. 回收资源
    

---

# 七、重要区别

|项目|cancel|join|detach|
|---|---|---|---|
|是否终止线程|请求终止|否|否|
|是否阻塞|否|是|否|
|是否回收资源|否|是|是|
|是否能获取返回值|否|是|否|

---

# 八、关键结论

- 线程结束 ≠ 资源释放
    
- 不 `join` 又不 `detach` ⇒ **资源泄漏**
    
- `cancel` 只是“请求”，不是强杀
    
- 生命周期 = **创建 + 终止 + 回收**
    

---

如果只记一句话：

> cancel 控制“结束”，join/detach 负责“收尸”。