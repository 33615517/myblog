@[TOC]
>个人主页：[星轨初途](https://blog.csdn.net/2502_93089837?type=blog)
>个人专栏：[C语言](https://blog.csdn.net/2502_93089837/category_13078759.html)，[数据结构](https://blog.csdn.net/2502_93089837/category_13078761.html)，[C++学习（竞赛类）](https://blog.csdn.net/2502_93089837/category_13094661.html)
>

# 前言
嗨☆(≧∀≦*)ﾉ ！上一篇我们初识了C++,今天我们继续来学习C++的相关知识，这一篇主要讲解C++的输入输出，让我们一起来了解吧！
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/58ed2d7e9b83477fa8f29aec5743e921.jpeg#pic_center)

# 一、getchar和putchar
getchar() 和 putchar() 是属于 C 语⾔的库函数，但如果不进行练习的话，大家可能对这两个函数感到陌生，但这两个函数作用很大，所以这两个函数就进行简单的讲解啦！
## 1、getchar()（读取）
[函数相关链接](https://legacy.cplusplus.com/reference/cstdio/getchar/?kw=getchar)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f106a4a8d15a4bbcb24eb25faeec8192.png)
### （1）功能
getchar()函数返回用户从键盘输入的==一个字符==，使用时不带有任何参数。
程序运行到这个命令就会暂停，等待用户从键盘输入，等同于使用cin或scanf()方法**读取一个符**
### （2）头文件
  ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4209eb41a78d46378b37955bc02e026c.png)
### （3）返回值
**如果读取失败，返回常量 EOF ，由于 EOF 通常是 -1 ，所以返回值的类型要设为 int ，而不是char 。**
  也可以手动让它读取失败，直接按Ctrl+z就可以
  **读取成功返回字符的ASCll值**
### （4）读取方式
getchar() 不会忽略开头的空白字符，总是返回当前读取的第⼀个字符，无论是否为空格。
### （5）注意
```cpp
#include <iostream>
#include<cstdio>        
using namespace std;      
int main()                 
{
	int ch;
	ch=getchar();
	cout<<(char)ch<<endl;
    return 0;
}
```

这里输出时要以字符的形式输出，因为  ****读取成功返回字符的ASCll值**，所以我们输出时要转化为char,否则输出为该字符的ASCll值**
## 2、putchar()(输出）
[putchar函数链接](https://legacy.cplusplus.com/reference/cstdio/putchar/?kw=putchar)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/745f167b34474391982437c371d1760d.png)
### （1）功能及读取方式
参数

```cpp
int putchar( int character );
```
读取ASCll，输出对应的字符，如果character本身就是字符，就直接输出该字符
（输出一个字符）

```cpp
#include <iostream>
#include<cstdio>        
using namespace std;   
int main() 
{
    char c = 'b';
    putchar(55);   // 传入常量，输出'7'(ASCll码55） 
    putchar(c);     // 传入字符变量，输出'b'
    putchar('5');   // 传入字符'5'（ASCII码53），输出字符'5'
    return 0;
}
```
结果可以看出
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fca618fe03ae45de89160326efcd54ed.png)
### （2）头文件
和getchar一样

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e21a059a486c43fb8c1a0abe8b09a326.png)
### （3）返回值

操作成功时， putchar() 返回输出的字符，否则返回常量 EOF 。
## 3、结合
可以将这两个函数结合起来，一个读取一个输出
如：

```cpp
#include <iostream>
#include<cstdio>        
using namespace std;   
int main() 
{
    char c;
	c=getchar();
	putchar(c);
    return 0;
}
```
结果
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/cd45097c1245465cbda6a357b1e4d644.png)

或者循环读取，读取一个字符输出一个

```cpp
#include <iostream>
#include<cstdio>    
using namespace std;   
int main() 
{
    int c;
    while ((c = getchar()) != EOF) 
    {
        putchar(c);
    }
    return 0;
}
```
方便连续读取，想结束按Ctrl+z
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fe5ef0c275a4417d975278f8879a2ba6.png)
# 二、scanf 和printf及占位符
和C语言一样，这里就不进行讲解啦！
# 三、cin和cout
我们在上一篇已经进行cin和cout两个流对象的初识，本篇让我们进行更深入的了解吧！

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b79ecd047f534831ab939c72bfccad94.png)
>**我们在写程序的时候，经常需要处理数据，不管什么类型的数据都是以字符流的形式输⼊和输出的，也就是不管在键盘上输⼊什么类型的数据，还是将程序中的各种类型的数据输出显示到控制台屏幕上，都是以字符流的形式处理的。**
>
cin 和 cout 的输入输出十分方便，不需要手动控制格式，<font color="red">能够自动识别变量类型</font>
## 1、基础用法
cin用作输入，cout输出
如下：

```cpp
#include <iostream>
using namespace std;
int main()
{
 int a;
 char c;
 float f;
 
 cin >> a; // 读取1个整数
 cin >> c; // 读取1个字符
 cin >> f; // 读取取1个浮点数
 
 cout << "打印结果:"<<endl;
 cout << a << endl;
 cout << c << endl;
 cout << f << endl;
 return 0;
}
```
## 2、连续输入输出
那我们能不能一次读取多个，一次输出多个呀！当然可以！
**在上面的代码中，使用 cout 进行变量的输出，实质上是将变量插入到 cout 对象⾥，并以 cout对象作为返回值返回，因此我们还可以用 << 在后面连续输出多个内容，通过连续输入输出的方式对代码进行编写**
```cpp
#include <iostream>
using namespace std;
int main()
{
 int a;
 char c;
 float f;
 cin >> a>> c>> f; 
 cout << "打印结果:"<<endl;
 cout << a <<endl << c << endl << f << endl;
 return 0;
}
```
直接在后面加>>（cin）<<(cout)就行啦！

## 3、分析cin和cout的优势和劣势
### （1）优势
-  `cin`的好处
当输入若干个变量（数据量很少）时，可通过`>>`用一行代码完成接收，且无需关心数据的类型；和`scanf`函数相比，代码书写更简洁明了。

 - `cout`的好处
可以用来连续输出多个数值，且无需考虑数值的类型（自身会做类型处理）；和`printf`相比更方便。
### （2）劣势
**cin/cout的劣势（对比scanf/printf）：**
1. **效率低**：默认同步stdio，大数据量下易超时（需手动关闭同步）；
2. **格式控制麻烦**：小数位数、进制等设置比printf繁琐；
3. **输入灵活性差**：复杂分隔符/格式处理不如scanf方便；
4. **错误处理复杂**：类型不匹配时需手动清理状态。

# 四、cout的格式输出（加餐）
这一部分作为加餐，可以不做了解，因为printf比他控制格式方便，在竞赛中直接使用printf控制就行了，学有余力可以来了解一下

`printf`可通过指定格式输出数据（如宽度、小数位数、对齐方式等）。
 `cout`也能指定输出格式：结合`<iomanip>`（IO manipulators）头文件中的操纵符，即可灵活控制输出格式，满足格式化需求。
 头文件：`<iomanip>`
## 1、控制宽度和填充
`setw` ：设置字段宽度（只对紧接着的输出项有效）。==默认右对齐==
 `setfill `：设置填充字符。（紧跟着stew）
 [setw函数解释](https://legacy.cplusplus.com/reference/iomanip/setw/?kw=setw)
 [setfill函数解释](https://legacy.cplusplus.com/reference/iomanip/setfill/?kw=setfill)
 这里两个函数比较简单，就不详细讲解啦！
 相信大家可以通过代码就能理解

```cpp
#include <iostream>
using namespace std;
#include <iomanip>
int main()
 {
 int a = 123;
 
 cout << "默认宽度: " << a << endl;
 cout << "宽度设置为10: " << setw(10) << a << endl;
 cout << "宽度为10，不够时填充*: " << setw(10) << setfill('*') << a << endl;
 
 return 0;
}
```
结果
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c1f2fd64e8564d928fd0b5cac8fe6e3e.png)

## 2、控制数值格式
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8810de0268c341baab5bd66e89e30303.png)
还是通过代码来理解

```cpp
using namespace std; 
#include <iomanip>
int main() 
{
 double pi = 3.141592653589793;
 
 cout << "默认: " << pi << endl;
 cout << "固定小数点方式: " << fixed << pi << endl;
 cout << "科学计数法?式: " << scientific << pi << endl;
 cout << "固定小数点，?数点后2位有效数字: " << fixed << setprecision(2) << pi 
<< endl;
 
 return 0;
}
```
结果
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a5cc65026a834363baac5d173e99d029.png)
## 3、控制整数格式
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/abdda6a3ca0d48db843be3c40a723797.png)
实例代码

```cpp
#include <iostream>
using namespace std;
int main() {
    int num = 20;
    cout << "十进制（dec）：" << dec << num << endl; // 默认，输出20
    cout << "十六进制（hex）：" << hex << num << endl; // 输出14
    cout << "八进制（oct）：" << oct << num << endl;   // 输出24
    return 0;
}
```
结果
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b97ac0f8253047cba2a3cf55c8c55b3f.png)
我们发现真的是进行转换了，但是会有坑，下面进行讲解
### 注意
==这些操纵符是 “全局生效” 的 —— 一旦设置，后续所有整数输出都会沿用该进制，直到重新修改（如再次用dec切回十进制）。==

```cpp
#include <iostream>
using namespace std;
int main() 
{
    int num = 20;
    cout << "十进制（dec）：" << dec << num << endl; // 默认，输出20
    cout << "十六进制（hex）：" << hex << num << endl; // 输出14
    cout << "八进制（oct）：" << oct << num << endl;   // 输出24
    cout<<num<<endl; 
    int count=20;
    cout<<num<<endl;
    return 0;
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d3758ed1e25b408f934000b78113a309.png)
**我们发现真的是后面都按照8进制进行输出了！**

```cpp
#include <iostream>
using namespace std;
int main() 
{
    int num = 20;
    cout << "十进制（dec）：" << dec << num << endl; // 默认，输出20
    cout << "十六进制（hex）：" << hex << num << endl; // 输出14
    cout << "八进制（oct）：" << oct << num << endl;   // 输出24
    cout << "十进制（dec）：" << dec << num << endl; // 默认，输出20
    cout<<num<<endl; 
    int count=20;
    cout<<num<<endl;
    return 0;
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a3b8f119b6b64148a787bb1f141dff0d.png)
这样就可以啦！但我们使用时要千万注意
## 4、控制对齐方式
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/03d4dd74099e4cafaae0b764e5e64994.png)
我们在设置宽度是默认右对齐，我们想进行左对齐怎么办呢，可以用这两个方式进行。参考代码如下：

```cpp
#include <iostream>
using namespace std; 
#include <iomanip>
int main() 
{
 int n = 123;
 
 cout << "右对齐: " << setw(10) << right << n << endl;
 cout << "左对齐: " << setw(10) << left << n << endl;
 return 0;
}
```
效果如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/85322442c9cc40038cc7d8e93f2a97bd.png)
# 五、结束语
嗨(★＞U＜★)！本篇到这里就结束啦！本篇讲解了C++的一些输入输出方式及cout的加餐，相信大家都有所收获！博主将在带大家对C++更多的了解后，在进行C++的输入输出（下）的讲解（还涉及到其他知识），我们下一篇将要讲解《条件判断与循环和数组》的相关概念，感谢大家的支持啦‧★,:*:‧\(￣▽￣)/‧:*‧°★*　 ！


![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/265909951ee9429b850c1ff16f462e21.jpeg)
上期回顾
[C++入门（算法竞赛类）](https://blog.csdn.net/2502_93089837/article/details/155380481)







