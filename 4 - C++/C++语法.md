[TOC]

## 一、经验总结

1. 代码出现bug调试，有多层复杂的函数调用，调试的时候，直接到最后的地方，逐步查找问题

2. 程序出现问题要调试运行，不要直接运行

3. 读代码思维要清晰，不能急躁

4. 复杂的函数中，最好给每个声明的变量添加一个注释

5. 当程序出现诡异问题的时候，最好重新编译一下代码

   

## 二、指针

### 1. 函数指针

### 2. 普通指针

  示例：

```c++
int (*p_func)(int, int);        //声明
int func (int num_1, int num_2){...}
p_func = func;
```

成员函数指针：

```c++
int (*ClassName::p_func)(int, int);
int ClassName::cacl (int num_1, int num_2){...}
```


### 3. 智能指针

​	**1) shared_ptr**

​		作用跟指针一样，但是不用自动销毁创建的对象，它会记录共有多少个shared_ptr指向这个对象，一旦当某个对象的引用计数，变为0，那么这个对象就会被自动销毁。

​		使用方法: 

```c++
std::shared_ptr<classname> ptr = new classname();
std::shared_ptr<classname> ptr = std::make_shared<classname>();
```



​	**2) unique_ptr**

​	**3) weak_ptr**



## 三、引用

​    引用为对象起了另外一个名字，在变量初始化时，初始值会被拷贝到新建的对象中，而定义引用时，程序则把引用和初始值绑定过在一起，而不是将初始值拷贝给引用。

<font size=5px>**1. 引用作为函数形参**</font>

* 如果一个引用作为函数参数，实参为一个函数的返回值，那么这个返回值也要是引用
* 如果一个引用作为函数形参，那么实参不要是临时变量，如果实参为临时变量，那么形参要为常引用

<font size=5px>**2. 引用的生命周期**</font>

* 引用的生命周期与它绑定的变量有关

* 一个函数无须修改参数的值时，最好将其定义成常引用

<font size=5px>**3. 引用的生命周期**</font>

​	1) C++98只有左值引用，C++11新增了右值引用

​	2) 相同点：
​    		--都属于引用类型
​    		--都必须在声明的时候初始化

​	3) 不同点：
​    		--左值引用是具名变量的别名，右值引用是匿名变量的别名

​	4) 意义
​    		延长临时变量的生命周期，避免临时变量传值时拷贝与析构的消耗。

​	5) 用法
​    		int&& num = getNum();



## 四、类的继承

### 1. 基本概念

* 基类(父类)：被继承的类	class Base{...}
* 派生类(子类)：继承的类	class derive : public Base{...}

* 一个派生类可以同时有几个基类，叫多继承，只有一个基类叫单继承
* 派生类继承基类中除构造函数和析构函数以外的所有成员



### 2. 调用父类虚函数

​	示例：

```c++
class A
{
	virtual void foo(){...}
}
class B : public A
{
	virtual void foo()
	{
		A::foo();
	}
}
```


### 3. 构造和析构

如果基类中没有默认构造函数，那么派生类的构造函数中必须调用基类的构造函数以初始化基类的成员变量。

 * **构造函数的执行顺序：**

    * 基类的构造函数(若有多个基类，则按照它们被继承时声明的顺序，从左往右)
    * 内嵌子对象的构造函数(若有多个按照它们声明的顺序)
    * 自己的构造函数

 * **派生类构造函数的语法：**

   ​	类名::类名(参数总表) : 基类名(参数表1), 基类名(参数表2), ... , 内嵌子对象(参数表1), ... , 成员变量(参数)

 * **析构函数的执行顺序**

   ​	**与构造函数顺序相反**

 * **虚析构**

    ​    普通的析构函数不会被派生类继承，但虚析构函数不同，它与其他虚函数一样，也会被继承

    ​	通常基类的析构函数都应该定义为虚函数，以防止内存泄露的情况，以下为例：

    ```C++
    class A
    {
    	~A(){};
    };
    class B : public A
    {
    	~B(){};
    };
    A* a = new B();
    delete a;
    a = NULL;
    //如果delete的是父类的指针，那么只会执行父类的析构函数，如果是子类的指针，那么它既会执行子类的析构函数，也会执行父类的析构函数
    ```

 * **构造函数的隐式类型转换**

   如果一个构造函数只有一个参数(或者除了第一个参数外后面的参数都有默认值)，那么它除了是个构造器外，还是一个 默认隐式类型转换符。
   
   示例：
   
   ```c++
   class A
   {
   	A(std::string str){...}
   };
   A a = "string";		//在这里会发生隐式类型转换
   ```
   



### 4. 虚函数&多态

 * **虚函数**

   * 定义：用virtual修饰的函数叫做虚函数

   * 哪些函数**不能成为虚函数**

     * **普通函数**和**全局函数**不能是虚函数，虚函数必须是一个类的成员函数
     * **静态成员函数**不能是虚函数，因为虚函数是基于对象的，而静态成员函数是基于类的
     * **内联函数**不能是虚函数，如果用virtual修饰内联函数，那么inline会被忽略
     * **构造函数**不能是虚函数
     * **友元函数**不能是虚函数，友元函数不属于类的成员函数，不能被继承，所以不能为虚函数

   * 普通成员函数在编译期确定调用的函数，而虚函数到了创建对象的时候才能确定绑定的虚函数地址

     

 * **纯虚函数**

   纯虚函数是一种特殊的虚函数，只定义函数名，没有具体的实现，而交给派生类去完成实现

   声明形式如下：

   ```c++
   class A
   {
   	virtual bool Func() = 0;
   }
   ```

   包含纯虚函数的类叫做抽象类，只是作为基类为派生类服务，不能声明对象。

   

 * **虚函数表**

   * 若一个类中有虚函数，编译器会自动为该类维护一个虚函数表，来存放virtual成员函数，该类对象的首地址向后4个字节的内容就是该类虚函数表的地址，同类型的对象共用同一个虚函数表。
   
   * 注意：virtual只能修饰普通成员函数(属于对象)，不能是静态函数(属于类)，而虚函数表又是属于类的。

     

 * **多态的实现**

     *   静态多态和动态多态

         *   静态多态：程序在编译阶段确定调用哪个函数，也称早绑定(重载就属于早绑定)
         *   动态多态：发生在运行时，也称晚绑定，主要通过虚函数来实现

     *   实现方式

         *   宏定义：在编译阶段确定
         *   重载(函数重载和运算符重载)：在编译阶段确定
         *   虚函数：在运行阶段确定

     *   实现原理

         对于带有虚函数的类，在数据区会有一个与之对应的虚函数表，如果是同一类型的对象，虚函数指指向同一地址。父类和子类都会有虚函数表，虚函数指针是在对象构造函数调用的时候生成的。把子类对象给父类指针时，由于虚函数指针只是属于对象的，所以父类指针中的虚函数指针是子类的，所以在调用虚函数时根据这个指针在虚函数表里面找出来指向的是子类的虚函数。

     

### 5. 抽象类

* 类中包含有纯虚函数的类叫做抽象类，这种类只能作为基类为派生类服务，不能声明具体的对象

* 如果派生类继承的基类是抽象类，而没有实现基类全部的纯虚函数，则派生类也将变成抽象类，不能声明对象

  

### 6. 函数的重载、重写(覆盖)和隐藏

* **重载**

    * 定义：在同一作用域下，函数名相同，参数列表不同的函数形成重载，与返回值类型没有关系
    * 说明：若一个重载版本前面有virtual修饰，代表它是虚函数的同时，也属于重载的一个版本

* **重写**

  * 定义：在不同作用域下(基类和派生类)，加virtual修饰，函数名、参数列表相同，返回值也相同(有一种特殊情况)的函数形成重写。

  * 说明：不能是static，只能是virtual
    			重写函数的访问属性可以不同(后面有详细介绍)
          			若只是函数名相同，参数列表或返回值不同，则会发生隐藏，不属于重写。
    
  * 特殊情况：**有一种特殊情况下，重写的返回值类型可以不同**
    
    就是当基类的函数返回基类类型的指针或引用，同时派生类函数返回派生类类型的引用或指针，此种情况下即使返回值类型不同也可以实现覆盖。例：
    
    ```c++
    class A
    {
    	virtual A* func(int num){}
    }
    class B : public A
    {
    	virtual B* func(int num){}
    }
    ```
    
    

  

 * **隐藏**

    * 定义："隐藏"指的是派生类的函数屏蔽了基类与其同名的函数		

    * 说明：当派生类函数和基类函数名字相同而参数列表不同时，不管有无virtual, 都将发生函数的隐藏

      ​			当派生类函数和基类函数名字相同且参数列表相同时，没有virtual将发生函数的隐藏，有发生覆盖

    * 总结：只要子类有跟父类同名的函数，且没有virtual修饰，就会发生隐藏

         ​			子类有跟父类同名但参数列表不同的函数，即使有virtual修饰，也会发生隐藏



## 五、类的仿控属性

### 1. public

### 2. private

### 3. protected



## 六、函数

### 1. 函数基础

### 2. 函数执行流程

### 3. 友元函数

* 定义：定义在类外部，声明在类内部，不是类的成员函数，却有权访问类的所有私有和保护成员的函数

* 使用说明：

  ```c++
  void printWidth(Box box){...}
  class Box
  {
  public:
  	friend void printWidth(Box box);
  	void setWidth(double width);
  private:
  	double width;
  }
  ```

  

### 4. 内联函数



## 七、boost

### 1. 回调函数

​	使用方法：

​	fileA.cpp：

```c++
class A
{
	void SetCallBack(boost::function<void(int)> callback);
    void A::RunCallBack(int num);
	boost::function<void(int)> m_callback;
}

void A::SetCallBack(boost::function<void(int)> callback)
{
	m_callback = callback;
}
void A::RunCallBack(int num)
{
	m_callback(num);
}
```


​	fileB.cpp：

```c++
class B
{
	void OnCallBack(int);
	A m_a;
}
m_a.SetCallBack(boost::bind(&B::OnCallBack, this, _1));
```


## 八、容器的使用

## 九、预处理指令



## 十、一些常用的关键字

### 1. 友元

​	对于普通函数，是不可能访问类的保护成员的。但是用friend来声明一个友元函数，就可以随意的访问操作它了。简单地说，声明一个友元函数或者友元类，就是把自己完全暴露给对方。

* 优点：使得普通函数直接访问类的保护数据，避免了类成员函数的频繁调用，节约处理器开销，提高程序效率

* 缺点：破坏了类的封装特性。

* 普通函数作为友元函数：

  ```c++
  //在类里面声明友元函数
  class MyClass
  {
  public:
  	friend void Show(MyClass& class);
  protected:
  	std::string m_name;
  	private:
  	int num;
  }
  
  //在类外部定义友元函数
  void Show(MyClass& class)
  {
  	std::cout << class.m_name << " : " << class.num << std::endl;
  }
  ```

  

### 2. static静态

* **静态成员变量**

  ​    如果一个变量属于类，而不属于任何独立的对象(也就是说所有类对象的该成员变量都一样)，该变量就可定为静态成员变量。

  ​    静态成员变量的初始化，在类外(首先类的内部要有声明)：

  ```c++
  //变量类型 类名::变量名 = 值
  class A
  {
      static int m_num;
  }
  int A::m_num = 5;
  ```

  

* **静态成员函数**

  ​	如果一个函数是一个不用向对象实时操作的函数时，就可以定义为静态成员函数。

  ​	调用示例：

  ```c++
  //类名::函数名(参数)
  class student
  {
  	static std::string GetOccupation(){return "student";}
  }
  
  std::string occupation = student::GetOccupation();
  ```
  
  ​	静态成员函数不可以访问非静态成员变量，因为静态成员函数不属于任何一个对象，没有this指针，而非静态成员必须随着对象的产生而产生，所以不可以访问。
  
  ​	但如果静态成员函数通过引用一个对象，是可以直接访问成员的，也体现了它成员函数的特权，示例如下：

  ```c++
  class A
  {
  private:
  	int m_i;
  publid:
  	static void func()
  	{
  		m_i := 666;		//错误，这个等价于this->m_i = 666，而静态函数没有this
  	}
  	static void func(A& a)
  	{
  		a.m_i = 666;	//正确
  	}
  }
  ```
  
  
  
* **静态全局变量**

    又名全局静态变量，表示该变量只能在当前声明这个变量的源文件中使用。

    

* **静态局部变量**

  ​	当需要函数中的局部变量在函数调用结束后不消失而保留其原值(其占用的内存资源不释放)，在下一次调用函数时，该变量保留上一次函数调用结束时的值，该变量就可定义为静态局部变量。

  ```c++
  void fun()
  {
  	static int times;
  	times++;
  }
  ```

  

### 3. const

* **用Const修饰的变量必须在声明的时候初始化**

* **顶层const和底层const**

  * 顶层const：表示指针本身是一个常量

  * 底层const：表示指针所指的对象是一个常量

    例子：

    ```C++
    int i = 0;
    int* const p1 = &i;              //顶层const(p1的值不可改变)
    const int ci = 42;               //顶层const(ci的值不可改变)
    const int* p2 = &ci;             //底层const(p2的值可以改变)
    const int* const p3 = p2;        //左边的是底层const，右边的是顶层const
    const int& r = ci;               //底层const，用于声明引用的const都是底层const
    ```

    

* **const引用**

  * const引用可以引用非const对象，但const对象必须用const引用来引用

  * 用const引用来引用非const对象，不能通过引用来修改它的值，但仍可以对对象本身进行赋值操作

  * 临时变量不能给非常引用初始化，但可对const引用初始化

    ```c++
    int& num = 5;          //错误
    const int& num = 5;    //正确
    ```

    

* **const和指针**

  * 指针符号*在const前面，表示const修饰的是指针本身，指针所指向的地址不能改变(顶层const)

  * 指针符号*在const后面，表示const修饰的是指针所指向的地址的内容，指针所指向地址的内容不可通过指针修改，但如果此地址的变量为非const，仍可以对象本身进行赋值操作    

    ```c++
    int number = 5;
    int const *num = &number;
    *num = 6;		//错误，不可通过const修饰的指针改变它所指向地址的变量
    number = 6;		//正确，number本身没有用const修饰，可以修改它的值
    ```

    

* **const在各位置的用处**

  * 函数参数列表后、函数体前：

    * 注意：此种情况，只能修饰类的成员函数

    * 表示不能再函数体内修改类的成员变量
    * 被const修饰的成员函数，只能调用被const修饰的成员变量

  * 函数形参：

    * 表示该参数是传入参数，无法修改它的值
    * 若给函数的实参是临时变量，那么形参要么不用引用，要么用const修饰的常引用
    * 注：临时变量是由编译器在字面常量值、计算表达式、类型转换、函数返回值等情况下产生的。

  * 普通变量：表示该变量为常量，值无法被修改

  * 指针：

    * 顶层指针：指针本身不可修改 (即指针所指向的地址不可再被修改)
    * 底层指针：指针所指向地址的值不能通过该指针修改 (只是不能通过该指针修改)



### 4. typeid

* 包含于头文件<typeinfo>

* RTTI机制为每个类型产生一个type_info类型数据，可以用typeid查询一个变量的类型

  ```c++
  class A;
  A a;
  typeid(a).name();            	//类型名字
  typeid(a).hash_code();        //类型唯一哈希值
  ```

  

* 在所有情况下，typeid都忽略CV限定符，即：typeid(T) == typeid(const T)

* 当typeid的操作数是NULL指针时，typeid运算符将引发bad_typeid异常：

  ```c++
  A* a = NULL;
  try
  {
  	std::cout << typeid(a).hash_code << std::endl;
  }
  catch(bad_typeid)
  {
  	std::cout << "Object is null!" << std::endl;
  }
  ```



### 5. explicit

​	可以阻止不应该允许的经过转换构造函数进行的隐式转换的发生。声明为explicit的构造函数不能在隐式转
换中使用。

​	如在第四章3.6里面提到的构造函数的隐式类型转换，如果如果把代码换成如下，那么编译就会报错:

```c++
class A
{
	explicit A(std::string str){...}
};
A a = "string";	//编译报错，因为explicit关键词禁止了经过转换构造函进行的隐式转换
```


### 6. template

​		模板是泛型编程的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

* **函数模板**

    定义形式：

    ​	template<typeName T>

    ​	*RetType* *FuncName*(param list...)

    实例：

    ```c++
    template<typename T>
    T const& max(T const&a, T const& b)
    {
        return a>b?a:b;
    }
    
    int i = 20;
    int j = 15;
    std::cout << "max(i,j): " << max(i,j) << std::endl;
    ```

    

* **类模板**

    定义形式：

    ​	template<class type>

    ​	class *ClassName*{...};

    实例：

    ```c++
    template<class T>
    class Stack
    {
    public:
        void push(T const&){...}
        void pop(){...}
        T top() const{...}
        bool empty() const{return elems.empty();}
    
    private:
        std::vector<T> elems;
    }
    
    //...
    
    try{
        Stack<int> stack_int;
        Stack<std::string> stack_string;
        
        stack_int.push(7);
        std::cout << stack_int.top() << std::endl;
        
        stack_string.push("hello");
        std::cout << stack_string.top() << std::endl;
    }catch(exception const& ex){
        std::cerr << "Exception: " << ex.what() << std::endl;
        return -1;
    }
    ```

    

* **函数模板 -- 参数是函数指针**

    定义函数模板：

    ```c++
    // _Condi是一个自定义的类
    template<class _Condi, class _Predicate>
    void Wait(_Condi _condi, _Predicate _pred, int mode)
    {
    	bool flag = false;
    	while(!flag)
    	{
    		if(mode == 1)
    		{
    			bool condi = _condi->Condi();
    			flag = _pred(condi);
    		}
    		else if (mode == 2)
    		{
    			int point = _condi->Point();
    			flag = _pred(point);
    		}
    		Sleep(1);
    	}
    	_condi->Reset();
    }
    ```

    定义要传递的函数：

    ```c++
    bool Pred1(bool condi)
    {
    	return condi;
    }
    bool Pred2(int point)
    {
    	return point > 5;
    }
    ```

    定义Condi类：

    ```C++
    class MyCondi
    {
    public:
        MyCondi(){}
    
        void SetCondi(bool flag)
        {
            std::lock_guard<std::shared_mutex> lk(m_mux);
            m_condi = flag;
        }
        bool Condi(){return m_condi;}
    
        void PointInc()
        {
            std::lock_guard<std::shared_mutex> lk(m_mux);
            m_point++;
        }
        int Point(){return m_point;}
    
        void Reset() {m_point=0; m_condi=false;}
    
    private:
        template<class _Predicate>
        void wait(_Predicate _Pred)
        {
            while(!_Pred())
                Sleep(1);
        }
    
    public:
        void Wait1()
        {
    		//wait([this]{return Pred1(this->m_condi);});
            wait( [this]{return (this->m_condi);} );
            m_condi = false;
        }
    
        void Wait2()
        {
            wait([this]{return Pred2(this->m_point);});
            m_point = 0;
        }
    
    private:
        bool m_condi = false;
        int m_point = 0;
        std::shared_mutex  m_mux;
    };
    ```

    使用：

    ```c++
    MyCondi* condi = new MyCondi();
    
    void WaitThread()
    {
        while(true)
        {
            std::cout << "[Waiting...]" << std::endl;
            condi->Wait1();
    //        condi->Wait2();
    //        Wait(condi, Pred1, 1);
    //        Wait(condi, Pred2, 2);
            std::cout << "[Wait Success!]" << std::endl;
            std::cout << "==========================" << std::endl;
        }
    }
    
    void GrowThread()
    {
        static int count = 0;
        while(true)
        {
            ////////// [1] //////////
            Sleep(1000);
            count ++;
            std::cout << "[Growing...]" << std::endl;
            if(count >= 6)
            {
                condi->SetCondi(true);
                count = 0;
            }
    
            ////////// [2] //////////
    //        Sleep(1000);
    //        std::cout << "[Growing...]" << std::endl;
    //        condi->PointInc();
        }
    }
    ```

    

* **函数模板 -- 参数是lambda表达式**

    写函数模板：

    ```C++
    template<class _Predicate>
    void wait(_Predicate _Pred)
    {
    	while(!_Pred())
    		Sleep(1);
    }
    ```

    使用：

    ```C++
    wait([this]{return (this->m_condi);});
    ```



### 7. extern

*   在变量的声明和定义中

    ​	C++语言支持分离式编译机制，为了将程序分为多个文件，需要在文件中共享代码，比如一个文件中的代码可能需要用到另一个文件中定义的变量。

    ​	为了支持分离式编译，C++允许将声明和定义分离开来，变量的声明规定了变量的类型和名字，即使一个名字为程序所知，一个文件如果想使用别处定义的名字则必须包含对那个名字的声明。

    ```c++
    extern int i;	// 声明i，不定义
    int i;			// 声明并定义i
    ```

    ​	我们也可以给extern关键字标记的变量赋值一个初始值，但这样就不是声明而是定义了，extern的作用将被抵消。

    ```c++
    extern int v = 2;
    int v = 2;			// 这两个语句的效果完全一样，都是v的定义
    ```

    声明和定义的区别：

    -- 声明使得名字被程序所知

    -- 定义负责创建与名字关联的实体

    -- 变量可以被声明多次，但只能被定义一次

    

*   在多个文件中共享const

    ​	默认情况下，一个const对象仅在本文件内有效，如果多个文件中出现了同名的const变量时，其实是在多个文件中分别定义了独立的同名的变量。

    ​	有时我们希望在一个文件里面声明的const对象，被多个文件使用，方法是对于const变量不管是声明还是定义都添加**extern**关键字，这样只需要定义一次就可以了

    

*   模板的控制实例化

    

*   extern "C"

    这种写法是在告诉编译器，按照C语言编译器的方式生成函数调用名。

    

*   总结

    从上面总结，extern的作用总的来说无非就是两种：

    *   用extern修饰目标，为了文件共享。被extern修饰的目标，要么是其它文件中已经定义好的，要么是让声明目标让其它文件使用。
    *   extern "C" 函数声明，告诉编译器按照C语言编译器的方式生成函数调用名。





## 十一、一些实用的标准库函数

### 1. memset

### 2. strcpy和memcpy



## 十二、类型转换

一般情况下，数据的类型的转换通常是由编译器自动进行的，不需要人工干预，所以被称为隐式类型转换。但如果程序要求一定要将某一类型的数据转换为另外一种类型，则可以利用强制类型转换运算符进行转化，这种转换称为显示转换。



### 1. 隐式类型转换

##### 概念

值不需要用户干预，编译器私下进行的类型转换行为。很多时候用户可能都不知道进行了哪些转换。



##### 基本类型的隐式转换

char -> short -> int -> unsigned -> long -> float -> doule

发生条件：

	* 混合类型的算术运算表达式
	* 不同类型的赋值操作
	* 函数参数传值
	* 函数返回值



##### 构造函数的隐式转换

如果一个**构造函数只有一个参数**，那么它除了是个构造器之外，**还是一个默认隐式类型转换符**。

```c++
class A
{
	A(std::string str){...}
};
A a = "string";		// 这里会发生隐式类型转换

class B
{
	explicit B(std::string str){...}
};
B b = "string";		// 这里会报错，因为explicit禁止了经过转换构造函数进行的隐式转换
```



### 2. 显示类型转换

##### static_cast

##### dynamic_cast

##### const_cast

##### reinterpret_cast



## 十三、编译问题



## 十四、补充介绍

### 1. RTTI问题

##### 简介

RTTI全程 RunTime Type Identification，这个机制是用来记录类型信息的，typeid或dynamic_cast等操作符会用到这些信息。

##### 问题原因

当代码中dynamic_cast或者typeid等需要typeinfo的操作符时，会要求有RTTI。而通常编译C++代码的时候，默认是自带RTTI机制的，所以一般情况下我们不用去管这个问题。

但是当工程中导入的第三方库是由ndk编译生成的时候，由于ndk默认并不带有RTTI机制，所以在库中用到了dynamic_cast操作符时，就会报如下的错：

​	<font color=red>**undefined reference to 'typeinfo for xxx'**</font>

**说明**：如果你的项目中，调用的库中的类或者函数没有涉及到dynamic_cast或typeid的这些代码块时，也能顺利编译通过，但这是一个潜在的问题，如果以后添加了相关的代码，还是会报错。

##### 注意事项

*   导入的库和工程对于是否带RTTI要保持一致，如果混合了，同样编译不过
*   如果代码中对库的类使用了dynamic_cast或typeid等需要typeinfo的操作时，则必须带RTTI



