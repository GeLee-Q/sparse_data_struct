# **CMake 工程构建**

- src        源码文件
- mylib    包含源码所需要的头文件
- sparse_data_structure 稀疏数据结构代码
- 顶层CMakeLists.txt 构造项目

# 康威声明游戏优化

## 代码环境

> WSL + GCC 9.2

> Parallel_Computing_Practice/src/Conway_Game_of_Life.cpp

## 实验优化记录

- 原始代码未开启openMP

```
134.008686s
```

- 原始代码开启openMP

```
49.100373s        加速：2.72x
```

- 使用指针数组这种稀疏数据结构来优化

```
OpenMp  29.86s    加速：4,48x
tbb     17.48s    加速：7.66x
```

- 在指针数组加上spin_mutex优化，效果不明显

```
 openMP 33.47s    加速：4.06x
 tbb    17.76s    加速：7.54x
```

## 优化**方法总结**

**封装稀疏网格的 Grid 类的？**

- 采用指针数组封装稀疏的Grid表格
- 为了减轻mutex的系统开销，使用了spin_mutex自旋锁
- 如果使用hash().pointer().dense的话，在WSL上回出现 out of memory

**使用位运算量化减轻内存带宽？**

- & 替代 %， >> 替代 /， | 替代 +

**使用并行访问访问库进行优化？**

- 使用OpenMP进行并行访问。

**有没有用访问者模式缓存坐标，避免重复上锁？**

- 没有使用, 在WSL里使用这个，会莫名出现segment fault

**`step()` 函数中这种插桩方式优化**

- 将step()改造成了tbb并行。