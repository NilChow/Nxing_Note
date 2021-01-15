[TOC]

# function、bind

### std::bind

_1标识符在std::placeholder中，如果只想写 _1，using namespace std::placeholsers



# 容器

除了list，vector和deque的pop等函数不会自动释放内存，需要手动释放(当容器里存放的是指针对象，并且指向的是new出来的地址的时候)

### 容器介绍

C++中有两种类型的容器，顺序容器和关联容器。

容器自动申请和释放内存，我们无需new和delete操作。(注意是释放容器内元素的内存，而不是容器本身占用内存)

顺序容器：

*   vector：表示一段连续的内存地址
*   deque：与vector类似，但是对首元素提供删除和插入的双向支持
*   list：表示非连续的内存，基于链表实现

关联容器：

*   map：key-value形式
*   set：单值形式
*   multimap、multiset：同于map和set，区别是可以存放多个相同的key值



### vector

##### 简介

vector是将元素放到动态数组中加以管理的容器。可以随机存取元素，也就是说支持 _[ ]_ 运算符和 _at_ 方式存取。vector在尾部添加或移除元素非常快，但是在中间操作非常耗时，因为它需要移动元素。

##### 初始化

vector有四种初始化方式：

*   构造函数初始化：

```c++
// 直接构造函数初始化
std::vector<int> v1;
v1.push_back(1);
v1.push_back(2);
```

*   拷贝构造函数初始化

```c++
// 通过拷贝构造函数初始化
std::vector<int> v2 = v1;
```

*   使用部分元素初始化

```c++
// 使用部分元素初始化
std::vector<int> v3(v1.begin(), v1.begin()+1);
std::vector<int> v3(v1.begin(), v1.end());
```

*   使用构造函数：

```c++
// 存放三个元素，每个元素值都为9
std::vector<int> v5(3, 6);
```

##### 插入数据

*   insert

```c++
insert(iter, 5);		// 在iter前面添加1条数据，值为5
insert(iter, 3, 6);		// 在iter前面添加3条数据，值为6
```

*   emplace

```c++
emplace(iter, 5);		// 在iter前面添加1条数据，值为5(C++11新特性，效率比push_back高)
```

*   push_back

```c++
push_back(5);			// 在容器尾部添加1条数据，值为5
```

*   emplace_back

```c++
emplace_back(5);		// 在容器尾部添加1条数据，值为5(C++11新特性，效率比push_back高)
```

##### 删除数据

*   erase

    **注意**：erase的时候要用iter接收返回值，否则iter会变成野指针，返回值时下一个有效的迭代器

```c++
iter = vec.erase(iter);	// 移除iter位置的元素
```

*   pop_back

```c++
pop_back();				// 移除尾部元素
```

##### 遍历

*   旧式写法

```c++
std::vector<int>::iterator iter = vec.begin();
for(iter; iter != vec.end(); iter++)
{
	//...
}
```

*   C++11新特性

```c++
for(auto& temp : vec)
{
	//...
}
```

##### 访问

```c++
vec[2];			// 通过下标访问元素。若越界，不会抛异常，可能会崩溃，或出现随机数据
vec.at(2);		// 通过下标访问元素。若越界，会抛异常，需要捕获异常并处理
vec.front();	// 返回头部元素的引用，可以当左值
vec.back();		// 返回尾部元素的引用，可以当左值
```

##### 容器信息

```c++
vec.capacity();	// 返回当前vector在重新申请内存前最多可容纳的元素个数
vec.data();		// 返回vector第一个元素的指针
vec.empty();	// 判断vector是否为空
vec.max_size();	// 返回极限情况下，容器所能容纳的最大元素的个数
vec.size;		// 返回当前vector的元素个数
```

##### 其它常用操作

```c++
vec.reserve()
vec.resize()
vec.swap()
vec.clear()
vec.shrink_to_fit()
```

##### 内存

*   若容器内存放的是指针，而指针指向的对象是手动申请的，需要在合适的地方释放(最好在删除元素前释放)
*   将非指针元素从容器移除时，会自动调用析构函数，但需注意，容器申请的内存还是不变(也就是说容器占用的内存不变)
*   任何情况下，vector容器的内存只能增，不能减，除非手动shrink_to_fit()



### list

### deque

### map

### multimap

