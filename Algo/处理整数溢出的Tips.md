>先规定结果必须满足 `L <= expr <= U`，再在数学意义上反解这个不等式


### 1. 加法 `a + b`

要求：

```
L <= a + b <= U
```

根据 `b` 的正负处理：

```
bool addOverflow(int a, int b) {
    if (b > 0 && a > INT_MAX - b) return true;
    if (b < 0 && a < INT_MIN - b) return true;
    return false;
}
```

可以理解成：

```
b > 0：防止撞上界
a <= U - b

b < 0：防止撞下界
a >= L - b
```

注意这里 `INT_MAX - b`、`INT_MIN - b` 本身都是安全的。

---

### 2. 减法 `a - b`

要求：

```
L <= a - b <= U
```

于是：

```
bool subOverflow(int a, int b) {
    if (b > 0 && a < INT_MIN + b) return true;
    if (b < 0 && a > INT_MAX + b) return true;
    return false;
}
```

记忆方式：

```
减正数 → 往下走 → 防 INT_MIN
减负数 → 往上走 → 防 INT_MAX
```

即：

```
b > 0：
a >= L + b

b < 0：
a <= U + b
```

---

### 3. 乘法 `a * b`

这个稍微烦一点，因为除以负数会改变不等号方向。

万能安全模板：

```
bool mulOverflow(int a, int b) {
    if (a == 0 || b == 0)
        return false;

    if (a > 0) {
        if (b > 0)
            return a > INT_MAX / b;
        else
            return b < INT_MIN / a;
    } else {
        if (b > 0)
            return a < INT_MIN / b;
        else
            return a < INT_MAX / b;
    }
}
```

不用死背代码，可以理解成两个规则：

```
同号相乘 → 结果为正 → 防 U
异号相乘 → 结果为负 → 防 L
```

然后通过除法把乘法反解掉。

例如：

```
a > 0, b > 0

a*b <= U

a <= U/b
```

所以：

```
a > U / b
```

就是溢出。

再比如：

```
a > 0, b < 0

a*b >= L
```

除以正数 `a`：

```
b >= L/a
```

所以：

```
b < L / a
```

就是溢出。

这就是乘法四个分支的来源。

---

### 4. 除法 `a / b`

整数除法反而非常简单，只有两个危险：

```
b == 0
```

以及：

```
a == INT_MIN && b == -1
```

因为：

```
INT_MIN = -2147483648
```

所以数学结果是：

```
2147483648
```

超过 `INT_MAX`。

因此：

```
bool divOverflow(int a, int b) {
    if (b == 0)
        return true;

    if (a == INT_MIN && b == -1)
        return true;

    return false;
}
```

这也解释了为什么：

```
-INT_MIN
abs(INT_MIN)
```

都有问题。


**遇到复杂表达式, 例如a * k + b, 就把复杂判断拆解为上述的简单判断的组合**
