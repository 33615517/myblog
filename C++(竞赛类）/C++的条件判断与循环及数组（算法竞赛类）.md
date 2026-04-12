@[TOC]
>个人主页：[星轨初途](https://blog.csdn.net/2502_93089837?type=blog)
>个人专栏：[C语言](https://blog.csdn.net/2502_93089837/category_13078759.html)，[数据结构](https://blog.csdn.net/2502_93089837/category_13078761.html)，[C++学习（竞赛类）](https://blog.csdn.net/2502_93089837/category_13094661.html)
>
# 前言
嗨(✪ω✪)！我们又见面啦！上一篇我们讲解了C++的输入和输出，今天我们要了解C++的条件判断与循环及数组（算法竞赛类），让我们一起来了解吧！
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d430c66244854b9dbf03eaad400b08f4.jpeg#pic_center)

# 内容注意
**本篇文章中范围for的原理涉及到后面几篇的内容，这里只需要先理解2的前3部分即可，会遍历一维二维静态数组即可**
# 一、条件判断与循环
在C++的条件判断与循环中
包括
>**if-else语句、关系操作符、条件操作符、逻辑操作符、switch语句、while循环语句、for循环语句、do-while循环语句、break和continue语句、循环嵌套**

==除了C++for循环比C多了个范围for，其他与C语言完全相同==,所以这里只讲解范围for的相关知识啦！

范围for和数组相联系（用C语言中的数组完全可以理解），所以我放在数组讲解的第二个板块进行讲解啦！
# 二、一维和二维数组数组

数组的创建和初始化，元素访问及元素打印与C一样，这里只进行auto关键字，范围for和memset设置数组内容,memcpy拷贝数组内容相关知识啦！
==其中范围for只需学会应用可以直接用的场景即可，实际做题就够用了，而且也不常用==
## 1、auto关键字
**功能**：auto 的主要用途是让编译器自动推导出变量的类型的
如下

```cpp
#include <iostream>
using namespace std;
int main()
{
    auto a = 3.14;
    auto b = 100;
    auto c = 'x';

    return 0;
}
```
我们在VS调试时可以看出auto自动识别类型
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/73272c52303b48f2ae1abfdc89c584f5.png)


## 2、范围for
我们在遍历数组元素时，通常用for循环及while等等，除了这些方法外，还有一个更方便的方式——使用范围for
### 注意
==我们在这个函数中只需学会（3）（4）直接可以直接用范围for，和了解（5）不能直接用范围for的场景即可==，（6）用的很少，仅做了解
### （1）范围for的核心功能

范围for是**C++11及以上版本引入的遍历语法**，核心作用是：
- 自动遍历「可迭代对象」的**所有元素**，无需手动写索引、计算长度；
- 简化遍历代码，减少“索引越界”等错误；
- 配合`auto`关键字可自动推导元素类型，进一步简化代码。

相当于将数组中取到的元素依次存到元素变量中进行变量
### （2）范围for的语法格式
基础语法：
```cpp
for (元素类型 元素变量 : 可迭代对象) {
    // 对元素变量的操作
}
```
常用简化写法（用`auto`自动推导元素类型）：
```cpp
// 只读遍历（元素变量是拷贝）
for (auto 元素变量 : 可迭代对象) { ... }

// 可修改元素（元素变量是引用）
for (auto& 元素变量 : 可迭代对象) { ... }
```

### （3）简单直观了解功能
我们先简单了解一下它的功能，再进行分析

```cpp
#include <iostream>
using namespace std;

int main()
{
    int arr[10] = { 1,2,3,4,5,6,7,8,9,10 };
    for (int e : arr)
    {
        cout << e << " ";
    }
    return 0;
}
```
上面代码中的`for`就是`范围for`，代码的意思是将`arr`数组中的元素，依次放在`e`变量中，然后打印`e`，直到遍历完整个数组的元素。这里的`e`是一个单独的变量，不是数组的元素，所以对`e`的修改，不会影响数组。
### （4）能直接用范围for的场景（及背后原理）
范围for能工作的**核心前提**是：**编译器能自动获取「可迭代对象」的遍历起点（`begin`）和终点（`end`）**。==（这里begin表示数组起始位置，end表示数组结束位置的下一个位置，下一篇会进行讲解这里可以先做了解）==

以下场景满足这个前提：

#### 1. 静态一维数组
（如`int arr[5]`）
- 原理：静态数组的类型是「`元素类型[长度]`」（比如`int[5]`），编译器能通过数组类型直接知道**元素个数**，从而生成`begin(arr)`（数组首地址）和`end(arr)`（数组首地址+长度）的迭代器。
- 示例：
  ```cpp
  int arr[5] = {1,2,3,4,5};
  for (auto num : arr) { // 编译器知道arr是int[5]，自动遍历5个元素
      cout << num << " ";
  }
  ```


#### 2. 静态二维数组
（如`int arr[3][4]`）
- 原理：静态二维数组的“行”是「一维数组类型」（比如`int[4]`），外层用`auto& row`（引用每行的一维数组），编译器能识别每行的长度，进而内层遍历元素。
- 示例：
  ```cpp
  int arr[3][4] = {{1,2}, {3,4}, {5,6}};
  for (auto& row : arr) { // row是int[4]的引用，编译器知道每行长度
      for (auto num : row) {
          cout << num << " ";
      }
  }
  ```


#### 3. STL容器（先作了解）
（如`vector`、`string`、`list`）
- 原理：STL容器内置了`begin()`和`end()`成员函数，编译器直接调用这两个函数获取迭代器，无需额外信息。
- 示例：
  ```cpp
  vector<string> vec = {"a", "b", "c"};
  for (auto& s : vec) { // 调用vec.begin()/vec.end()
      s += "!"; // 可修改元素
  }
  ```
==这里可以先不看，我们还没进行了解，后面会有几篇会进行讲解，可以先把这个跳过去，不影响，学完容器后再来看也可以==

### （5）不能直接用范围for的场景（及背后原理）
以下场景中，**编译器无法自动获取遍历的起点/终点**，因此范围for无法直接使用：

#### 1. malloc动态一维数组
（如`int* arr = (int*)malloc(5*sizeof(int))`）
- 原理：malloc返回的是「裸指针（如`int*`）」，指针仅记录内存起始地址，**不包含任何长度信息**，编译器不知道要遍历多少个元素，无法生成`begin(arr)`和`end(arr)`。
- 错误示例（编译报错）：
  ```cpp
  int* arr = (int*)malloc(5*sizeof(int));
  // for (auto num : arr) { ... } // 报错：无法确定arr的遍历边界
  ```


#### 2. 动态二维指针数组
（如`int** arr`）
- 原理：动态二维数组的“行”是「指针（如`int*`）」，指针同样没有长度信息，内层范围for无法识别要遍历多少个元素。
- 错误示例（编译报错）：
  ```cpp
  int** arr = (int**)malloc(2*sizeof(int*));
  arr[0] = (int*)malloc(3*sizeof(int));
  // for (auto& row : arr) { for (auto num : row) { ... } } // 报错：row是int*，无长度信息
  ```


### （6）补充：让“不能用的场景”支持范围for的方法（可忽略）
==注意==：这里仅做了解，在我们学完包装指针后再来看
如果想让动态数组支持范围for，可通过**包装指针+长度**的方式，告诉编译器遍历边界（C++20+推荐用`std::span`）：
```cpp
#include <span> // C++20及以上
int* arr = (int*)malloc(5*sizeof(int));
// 用span包装“指针+长度”，告诉编译器要遍历5个元素
for (auto num : std::span<int>(arr, 5)) {
    cout << num << " ";
}
```

### （7） 总结
范围for的核心是“**依赖编译器自动识别遍历边界**”——能识别（静态数组、STL容器）就可以直接用；不能识别（裸指针、动态指针数组）就无法直接用。


>**但是对于范围fo要慎重使用！范围for是对数组中所有元素进行遍历的，但是我们实际在做题的过程中，可能只需要遍历数组中指定个数的元素，这样范围fo就不合适了。**


## 3、编译器配置C++11

上面讲解的范围for是在 C++11 这个标准中引入的，如果你使用的编译器默认不支持C++11 ，可能需要配置才能使用。
这里以DevC++为例教程
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/770d975009c14886acfc4730c1752751.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/26541210f42f4b6aab4f0aa939d5a007.png)
勾选【编译时加入一下命令】，然后在下方的编译框中加入：

```cpp
-std=c++11
```
点击确定即可。
要想支持其他C++的标准也是一样的方法。

```cpp
-std=c++14
-std=c++17
-std=c++20
等
```
## 4、memset设置数组内容
函数链接[memset](https://legacy.cplusplus.com/reference/cstring/memset/?kw=memset)


==函数原型==
```cpp
void * memset ( void * ptr, int value, size_t num );
```

==参数解释==
- ptr -- 指针：指向了要设置的内存块的起始位置
- value -- 要设置的值
- num -- 设置的字节个数
### （1）功能
memset是用来设置内存的，将内存中的值以字节为单位设置成想要的内容，需要头文件
### （2）头文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/33f014b6f23d445f8813eb7df5dfb4d9.png)
### （3）设置数组功能实现

```cpp
#include <iostream>
#include <cstring>
using namespace std;
int main ()
{
    char str[] = "hello world";
    memset(str, 'x', 6);
    cout << str << endl;

    int arr[] = {1,2,3,4,5};
    memset(arr, 0, sizeof(arr));//这里数组的大小也可以自己计算
    for(auto i : arr)
    {
        cout << i << " ";
    }
    cout << endl;
    return 0;
}
```

这里就把数组arr内容设置为全为0了
#### 注意
这是关于`memset`使用的关键注意事项，核心是基于其“按字节填充内存”的特性，明确不同数组类型的使用限制，可拆解为以下两点并补充解读：

 1. 按字节填充的类型限制
`memset`以字节为单位设置内存内容，因此：
- **普通数组（如`int`/`double`数组）**：仅能设置为`0`。因为这类数组的元素是多字节类型（如`int`占4字节），若设置非`0`值，每个字节会被填充为目标值，最终元素值会是多个字节拼接的结果（如设置`1`会得到`0x01010101`，而非预期的`1`）；只有`0`的每个字节都是`0`，填充后元素值符合预期。
- **字符数组**：可任意设置值。因为字符类型（`char`）本身占1字节，按字节填充的结果与预期一致。


 2. 不同数组类型的使用差异
- **一维数组、二维静态数组**：可直接用`memset`设置。这类数组的内存是连续的整块存储，`memset`能一次性覆盖所有元素对应的内存区域。
- **二维动态指针数组**：需逐行调用`memset`。这类数组的内存是分散的（每个行指针指向独立的内存块），直接用`memset`无法覆盖所有行的内容，因此要对每个行指针指向的内存块单独执行`memset`操作。

### （4）用代码实现所以类型数组设置
这里用代码实现所以类型数组设置，便于大家理解

```cpp
#include <iostream>
#include <cstring>
#include <cstdlib>
using namespace std;

int main() {
    // -------------------------- 1. 一维数组设为 0 --------------------------
    int a1[5] = {1, 2, 3, 4, 5};  // a1=array 1（一维数组）
    memset(a1, 0, sizeof(a1));     // 内存连续，直接整体填充
    cout << "1. 一维数组设为 0 结果：";
    for (int num : a1) cout << num << " ";
    cout << "\n" << endl;

    // -------------------------- 2. 二维静态数组设为 0 --------------------------
    int a2[3][4] = {               // a2=array 2（二维静态数组）
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    memset(a2, 0, sizeof(a2));     // 静态数组内存连续，直接填充总字节数
    cout << "2. 二维静态数组设为 0 结果：" << endl;
    for (auto& row : a2) {
        for (int num : row) cout << num << " ";
        cout << endl;
    }
    cout << endl;

    // -------------------------- 3. 二维动态指针数组设为 0 --------------------------
    const int rows = 3, cols = 4;
    int* ad[rows];                 // ad=array dynamic（二维动态数组）
    
    // 为每行分配内存并逐行填充
    for (int i = 0; i < rows; i++) {
        ad[i] = (int*)malloc(cols * sizeof(int));
        memset(ad[i], 0, cols * sizeof(int));  // 内存分散，逐行填充
    }

    // 验证结果
    cout << "3. 二维动态指针数组设为 0 结果：" << endl;
    for (auto rowPtr : ad) {
        for (int j = 0; j < cols; j++) cout << rowPtr[j] << " ";
        cout << endl;
    }

    // 释放内存
    for (int i = 0; i < rows; i++) free(ad[i]);
    cout << endl;

    // -------------------------- 4. 字符数组设为 'w' --------------------------
    char ac[10] = "hello";         // ac=array char（字符数组）
    memset(ac, 'w', sizeof(ac));   // char占1字节，直接填充任意字符
    cout << "4. 字符数组设为 'w' 结果：" << ac << endl;

    return 0;
}
```

## 5、memcpy 拷贝数组内容
在使用数组的时候，有时候我们需要将数组的内容给数组b,比如：

```cpp
int a[10] = {1,2,3,4,5,6,7,8,9,10};
int b[10] = {0};
```
函数链接[memcpy](https://legacy.cplusplus.com/reference/cstring/memcpy/)
函数原型


```c
void * memcpy ( void * destination, const void * source, size_t num );
//destination -- 目标空间的起始地址
//source      -- 源数据空间的起始地址
//num         -- 拷贝的数据的字节个数
```
memcpy其实是用来做内存块的拷贝的，当然用来做数组内容的拷贝也是没问题的。
和memset注意点一样，这里就不做说明了
### （1）头文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ee404a0ab5884517b664af31fee48ebe.png)
### （2）功能实现
这里直接展示对所有类型数组进行操作的代码，便于理解

```cpp
#include <iostream>
#include <cstring>
#include <cstdlib>
using namespace std;

int main() {
    // -------------------------- 1. 一维数组拷贝 --------------------------
    int s1[5] = {10, 20, 30, 40, 50};  // 源：s1=source 1（一维）
    int d1[5];                          // 目标：d1=dest 1（一维）
    
    memcpy(d1, s1, sizeof(s1));
    
    cout << "1. 一维数组拷贝结果：";
    for (int num : d1) cout << num << " ";
    cout << "\n" << endl;

    // -------------------------- 2. 二维静态数组拷贝 --------------------------
    int s2[3][4] = {                    // 源：s2=source 2（二维静态）
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };
    int d2[3][4];                       // 目标：d2=dest 2（二维静态）
    
    memcpy(d2, s2, sizeof(s2));
    
    cout << "2. 二维静态数组拷贝结果：" << endl;
    for (auto& row : d2) {
        for (int num : row) cout << num << " ";
        cout << endl;
    }
    cout << endl;

    // -------------------------- 3. 二维动态指针数组拷贝 --------------------------
    const int rows = 3, cols = 4;
    // 源：ds=dynamic source（二维动态）
    int* ds[rows];
    for (int i = 0; i < rows; i++) {
        ds[i] = (int*)malloc(cols * sizeof(int));
        for (int j = 0; j < cols; j++) {
            ds[i][j] = i * cols + j + 1;
        }
    }

    // 目标：dd=dynamic dest（二维动态）
    int* dd[rows];
    for (int i = 0; i < rows; i++) {
        dd[i] = (int*)malloc(cols * sizeof(int));
        memcpy(dd[i], ds[i], cols * sizeof(int));
    }

    cout << "3. 二维动态指针数组拷贝结果：" << endl;
    for (auto rowPtr : dd) {
        for (int j = 0; j < cols; j++) cout << rowPtr[j] << " ";
        cout << endl;
    }

    // 释放内存
    for (int i = 0; i < rows; i++) {
        free(ds[i]);
        free(dd[i]);
    }
    cout << endl;

    // -------------------------- 4. 字符数组拷贝 --------------------------
    char cs[] = "Hello, memcpy!";       // 源：cs=char source（字符源）
    char cd[20];                        // 目标：cd=char dest（字符目标）
    
    memcpy(cd, cs, strlen(cs) + 1);
    
    cout << "4. 字符数组拷贝结果：" << cd << endl;

    return 0;
}
```

## 6、对范围for，memset，memcpy总结
范围for——遍历数组
memset——初始化普通数组或设置字符数组内为特定字符
memcpy——拷贝数组内容

 1. 范围for是**无需索引的自动遍历语法**，仅支持编译器可获取边界的可迭代对象（如静态数组、STL容器），**不支持裸指针和动态指针数组**（因无长度信息）；
 2. memset按字节填充连续内存（常用填0），memcpy按字节拷贝连续内存，二者均**需手动指定字节数、依赖内存连续**，**不支持动态指针数组等分散内存**，且memset对多字节类型（如int）**不能填非全字节值**（如1），memcpy需**避免内存重叠和浅拷贝**；

三者对静态数组均适用，核心差异在于范围for侧重**遍历便捷**，memset/memcpy侧重**底层内存操作**。

# 三、字符数组

字符数组的初始化，strlen，输入输出都和C语言一致，我们在这再进行字符数组输入中gets和fgets两个函数，以及简单回顾一下strcpy和strcat这两个函数
## 1、读取带有空格的字符串方法
我们在读取字符串时直接用scanf和cin到空格就停止了，无法继续读取，如果需要继续读取我们怎么办呢？
### （1）gets 和 fgets
函数链接[gets](https://legacy.cplusplus.com/reference/cstdio/gets/?kw=gets)
[fgets](https://legacy.cplusplus.com/reference/cstdio/fgets/?kw=fgets)
函数原型

```cpp
1 char * gets ( char * str );
2 char * fgets ( char * str, int num, FILE * stream );
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/54256448b4054eea80752edb8e1afd19.png)
==注意：==

>**使用gets函数的方式，这种方式能解决问题，但是因为gets存在安全性问题，在C+11中取消了gets,给出了更加安全的方案：fgets。**

实现

 方案1
```c
#include <cstdio>
//方案1
int main()
{
    char arr[10] = {0};
    gets(arr);
    printf("%s\n", arr);
    return 0;
}
```
替代方案-方法2
```c
//替代方案-方法1
//替代方案-方法2
#include <cstdio>
int main()
{
    char arr[10] = {0};
    fgets(arr, sizeof(arr), stdin);
    printf("%s\n", arr);
    return 0;
}
```
差异：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0bde06471de04bf68468f43300c1422e.png)
### （2）改变scanf读取方式
C语言中用`scanf`也能读取带空格的字符串，只需将格式符`"%s"`改为`"%[^\n]s"`：其中`[^\n]`表示一直读取直到遇到`\n`，遇到空格不会结束读取。
这种方式不会将`\n`读入，但会在读取的字符串末尾自动加上`\0`。
实现
```cpp
#include <cstdio>
int main()
{
    char arr[10] = "xxxxxxxx";
    scanf("%[^\n]s", arr);
    printf("%s\n", arr);
    return 0;
}
```
### （3）getchar逐个字符的读取
其实我们上一篇已经顺便介绍了getchar逐个字符读取，这里直接实现

```cpp
#include <cstdio>

int main()
{
    char arr[10] = { 0 };
    int ch = 0;
    int i = 0;
    while ((ch = getchar()) != '\n')
    {
        arr[i++] = ch;
    }
    printf("%s\n", arr);
    return 0;
}
```
## 2、strcpy
strcpy这个属于字符数组的复制
函数链接[strcpy](https://legacy.cplusplus.com/reference/cstring/strcpy/?kw=strcpy)
==函数原型==

```cpp
// 函数声明
char *strcpy ( char * destination, const char * source );

// 注释说明
// destination - 是目标空间的地址
// source      - 是源头空间的地址
// 需要的头文件 <cstring>
```
代码实现

```cpp
#include <cstdio>
#include <cstring>

int main()
{
    char arr1[] = "abcdef";
    char arr2[20] = {0};
    strcpy(arr2, arr1);
    printf("%s\n", arr2);
    return 0;
}
```
## 3、strcpy和memcpy对比
我们发现这两个函数功能相似，我们来分析一下他们的区别
 **核心区别：strcpy 是「字符串专用拷贝」，memcpy 是「通用内存拷贝」（本质差异）**
两者的所有差异都源于此定位，以下是关键对比（简洁版）：

| 对比维度         | strcpy                          | memcpy                          |
|------------------|---------------------------------|---------------------------------|
| 核心用途         | 仅拷贝字符串（char 数组/指针）  | 拷贝任意类型数据（int、struct、字符串等） |
| 拷贝依据         | 以字符串结束符 `\0` 为终止信号  | 以用户指定的「字节数」为终止依据 |
| 长度指定         | 无需手动指定（自动识别 `\0`）   | 必须手动指定拷贝的总字节数      |
| 适用数据类型     | 仅支持 char 类型（字符串）      | 支持所有数据类型（通用）        |
| 安全性           | 无长度检查，目标空间不足会溢出  | 无长度检查，但需手动确保字节数≤目标空间 |
| 头文件           | `<cstring>`（C/C++）            | `<cstring>`（C/C++）            |


**关键补充（实际使用重点）**
1. **strcpy 特点**：  
   - 自动拷贝到 `\0` 为止（包括 `\0` 本身，保证字符串完整）；  
   - 风险：若源字符串无 `\0` 或目标空间太小，会越界溢出（未定义行为）；  
   - 示例：`strcpy(dest, "abc")` → 自动拷贝 'a'/'b'/'c'/'\0' 共4字节。

2. **memcpy 特点**：  
   - 不关心数据内容，仅按字节「机械拷贝」，完全依赖字节数控制；  
   - 通用性强：可拷贝 int 数组、结构体等非字符串数据；  
   - 示例：`memcpy(dest, src, 4*sizeof(int))` → 拷贝4个 int （共16字节）。


 **==一句话总结==**
strcpy 是「字符串专属工具」，靠 `\0` 自动终止；memcpy 是「底层内存工具」，靠字节数精准控制，适用于所有数据类型。
## 4、strcat
有时候我们需要在⼀个字符的末尾再追加⼀个字符串，需要用到strcat
函数链接[strcat](https://legacy.cplusplus.com/reference/cstring/strcat/?kw=strcat)
==**函数原型**==

```cpp
// 函数声明
char * strcat ( char * destination, const char * source );

// 注释说明
// destination - 是目标空间的地址
// source      - 是源头空间的地址
// 需要的头文件 <cstring>
```
代码实现

```cpp
#include <cstdio>
#include <cstring>

int main()
{
    char arr1[20] = "hello ";
    char arr2[] = "world";
    strcat(arr1, arr2);
    printf("%s\n", arr1);
    return 0;
}
```
## 5、其余字符串函数
除了上面的两个字符串相关函数外，其实C/C++中还提供了⼀些其他的函数，有兴趣的可以拓展学习
链接：[其他函数](https://legacy.cplusplus.com/reference/cstring/)
# 本篇函数头文件
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ebc95f5f73834a9bbe2ab2f352d897d1.png)
# 结束语
本篇到这里就结束啦٩(๑❛ᴗ❛๑)۶！主要讲解了条件判断与循环及数组的相关内容，本篇中范围for讲的有点超纲，大家可以先学会范围for其中前三中的内容即可，后面学到相关知识后再来继续理解！感谢大家的支持啦(◕ᴗ◕✿)！

![请添加图片描述](https://i-blog.csdnimg.cn/direct/287fbde9f79040b08361bcd5d91264e3.jpeg)

上期回顾
[C++入门（算法竞赛类）](https://blog.csdn.net/2502_93089837/article/details/155380481)
[C++的输入输出（上）（算法竞赛类）](https://blog.csdn.net/2502_93089837/article/details/155392064?spm=1011.2415.3001.5331)
