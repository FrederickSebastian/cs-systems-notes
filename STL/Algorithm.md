
## 概述

- `<algorithm>` 是所有STL头文件中最大的一个, 包含了比较, 交换, 查找, 遍历, 复制, 修改, 移除, 反转, 排序和合并等操作
- `<numeric>` 包含了一些序列上数学运算的函数
- `<functional>` 定义了函数对象, 可用于算法函数的参数
- 算法的分类: 
	1. 非可变序列算法: 仅读取, 比较数据, 不修改数据, e.g. find, equal, binary_search.
	2. 可变序列算法: 读取并修改数据, e.g. reverse, sort, remove
	3. 排序算法: 排序, e.g. sort, stable_sort, partial_sort
	4. 数值算法: 主要在`<numreic>` 中, 用于数学运算


---

## 一. 排序

### std::sort 
可以对**支持随机访问的容器**进行排序, 默认按照**升序**排序, 采用**Introsort**自适应算法, 其基于快速排序, 插入排序, 堆排序.
1. 一般用法即升序排序:
	`std::sort(v.begin(), v.end());`
2. 添加第三个参数`std::greater`或者自定义比较函数可以改为降序排序:
	`std::sort(v.begin(), v.end(), std::greater<int>());  // 务必要注意这里的greater有实例化`
	`std::sort(v.begin(), v.end(), [](int a, int b) {return a > b; })  // 自定义labmada`
3. sort也可以用于**定义了比较操作符(<)** 的自定义类型, 例如类或结构体, sort会自动使用该操作符来比较其对象. 

### std::stable_sort 
与sort功能类似, `stable_sort`能**保证相等元素的相对位置不变**. 基于归并算法, 其平均时间复杂度为O(N logN), 但需要额外的空间实现稳定性.


### std::is_sorted
用于判断序列是否有序(升序), 返回bool值, 平均时间复杂度O(N):
	`if(std::is_sorted(a.begin(), a.end())`
	同样也可以自定义第三个参数来改变排序规则

### std::partial_sort
对迭代器参数指定的部分元素排序, 基于堆排序, 平均时间复杂度O(N logN):
	`std::partial_sort(v.begin(), v.begin() + 4, v.end());  // 对前四个元素升序排序`
	定义第4个参数改变排序规则.

### std::reverse
用来反转指定范围内的元素顺序(貌似跟排序不沾边), 线性时间复杂度O(N), 通过**交换序列的首尾元素**来反转整个序列，每次交换操作的时间复杂度是 O(1)，总共需要进行 n/2 次交换。
	`reverse(v.begin(), v.end());`


### std::merge
将两个**已排序**的序列合并为一个有序序列, 第六个参数可以自定义排序规则, **目标序列必须足够大来容纳两个原序列**:
```cpp
std::vector<int> v1 = {1, 3, 5, 7, 9}; // 输入序列必须是已排序的
std::vector<int> v2 = {2, 4, 6, 8, 10};
std::vector<int> v_out(v1.size() + v2.size()); // 输出序列的大小必须足够大，以容纳两个输入序列的所有元素

std::merge(v1.begin(), v1.end(), v2.begin(), v2.end(), v_out.begin());
```


---

## 二. 拷贝/替换

### std::copy
将v1中的指定范围的元素拷贝到v2容器中, **务必确保目标容器有足够的空间存放所有的元素**, 如果空间不足, 会发生未定义的行为. 函数返回迭代器指向最后一个被拷贝的元素的下一个位置, 时间复杂度O(N): 
	`std::copy(v1.begin(), v1.end(), v2.begin());`

### std::copy_if
只拷贝满足特定条件op的元素, 其余与copy相同
	`copy_if(v1.begin, v1.end(), back_inserter(v2), op);`

### std::replace
`std::replace(a.begin(), a.end(), old_value, new_value)` 函数用于**将一个范围内所有等于给定值 old_value 的元素替换为新值 new_value**，没有返回值。平均时间复杂度为 O(N)。
	`std::replace(v1.begin(), v1.end(), 2, 9); // 将 v1 中所有等于 2 的元素替换为 9`

### std::replace_if
将容器中**指定范围**满足条件的元素替换为新值(最后一个参数是将要替换的新值):
```cpp
bool isEven(int x) {
    return x % 2 == 0;
}

int main() {
    std::vector<int> v1 = {1, 2, 3, 4, 5};
    std::replace_if(v1.begin(), v1.end(), isEven, 0);
    for (auto i : v1)
        std::cout << i << ' '; // 输出序列为1,0,3,0,5
    return 0;
}
```

### std::swap
交换任意两个容器(对象)的值, 平均时间复杂度O(1)
	`swap(v1, v2);  // 注意这里不是begin()`


---

## 三. 遍历

### std::for_each
std::for_each(a.begin(), a.end(), op) 函数**对容器 a 中 `[begin, end) ` 内的元素执行指定的操作 op**，复杂度为O(N)。op 是一个**可调用对象**，能被应用到范围内的每个元素上，表示执行一个操作如打印、修改或计数容器中的元素。平均时间复杂度为 O(N)。
```cpp
// 该可调用对象为一个函数，接受一个参数并返回 void，给每个元素执行平方操作并输出
void printSquare(int num) {
    std::cout << num * num << " ";
    std::cout.flush(); // 确保输出立即显示
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    std::for_each(numbers.begin(), numbers.end(), printSquare);  // 输出结果为 1 4 9 16 25
    return 0;
}
```
*Caution: 注意这里的函数对象*

### std::transform
std::transform(a.begin(), a.end(), b.begin(), op) 函数**将容器 a 的元素遍历并通过给定的 op 操作后存储到容器 b 中**。该算法不会自动为目标容器分配内存，因此需要确保目标容器有足够的空间。平均时间复杂度为 O(N)。
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
std::vector<int> v2;
v2.resize(v.size()); // 使 v2 大小与 v 相同，确保目标容器有足够的空间
std::transform(v.begin(), v.end(), v2.begin(), [](int i) { return i * 2; });
// 使用 transform 将容器元素加倍并存储到另一个容器中
for (auto i : v2)
    std::cout << i << ' '; // 输出序列为 2, 4, 6, 8, 10
```


---

## 四. 查找

### std::find
在指定范围内**按值value**查找指定元素, 并返回其迭代器, 否则返回迭代器`end()`, 时间复杂度O(N), 下面是常用用法: 
```cpp
auto it = std::find(v.begin(), v.end(), value);
if (it != v.end())   // 由于end位置无元素, 所以可以这样做
{
    std::cout << "Found: " << *it << std::endl;
} else 
{
    std::cout << "Not found" << std::endl;
}
```


### std::binary_search
使用二分查找高效查找指定元素, 返回bool. **要求序列必须有序** , 时间复杂度O(log N), 参数列表同`find`


### std::find_if
按照条件op查找指定元素, 返回对应迭代器, 找不到返回`end()`, 时间复杂度O(N):
	`auto it = std::find_if(v.begin(), v.end(), op);`

### std::adjacent_find
std::adjacent_find(a.begin(), a.end()) 函数用于查找相邻重复元素，若找到则返回相邻元素的第一个位置的迭代器，否则返回结迭代器 end()，平均时间复杂度为 O(N)。
```cpp
std::vector<int> v = {1, 2, 2, 3, 4, 4, 5};

auto it = std::adjacent_find(v.begin(), v.end());
if (it != v.end()) {
    std::cout << "Found adjacent elements: " << *it << " and " << *(it + 1) << std::endl;  // Tips
} else {
    std::cout << "No adjacent elements found" << std::endl;
}
```


### std::count
按值value统计元素出现次数, 统计自定义对象时需要重载== . 时间复杂度O(N): 
	`std::count(v.begin, v.end(), value);`

### std::count_if
按照条件op统计元素出现次数, 时间复杂度O(N):
	`std::count_if(v.begin(), v.end(), op);`


---


## 五. 最值

### std::max_element / std::min_element
std::max_element / std::min_element 可以分别找最大值和最小值，返回其**迭代器**，复杂度为 O(N)。
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

auto max_it = std::max_element(v.begin(), v.end());
auto min_it = std::min_element(v.begin(), v.end());
```

### std::max / std::min
std::max / std::min 可以分别找最大值和最小值并返回**其值** *(注意与上述区别)*，还可以添加自定义比较函数用于定义比较规则，对于两个值比较复杂度为 O(1)，列表 N 个值比较复杂度为 O(N)。

```cpp
int mx = std::max(a, b);
int mn = std::min(a, b);  // 用容器也可以
```

### std::minmax(a, b)
std::minmax(a, b) 返回一个 **pair 类型**，第一个元素 first 是 min(a, b)，第二个元素 second 是 max(a, b)，同样可以自定义比较函数，对于两个值比较复杂度为 O(1)，列表 N 个值比较复杂度为 O(N)。
```cpp
std::pair<int, int> t = std::minmax(4, 2);
// t.first = 2, t.second = 4

std::vector<int> v = {1, 4, 5, 2, 9, 10, 3};
std::pair<int, int> u = std::minmax(v.begin(), v.end());
// u.first = 1, u.second = 10
```


### std::minmax_element
返回**pair类型的迭代器**, 与上述差不多, tips: 可以解引用迭代器.


---

## 六. 堆操作

### 堆操作的几个常用函数
1. **std::make_heap(a.begin(), a.end())** 函数用于**将给定范围内的元素转换成一个堆**,默认情况下使用 less 来构建最大堆，也可以自定义比较函数。时间复杂度为 O(N)。
2. **std::push_heap(a.begin(), a.end())** 函数用于**在已存在的堆中添加一个新元素**，时间复杂度为 O(log N)。
3. **std::pop_heap(a.begin(), a.end())** 函数用于**删除堆顶元素并将最后一个元素移动到堆顶的位置**，然后通过下沉调整来恢复堆的性质，时间复杂度同样为 O(log N)。
4. **std::sort_heap(a.begin(), a.end())** 函数用于**对堆中的元素进行排序**。它通过重复执行 pop_heap 操作，将堆顶元素移动到容器的末尾，直到堆为空。这个操作将堆中的元素排序，时间复杂度为 O(N log N)。

### 堆操作示例
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 9, 2};

    // 使用 make_heap 函数将 vec 转换为最大堆
    std::make_heap(vec.begin(), vec.end());

    // 输出构建后的最大堆
    std::cout << "After making heap: ";
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    // 使用 push_heap 插入一个新元素
    vec.push_back(8); // 先将新元素添加到 vector 末尾
    std::push_heap(vec.begin(), vec.end()); // 然后调整堆
    std::cout << "After pushing 8: ";
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    // 使用 pop_heap 移除堆顶元素并调整堆
    std::pop_heap(vec.begin(), vec.end());
    vec.pop_back(); // 实际移除堆顶元素
    std::cout << "After popping the max: ";
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    // 使用 sort_heap 对堆进行排序
    std::sort_heap(vec.begin(), vec.end());
    std::cout << "After sorting heap: ";
    for (int i : vec) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

## 七. 数学运算

> **p.s.** 这些函数主要定义在`<numeric>` 中!

### std::accumulate
用于计算序列的累加和. 第三个参数可选, 如果提供则作为初始值.
	`int sum = std::accumulate(v.begin(), v.end(), 0);`

### std::iota
填充一个序列, 将序列的每个位置设置为递增的值, 可指定起始值:
```cpp
std::vector<int> v(5);
std::iota(v.begin(), v.end(), 10); // 填充序列，从10开始递增
for (int i = 0; i < v.size(); ++i) {
    std::cout << v[i] << ' '; // 输出: 10 11 12 13 14
}
std::cout << std::endl;
```

*更多数学运算的函数这里不再赘述.*



