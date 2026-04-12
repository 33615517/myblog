
 <div align=center>
<img src=https://i-blog.csdnimg.cn/direct/57d7d59c08fa41b4a868a099a7e89ada.jpeg>
</div>
<div align="center">
  <font color="#D2691E">✨ 把代码写进星轨，<br>用逻辑丈量宇宙。</font>
</div>

<br>


| <font color="#800080">导航</font> | <font color="#800080">链接</font> |
| :--- | :--- |
| **<font color="#800080">个人主页</font>** | [🏠 星轨初途](https://blog.csdn.net/2502_93089837?type=blog) |
| **<font color="#800080">基础语言专栏</font>** | [💻 C语言](https://blog.csdn.net/2502_93089837/category_13078759.html) 、 [📚 数据结构](https://blog.csdn.net/2502_93089837/category_13078761.html) |
| **<font color="#800080">C++ 进阶专栏</font>** | [🏆 C++学习（竞赛类）](https://blog.csdn.net/2502_93089837/category_13094661.html) 、 [⚙️ C++专栏（开发类）](https://blog.csdn.net/2502_93089837/category_13141032.html) |
| **<font color="#800080">刷题实战专栏</font>** | [🚀 算法及编程题分享](https://blog.csdn.net/2502_93089837/category_13078762.html) |




---
@[TOC]

# 前言

嗨！我们又见面啦٩(๑❛ᴗ❛๑)۶！欢迎回到星轨的 C++ 进阶频道！

在上一篇《[C++ 动态内存管理：new/delete 的底层真相与避坑黑科技](https://blog.csdn.net/2502_93089837/article/details/159473830?spm=1011.2415.3001.5331)》中，我们详细剖析了 `new/delete` 的底层逻辑。在实际开发中，我们常常会遇到需要为不同数据类型编写相同逻辑代码的场景。

今天，我们将探讨 C++ 中解决代码复用问题的核心机制——**模板（Template）与泛型编程**。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5d27e16f64634cc28ecc3c2d087fb7a7.jpeg#pic_center)


---

# 一、 泛型编程的引入

## 1. 传统函数重载的局限性
在没有模板的情况下，如果我们需要实现一个通用的 `Swap`（交换）函数以支持 `int`、`double` 和 `char`，通常只能依赖函数重载来实现：

```cpp
void Swap(int& left, int& right)
 {
    int temp = left;
    left = right;
    right = temp;
}

void Swap(double& left, double& right) 
{
    double temp = left;
    left = right;
    right = temp;
}

void Swap(char& left, char& right) 
{
    char temp = left;
    left = right;
    right = temp;
}
```

> **📌 痛点分析：**
> 1. **代码复用率低**：重载的函数仅仅是类型不同，逻辑完全一致。只要有新类型出现，就需要用户自己增加对应的函数。
> 2. **可维护性低**：一个逻辑出错可能导致所有的重载版本均出错，后期维护成本较高。

## 2. 泛型编程的概念
能否告诉编译器一个“模子”，让编译器根据不同的类型利用该模子来自动生成代码呢？

这就像工业中的**“浇筑钢水”**：如果在 C++ 中存在这样一个模具，通过给这个模具中填充不同的材料（类型），就能获得不同材料的铸件（即生成具体类型的代码），将极大提升开发效率。

这就是**泛型编程**的核心思想：**编写与类型无关的通用代码，它是代码复用的一种手段**。而模板正是泛型编程的基础，主要分为**函数模板**和**类模板**。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ffd40cf1aa814cdea035cbcfa69183d9.png)

---

# 二、 函数模板

函数模板代表了一个函数家族，该函数模板与具体类型无关，在使用时被参数化，根据实参类型产生函数的特定类型版本。

## 1. 函数模板格式
使用 `template` 关键字定义模板，并在尖括号 `< >` 内声明模板参数：

```cpp
template<typename T>
void Swap(T& left, T& right)
{
    T temp = left;
    left = right;
    right = temp;
}
```

> **💡 注意事项：**
> `typename` 是用来定义模板参数的关键字，也可以使用 `class` 代替。**但切记：不能使用 `struct` 代替 `class`**。

## 2. 函数模板的底层原理
**函数模板是一个蓝图，它本身并不是函数，而是编译器用使用方式产生特定具体类型函数的模具**。
模板的本质就是将本来应该由程序员做的重复性工作交给了编译器。

在编译器编译阶段，对于模板函数的使用，编译器需要根据传入的实参类型来**推演**生成对应类型的函数以供调用。例如：当用 `double` 类型使用函数模板时，编译器通过对实参类型的推演，将 `T` 确定为 `double` 类型，然后产生一份专门处理 `double` 类型的代码；对于字符类型也是如此。
流程如图所示
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c992e23aae45484491e4f5bf98081711.png)


## 3. 多模板参数（多个泛型类型）

在实际开发中，我们传入的参数类型往往并不完全相同。模板并不局限于单个类型参数，它支持定义多个不同的模板参数，其基础格式为：`template<typename T1, typename T2,......, typename Tn>` 。同样，类模板也支持这样的多参数格式 。

例如，当我们需要处理两个不同类型数据的加法运算时，如果只使用一个 `T`，编译器会因为无法确定唯一的类型而报错。此时，我们可以使用带有 `T1` 和 `T2` 的通用加法函数 ：

```cpp
// 通用加法函数：支持不同类型的参数相加
template<class T1, class T2>
T1 Add (T1 left, T2 right)
{
    return left + right;
}
```

> **📌 细节解析：**
> * **推演独立性**：当调用 `Add(1, 2.0)` 时，编译器会将 `1` 独立推演为 `T1`（即 `int`），将 `2.0` 独立推演为 `T2`（即 `double`），从而完美匹配，不再产生类型冲突报错 。
> * **返回值设定**：在上述代码中，返回值类型被设定为了 `T1` 。这意味着最终的计算结果会向第一个参数的类型进行隐式转换和靠拢。


---

# 三、 函数模板的实例化

用不同类型的参数使用函数模板时，称为函数模板的实例化。模板参数实例化分为：**隐式实例化**和**显式实例化**。

## 1. 隐式实例化
隐式实例化是指：**让编译器根据实参推演模板参数的实际类型**。

```cpp
template<class T>
T Add(const T& left, const T& right) 
{
    return left + right;
}

int main() 
{
    int a1 = 10, a2 = 20;
    double d1 = 10.0, d2 = 20.0;
    
    Add(a1, a2); // 编译器推演出 T 为 int
    Add(d1, d2); // 编译器推演出 T 为 double
    
    // 🚨 编译报错：
    // Add(a1, d1); 
    return 0;
}
```

> **📌 报错原因分析：**
> 该语句不能通过编译。因为在编译期间，当编译器看到该实例化时，需要推演其实参类型。通过实参 `a1` 将 `T` 推演为 `int`，通过实参 `d1` 将 `T` 推演为 `double` 类型，但模板参数列表中只有一个 `T`，编译器无法确定此处到底该将 `T` 确定为 `int` 或者 `double` 类型而报错。
> **注意**：在模板中，编译器一般不会进行类型转换操作，因为一旦转化出问题，编译器就需要担责。

**处理方式 1**：用户自己进行强制类型转换，如 `Add(a1, (int)d1);`。

## 2. 显式实例化
显式实例化是指：**在函数名后的 `< >` 中指定模板参数的实际类型**。

```cpp
int main(void)
 {
    int a = 10;
    double b = 20.0;
    
    // 显式实例化
    Add<int>(a, b); 
    return 0;
}
```
**注意：** 如果类型不匹配，编译器会尝试进行隐式类型转换，如果无法转换成功编译器将会报错。

---

# 四、 模板参数的匹配原则

当一个非模板函数和一个同名的函数模板同时存在时，其调用和匹配遵循以下原则：

1. **可以共存与特化**：一个非模板函数可以和一个同名的函数模板同时存在，而且该函数模板还可以被实例化为这个非模板函数。如果显式指定调用 `Add<int>(1, 2)`，则会调用编译器特化的模板版本。


2. **优先调用非模板函数**：对于非模板函数和同名函数模板，如果其他条件都相同，在调动时会**优先调用非模板函数**而不会从该模板产生出一个实例。
```cpp
// 专门处理int的加法函数
int Add(int left, int right)
{
	return left + right;
}

// 通用加法函数
template<class T>
T Add(T left, T right)
{
	return left + right;
}

void Test()
{
	Add(1, 2); // 与非模板函数匹配，编译器不需要特化
	Add<int>(1, 2); // 调用编译器特化的Add版本
}
```


3. **选择更匹配的版本**：如果模板可以产生一个具有**更好匹配**的函数，那么将选择模板。例如调用 `Add(1, 2.0)`，如果有完全匹配的模板（如 `template<class T1, class T2>`），编译器会根据实参生成更加匹配的 Add 函数版本，而不需要去调用非模板函数并进行类型转换。
```cpp
// 专门处理int的加法函数
int Add(int left, int right)
{
    return left + right;
}


// 通用加法函数
template<class T1, class T2>
T1 Add(T1 left, T2 right)
{
    return left + right;
}

void Test()
{
    Add(1, 2);    // 与非函数模板类型完全匹配，不需要函数模板实例化
    Add(1, 2.0);    // 模板函数可以生成更加匹配的版本，编译器根据实参生成更加匹配的
}
```
4. **自动类型转换限制**：模板函数不允许自动类型转换，但普通函数可以进行自动类型转换。

---

# 五、 类模板

## 1. 类模板的定义格式
与函数模板类似，类模板的定义格式如下：

```cpp
#include<iostream>
using namespace std;

// 类模板
template<typename T>
class Stack
 {
public:
    Stack(size_t capacity = 4)
     {
        _array = new T[capacity];
        _capacity = capacity;
        _size = 0;
    }
    
    // 类内成员声明
    void Push(const T& data);

private:
    T* _array;
    size_t _capacity;
    size_t _size;
};

// ⚠️ 类模板成员函数类外定义规范
template<class T>
void Stack<T>::Push(const T& data)
 {
    // 扩容逻辑省略
    _array[_size] = data;
    ++_size;
}
```

> **📌 重点提醒：**
> 1. **类外定义需限定作用域**：如果是类模板的成员函数要在类外定义，必须再次声明 `template<class T>`，并且在函数名前加上作用域限定符 `Stack<T>::`。
> 2. **分离编译问题**：现阶段，模板不建议将声明和定义分离到两个文件（如 `.h` 和 `.cpp`）中，否则会出现链接错误（具体原因后续探讨）。

## 2. 类模板的实例化机制
类模板实例化与函数模板实例化不同，**类模板实例化需要在类模板名字后跟 `< >`，然后将实例化的类型放在 `< >` 中即可**。

需要严格区分的是：**类模板名字不是真正的类，而实例化的结果才是真正的类**。

```cpp
int main()
 {
    // Stack 是类名，Stack<int> 才是类型
    Stack<int> st1;    // 实例化为 int 类型的栈
    Stack<double> st2; // 实例化为 double 类型的栈
    return 0;
}
```

---

# 结束语

嗨！٩(๑❛ᴗ❛๑)۶本篇到这里就结束啦！通过泛型编程与模板（Template）机制，C++ 将类型相关的工作转移给了编译器，极大地提升了代码的复用率和开发效率。

本篇所讲的函数模板与类模板只是泛型编程的基石。正是基于模板机制，C++ 才构建出了强大的 **STL（标准模板库）**。在后续的文章中，我们将进一步深入探讨模板进阶及 STL 的底层实现。下一篇我们将讲解string的相关内容，欢迎大家来欣赏！

**把代码写进星轨，用逻辑丈量宇宙。如果本文对您有帮助，欢迎点赞、收藏与关注支持！我们下期再见！** ヾ(◍°∇°◍)ﾉﾞ


<img src="https://i-blog.csdnimg.cn/direct/53fb210330534141bfbb7f1fc94f7736.jpeg" width="50%"><img src="https://i-blog.csdnimg.cn/direct/53fb210330534141bfbb7f1fc94f7736.jpeg" width="50%">
