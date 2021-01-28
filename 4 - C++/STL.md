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









# bitset

### 简介

bitset可以用来方便地管理一系列的bit位而不用程序员自己来写代码。除了可以访问指定位置的bit位外，还可以把它们作为一个整数来进行某些统计。



### 构造

```c++
// 默认构造
std::bitset<BIT_SIZE> myset;
std::bitset<4> myset;				// 0000
// 用字符串构造
std::bitset<10> set1("100101");		// 0000100101
// 用十进制数构造
std::bitset<8> set2(12);			// 00001100
// 用八进制构造
std::bitset<32> set3(012);			// 00000000 00000000 00000000 00001010
// 用十六进制构造
std::bitset<32> set6(0xFFFF);		// 00000000 00000000 11111111 11111111
```

**特别注意**：

构造时，如果参数的位数大于了bitset的size时

*   若为数字，取后面size大小的部分
*   若为字符串，取前面size大小的部分



### 访问、修改元素

```C++
myset[FIELDKEY_LASTSETTLE] = 1;		// 可以通过下标访问元素，也可以通过下标修改下标位置的值(不会对下标越界做检查，有风险)
	
myset.set(FIELDKEY_LASTCLOSE);		// 只有一个参数时，将参数位置的值置为1
myset.set(FIELDKEY_LASTCLOSE, 0);	// 有两个参数时，将参数位置的值置为 第二个参数的值(0或者1)
myset.set();						// 没有参数时，将bitset每一位都置为1
	
myset.reset(FIELDKEY_LASTCLOSE);	// 有一个参数时，将参数位置的值置为0 (不接受两个参数)
myset.reset();						// 没有参数时，将bitset每一位都置为0
	
myset.flip();						// 没有参数时，将bitset每一个位的值取反
myset.flip(FIELDKEY_LASTSETTLE);	// 有一个参数时，将参数位置的值取反
	
myset.test(FIELDKEY_LASTSETTLE);	// 检查参数位置的值为1还是0, 1返回true，0返回false
```



### 相关信息

```c++
size_t count = myset.count();		// 0---返回当前值为1的bit的个数
size_t size = myset.size();			// 41---返回当前bitsize的大小
	
bool flag = myset.any();			// 检查bitset中是否有1，,是返回true，否返回false
flag = myset.none();				// 检查bitset中是否没有1，是返回true，否返回false
flag = myset.all();					// 检查bitset中是否全为1，是返回true，否返回false
```



### 二进制操作

```c++
set5 ^= set6;				// set5对set6按位 异或 后赋值给set5
set5 &= set6;				// set5对set6按位  与   后赋值给set5
set5 |= set6;				// set5对set6按位  或   后赋值给set5
	
set5^set6;					// 按位 异或，不赋值
set5&set6;					// 按位  与  ，不赋值
set5|set6;					// 按位  或  ，不赋值
	
set5 <<= 2;					// 左移两位，低位补0，有自身赋值
set6 >>= 1;					// 右移一位，高位补0，有自身赋值

set5>>2;					// 右移两位，不赋值
set6<<1;					// 左移一位，不赋值
	
flag = (set5 == set6) ? true : false;		// 可以用 == 或 != 操作符来判断
```



### 类型转换

```c++
std::string str = myset.to_string();			// 将bitset转换成string类型
unsigned long a = myset.to_ulong();				// 将bitset转换成unsigned long类型
unsigned long long b = myset.to_ullong();		// 将bitset转换成unsigned long long类型
```











# 线程相关

### 线程 -- thread







### 锁 -- mutex







### 锁卫士 -- lock_guard







### 条件变量 -- condition_variable

##### 简介

std::condition_variable是条件变量，当它的对象在**调用wait函数后**，它会用std::unique_lock(通过std::mutex)来**锁住当前线程**，当前线程会被阻塞，直到另外一个线程在相同的std::condition_variable对象上**调用notification函数来唤醒当前线程**。

##### wait函数介绍

函数原型：

```c++
template<class _Predicate>
void wait(unique_lock<mutex>& _Lck, _Predicate _Pred)
{	// wait for signal and test predicate
	while (!_Pred())
	wait(_Lck);
}
```

第一个参数 _Lck:

*   如果是unique_lock对象，用它里面的mutex来阻塞线程
*   如果想要其它的lockable类型，则需要使用condition_variable_any类

第二个参数 _Pred:

*   是一个判断条件
*   判断触发条件是否真的达到，若条件为false，则还是会继续阻塞
*   可以传递一个函数指针，也可以传递一个lambda表达式



例子：

```c++
// 传递函数指针
bool IsEnough()
{
	std::cout << "To Check" << std::endl;
    return point > 5 ? true:false;
}

// 传递lambda表达式
std::unique_lock<std::mutex> lk(mux);
condi.wait(lk, [points]{return points>5;});
```



##### wait_for函数介绍

wait_for函数比wait函数多了一个时间参数，可以指定等待的时间，当时间到了(超时)了，即使没有收到notification，也会结束等待，返回

返回值：

*   true：当收到通知，且判定条件_Pred为true时，返回true
*   false：当因超时，或判定条件_Pred为false时，返回false



用法例子：

```c++
#include <chrono>
using namespace std::chrono_literals;

std::unique_lock<std::mutex> lk(mux);
bool res = condi.wait_for(lk, 10000ms, IsEnough);	// 判断返回原因
```



##### notification函数介绍

notification有两个可以通知的函数，notify_one和notify_all：

*   notify_one：
*   notify_all：唤醒所有的等待线程



##### 用法流程

![](..\0 - src\4\condition_variable.png)