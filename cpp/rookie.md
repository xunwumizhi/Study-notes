# 标准库STL

```cpp
```

# 基本语法

- 类、对象、方法

- 头文件

- 命名空间 namespace

- 分号、代码块


# 保留关键字

C++ 基本语法 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-basic-syntax.html

- typedef

定义类型
```cpp
typedef <type> new_name;
// eg
typedef int feet;
```

- extern / static / mutable / thread_local

声明变量,static 同个文件

- friend, inline 类定义相关


# 数据类型

C++ 数据类型 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-data-types.html

## 指针与引用

```cpp
int i = 17;

int *ptr;
ptr = &i;

// 引用变量是 某个已存在变量 的另一个名字
int &ref = ptr;
```

## 结构体struct
```cpp
struct Book
{
    int id;
    string title;
};
```


## 函数func

```cpp
// return_type function_name( parameter type list );

// 声明
int max(int num1, int num2);
// 定义
int max(int num1, int num2) 
{
   if (num1 > num2)
   {
      return num1;
   }
   else
   {
      return num2;
   }
}
```

- 函数形参

1. 值传递

2. 指针传递
```
void swap(int *x, int *y)
{
   int temp;
   temp = *x;    /* 保存地址 x 的值 */
   *x = *y;        /* 把 y 赋值给 x */
   *y = temp;    /* 把 x 赋值给 y */
  
   return;
}
```


## 数组
```cpp
// <type> arrayName [ arraySize ];
int arr[5];
double balance[5] = {1000.0, 2.0, 3.4, 7.0, 50.0};
double balance[] = {1000.0, 2.0, 3.4, 7.0, 50.0};
```

## 变量与常量

extern

常量
```
#define <identifier> <value>

const <type> <identifier> = <value>;
```


# 运算符
C++ 运算符 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-operators.html

## 特殊运算符
```cpp
sizeof
// 成员运算符
.  // 访问结构体成员
-> // 通过结构体指针访问成员
```

# 流程控制 if switch

```cpp
   for( int a = 10; a < 20; a = a + 1 )
   {
       cout << "a 的值：" << a << endl;
   }
```


# 类

## 构造函数与析构函数

类的构造函数是类的一种特殊的成员函数，它会在每次创建类的新对象时执行。
构造函数的名称与类的名称是完全相同的，并且不会返回任何类型，也不会返回 void。

```cpp
class Line
{
   public:
      Line();  // 这是构造函数
      ~Line(); // 析构函数
      Line(double len);

   private:
      double length;
};

// 成员函数定义，包括构造函数
Line::Line()
{
    cout << "Object is being created" << endl;
}
Line::~Line()
{
    cout << "Object is being deleted" << endl;
}

// 构造函数重载overload，并初始化成员
Line::Line(double len): length(len)
{
    // ...
}
// 等价于
Line::Line(double len)
{
    length = len
    // ...
}
```

## 拷贝构造函数
```
classname (classname &obj);

// 如
class Line
{
   public:
      Line(Line &obj);      // 拷贝构造函数
};
```

## 友元函数

提供访问类 private 和 protected 成员
```cpp
class Box
{
   double width;
public:
   friend void printWidth( Box box ); // 友元函数
   void setWidth( double wid );
};

// 成员函数定义
void Box::setWidth( double wid )
{
    width = wid;
}

// 请注意：printWidth() 不是任何类的成员函数
void printWidth( Box box )
{
   /* 因为 printWidth() 是 Box 的友元，它可以直接访问该类的任何成员 */
   cout << "Width of box : " << box.width <<endl;
}
```

## 内联函数

内联函数是通常与类一起使用。在类定义中的定义的函数都是内联函数，即使没有使用 inline 说明符。

引入内联函数的目的是为了解决程序中函数调用的效率问题，程序在编译器编译的时候，编译器将程序中出现的内联函数的调用表达式用内联函数的函数体进行替换，而对于其他的函数，都是在运行时候才被替代。空间换时间

## 静态成员

static
```cpp
class Box
{
   public:
      static int objectCount; // 声明
      // 声明并定义
      static int getCount()
      {
         return objectCount;
      }
      // 只声明不定义
      static int getCount2();
};
 
// 定义并初始化类 Box 的静态成员
int Box::objectCount = 0;
// 定义静态函数
int Box::getCount2()
{
   return objectCount;
}
```

## 继承

```
// 基类
class Animal {

};

//派生类
class Dog : public Animal {

};
```

## 虚函数

虚函数 是在基类中使用关键字 virtual 声明的函数。在子类中重新定义基类中定义的虚函数时，会告诉编译器不要静态链接到该函数。以便实现多态，运行期间决定

C++ 多态 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-polymorphism.html

```cpp
virtual int area(); // 只声明不定义的函数，称为纯虚函数
```

## 接口(抽象类)

如果类中至少有一个函数被声明为纯虚函数，则这个类就是抽象类。

C++ 接口（抽象类） | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-interfaces.html

## 泛型

C++ 模板 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-templates.html


# 异常处理

C++ 异常处理 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-exceptions-handling.html

```cpp
   try {
     division(x, y);
   }catch (const char *msg) {

   }
```

# 内存分配

C++ 动态内存 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-dynamic-memory.html

```cpp
   double* pvalue  = NULL; // 初始化为 null 的指针
   pvalue  = new double;   // 为变量请求内存
 
   *pvalue = 29494.99;     // 在分配的地址存储值
   cout << "Value of pvalue : " << *pvalue << endl;
 
   delete pvalue;         // 释放内存
```

# 文件组织

## 命名空间

```cpp
#include <iostream>
using namespace std;
 
// 第一个命名空间
namespace first_space
{
   void func(){
      cout << "Inside first_space" << endl;
   }
}
// 第二个命名空间
namespace second_space
{
   void func(){
      cout << "Inside second_space" << endl;
   }
}
int main ()
{
 
   // 调用第一个命名空间中的函数
   first_space::func();
   
   // 调用第二个命名空间中的函数
   second_space::func(); 
 
   return 0;
}
```

## 预处理器

C++ 预处理器 | 菜鸟教程: https://www.runoob.com/cplusplus/cpp-preprocessor.html

```
#include

#define PI 3.1415

#if 0
   /* 这是注释部分 */
   cout << MKSTR(HELLO C++) << endl;
#endif
```