
**磁盘格式化的本质是给磁盘写入文件管理信息**

## 基本概念
1. 分区: 
2. 块: 
3. inode:

## Block Group结构

```
| super | GDT | block  | inode  | inode | Data   |
| block |     | bitmap | bitmap | table | blocks |
```

1. super block: 包含整个组的inode数量, block数量, 以及其他的重要属性信息. 并且一个组可能拥有多个super block作为备份, 防止super block损坏导致整个组损坏. 
2. GDT: 
3. block bitmap: 记录哪些块已经使用, 哪些块未使用
4. inode bitmap: 同上
5. data blocks: 数据块, 实际存放数据. 
6. 一个分区包含多个Block Group

## inode
1. 一个文件对应一个inode
2. 一个inode可以指向多个数据块. inode struct中含有文件的数据块直接块指针数组, 一级间接指针, 二级间接指针, 三级间接指针. 
3. inode中包含文件的属性信息
4. 但inode中不包含文件名. 因为不同长度的文件名会影响inode的对齐. 
5. **inode number在整个分区内唯一.** 所以inode可以在整个分区内寻址, 而不是只能在块组内分配data block.
6. **如果block group的inode已经用尽, 但是data block没有用尽, 其他块组的inode也可以使用这个块组的data block.** 一般采用就近分配, 但不要求必须在同一块组内. 

## 文件夹(目录)
1. 文件夹本质也是文件
2. 文件夹内部保存的是文件名与inode的映射关系. 故文件名实际保存在文件夹中. 
3. 打开当前文件夹下的一个文件需要获取该文件的inode, 进而需要获取文件夹的inode, 所以需要从根目录递归解析文件夹. 

### 路径缓存
1. 并非每次访问文件夹中内容都需要递归解析. 为了解决递归解析过慢的问题, Linux引入了路径缓存
2. 路径缓存是一颗多叉树, 在kernel中实现为struct_dentry. 多叉树同时支持Hash
3. 系统会把常用的/打开次数多的文件/文件夹挂上树. 下一次访问文件时优先在树中寻找
4. 这个多叉树实质上是文件树的子集. 