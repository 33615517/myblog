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

![](https://i-blog.csdnimg.cn/direct/7e8075829ec547539a4df6e73ed9b568.jpeg#pic_center)
@[TOC]


# 前言


嗨(｡◕ˇ∀ˇ◕)！熬过了前三篇《类和对象》的硬核毒打，今天我们正式向 C++ 的深水区——**动态内存管理**进发！

C 语言那套不管对象“生老病死”（不调构造/析构）的 `malloc` 已经彻底不够用了。本篇带你手撕 `new/delete` 的底层逻辑，探秘 `operator new` 与大厂高频的内存防坑指南。废话不多说，直接起飞！
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b1488e6375ce458e8f890d060f759800.jpeg#pic_center)

---
# 一、C/C++内存分布
对于内存管理，我们在C语言中已经有所接触，相信大家都有一定的了解，下面我们通过一些代码和问题来进行引入
如果对下面的问题一点没印象的，可以温习一下[【C/C++底层修炼】拆解动态内存管理：四大动态内存函数、六大错误与柔性数组](https://blog.csdn.net/2502_93089837/article/details/159463465?spm=1011.2415.3001.5331)这一篇文章来进行复习
## 1.内存分配问题

```cpp
int globalvar = 1;
static int staticGlobalvar = 1;
void Test()
{
    static int staticvar = 1;
    int localVar = 1;

    int num1[10] = { 1, 2, 3, 4 };
    char char2[] = "abcd";
    const char* pChar3 = "abcd";
    int* ptr1 = (int*)malloc(sizeof(int) * 4);
    int* ptr2 = (int*)calloc(4, sizeof(int));
    int* ptr3 = (int*)realloc(ptr2, sizeof(int) * 4);
    free(ptr1);
    free(ptr3);
}
```

**【选项库】： A. 栈区  B. 堆区  C. 数据段(静态区)  D. 代码段(常量区)**

**🔍 深度硬核解析（务必区分变量本身和其指向的内容）：**

* **`globalVar` 在哪里？ —— 答案：C**
    * 解析：全局变量，存放在数据段（静态区）。
* **`staticGlobalVar` 在哪里？ —— 答案：C**
    * 解析：被 `static` 修饰的全局变量，依然存放在数据段。
* **`staticVar` 在哪里？ —— 答案：C**
    * 解析：局部静态变量。虽然定义在函数内部，但因为有 `static`，它早已脱离了栈区的束缚，被保存在数据段，生命周期被延长到全局。
* **`localVar` 在哪里？ —— 答案：A**
    * 解析：最普通的局部变量，函数调用时在栈帧中开辟空间。
* **`num1` 在哪里？ —— 答案：A**
    * 解析：局部数组，整个数组的内存空间都在栈上。

**(🚨 下方高能预警：字符串与指针的终极陷阱)**

* **`char2` 在哪里？ —— 答案：A**
    * 解析：`char2` 是一个局部数组的名字，整个数组开辟在栈上。
* **`*char2` 在哪里？ —— 答案：A**
    * 解析：`*char2` 解引用代表数组的第一个元素（字符 `'a'`）。因为**整个数组都在栈上**（它是把常量区的"abcd"拷贝了一份到栈上的数组里），所以元素自然也在栈上。
* **`pChar3` 在哪里？ —— 答案：A**
    * 解析：`pChar3` 是一个局部**指针变量**。只要是普通的局部变量，它**本身**就一定存在栈上！
* **`*pChar3` 在哪里？ —— 答案：D**
    * 解析：坑就在这！`pChar3` 本身在栈上，但它里面存的地址指向的是一个**只读的字符串常量** `"abcd"`。这个真正的字符串内容是存放在代码段（常量区）的！

**(🧨 动态内存陷阱)**

* **`ptr1` 在哪里？ —— 答案：A**
    * 解析：同理，`ptr1` 是个局部指针变量，本身必须在栈上。
* **`*ptr1` 在哪里？ —— 答案：B**
    * 解析：`*ptr1` 解引用访问的是 `malloc` 动态申请出来的那块真实内存空间，这块空间是开辟在**堆区**的！

> **💡 黄金开发经验总结：**
> 面对这类题，牢记一个铁律：**指针变量本身在哪里** 和 **它指向的那块内存到底在哪里** 是两码事！局部指针一定在栈上，至于它指哪，全看等号右边是字面常量（代码段）、还是普通数组（栈）、还是 `malloc/new`（堆）！

 - ==我们也可以通过图片来理解==（其实这张图片已经在C语言讲内存的时候进行展现了）

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e139b6c9fcf54d06b03430947eb73c4d.png)

## 2. C/C++ 内存区域硬核解析（四大战区）

做完上面那道坑爹的选择题，我们非常有必要重新审视一下 C/C++ 程序的内存布局。这不仅是面试必考的“八股文”，更是后期排查内存泄漏（Memory Leak）、段错误（Segfault）的理论基石。

在宏观视角下，操作系统的虚拟内存为我们的程序划分了以下几个核心战区：

* **1️⃣ 栈区 (Stack / 堆栈)**
  * **职责**：存放非静态局部变量、函数参数、返回值等。
  它就像是函数执行时的“草稿纸”，函数进栈就开始写，函数结束退栈就直接销毁。
  * **硬核特性**：**栈是向下增长的**（从高地址向低地址延伸）。
  它的分配效率极高，由编译器全权自动化管理，但缺点是**空间非常有限**（在 Linux/Windows 下通常只有几 MB）。如果你在栈上搞个超大数组或者无限递归，分分钟爆栈！
  * **栈的大小**：通常在链接时（Link-time）就固定了，一般非常小（比如 Windows 下默认 1MB，Linux 下默认 8MB）。

* **2️⃣ 内存映射段**
  * **职责**：这是一块高效的 I/O 映射区域，主要用来装载共享的动态内存库。
  * **进阶用途**：在高级系统编程中，用户可以使用系统接口（如 Linux 的 `mmap`）创建共享内存，用于极速的进程间通信（IPC）。

* **3️⃣ 堆区**
  * **职责**：用于程序运行时的**动态内存分配**。凡是你用 `malloc` / `calloc` / `realloc` 以及 C++ 的 `new` 申请出来的内存，全都在这里！
  * **硬核特性**：**堆是向上增长的**（从低地址向高地址延伸）。
  它的空间极其庞大，但分配效率相对较慢。
  最致命的是：**堆区的内存必须由程序员手动管理**！你申请了就必须手动释放（`free` / `delete`），只要你忘了，就会造成传说中的“内存泄漏”！
  但在现代 C++（C++11 及之后）的工业级开发中，我们**强烈抵制**手动调用 `delete`！最佳实践是拥抱 **RAII 思想**，使用**智能指针**（如 `std::unique_ptr`、`std::shared_ptr`）。将堆内存的生命周期绑定到栈对象上，利用栈自动销毁的特性实现堆内存的**自动化安全回收**，彻底告别内存泄漏噩梦！
  * **堆的大小与“4GB 陷阱”**：
    * **64位系统下**：受限于计算机的虚拟内存空间和操作系统限制，几乎可以认为是“无限”的（只要物理内存和虚拟内存够大）。
    * **🚨 32位系统的终极坑点**：很多初学者（甚至老手）会误以为 32 位系统有 $2^{32} = 4\text{GB}$ 的寻址空间，所以堆内存最多能达到 4GB。**这是极其经典的底层误区！**这 4GB 仅仅是**虚拟地址总空间**。操作系统是个霸道的家伙，它的内核（Kernel）会强行划走一大块（例如 Linux 默认霸占高地址的 1GB，Windows 默认霸占 2GB）。剩下留给用户的空间里，还要分给代码段、数据段、栈区、内存映射段等。因此，32 位系统下的堆内存**极限通常只有 1.5GB ~ 2GB 左右**，绝对不可能独吞 4GB ！




* **4️⃣ 数据段 (Data Segment / 静态区)**
  * **职责**：专门用来存储**全局数据**和**静态数据（`static` 修饰的变量）**。
  * **生命周期**：这块区域的数据命非常硬，它们的生命周期伴随着整个进程的始终，程序结束时才会统一由操作系统回收。


* **5️⃣ 代码段 (Code Segment / 常量区)**
  * **职责**：存放最终编译生成的可执行机器指令代码，以及**只读常量**（比如刚才题目里的字符串字面量 `"abcd"`）。
  * **防御机制**：这块区域通常是**严格只读**的！如果你试图用指针去强行修改常量区的数据（比如 `*pChar3 = 'x';`），操作系统会当场抛出段错误（Segmentation Fault）强杀进程。

![请添加图片描述](https://i-blog.csdnimg.cn/direct/c40bbd24bd7745bebba6250105045a1c.png)


> **💡 黄金开发经验：栈与堆的“双向奔赴”**
> 大厂面试官非常喜欢问：“栈和堆的增长方向为什么是相反的？”
> 答案在于内存布局的极致利用！栈在最高地址区向下生长，堆在较低地址区向上生长，两者相向而行，中间留有巨大的弹性空隙。这样既能让两者各自动态扩展，又能最大程度避免内存空间的相互踩踏。


## 3. 栈和堆的“静态/动态”分配之谜

我们通常会遇到极具迷惑性的问题：“**栈和堆都可以静态分配和动态分配吗**？”
很多新手会理所当然地认为：“栈是静态分配的，堆是动态分配的。” —— 如果你这么回答，那就大错特错了。

这里必须牢记一条硬核底层铁律：**堆只能动态分配，而栈既可以静态分配，也可以动态分配！**

**1. 堆（Heap）：只有动态分配（运行时按需申请）**
堆存在的唯一意义，就是为了在**运行时（Runtime）**处理那些大小未知、生命周期需要程序员手动控制的数据。编译器在编译阶段根本无法预知你需要多大的堆空间，因此堆**绝对不可能**进行静态分配。它只能通过 `malloc/new` 向操作系统动态索要内存。

**2. 栈（Stack）：静态分配 + 动态分配（高阶魔法）**
* **静态分配**：这是大家最熟悉的常规操作。比如写下 `int a = 10;` 或 `double arr[100];`，编译器在编译阶段就把大小算得明明白白，并在函数执行、栈帧建立时自动为你分配好空间。
* **动态分配（降维打击黑科技）**：栈其实**也可以动态分配**！在 C/C++ 中，我们可以使用系统底层的 `alloca()` 函数（Windows 环境下为 `_alloca()`），在程序运行时根据传入的变量大小，直接在当前栈帧上动态扩展空间！

**⚡ 栈动态分配的极致性能压榨（O(1) 魔法）**
为什么底层大牛喜欢在特定场景用栈的动态分配？因为它太快了！
相比于堆上繁琐的空闲链表搜索（容易产生内存碎片），栈的动态分配仅仅只是将栈顶指针（如 RSP/ESP 寄存器）做一下减法，时间复杂度是极其变态的 **O(1)**！而且，**函数退出时这块内存会随着栈帧销毁被自动回收，绝对不会发生内存泄漏，完全不需要手动释放！**

下面这段工业级代码演示了栈的动态分配魔法：

```cpp
#include <iostream>
#include <malloc.h> // Windows 下使用 _alloca，Linux 下包含 <alloca.h>

void StackDynamicAllocMagic(size_t dynamic_size) {
    // 【防御性编程】栈空间极其有限（通常仅几MB），分配前必须做严格的边界检查！
    if (dynamic_size == 0 || dynamic_size > 10000) {
        std::cerr << "Hisss... 分配大小不合法或存在爆栈风险！" << std::endl;
        return; 
    }

    // ✨ 见证魔法：在【栈】上根据运行时变量 dynamic_size 动态开辟数组
    // 🚨 注意：这里分配的空间，不需要也不能调用 free/delete！
    int* stack_dynamic_array = static_cast<int*>(_alloca(dynamic_size * sizeof(int)));

    // 初始化并使用这块极速内存
    for (size_t i = 0; i < dynamic_size; ++i) {
        stack_dynamic_array[i] = static_cast<int>(i * 10);
    }

    std::cout << "栈上动态分配成功，首元素：" << stack_dynamic_array[0] << std::endl;
    
    // 函数结束，退栈时 RSP 指针自动复位，内存瞬间归还系统，事了拂衣去！
}
```

> **💡终极总结：**
> * **堆的动态分配**：灵活性极高，空间大。但容易产生内存碎片，且必须强关联 `delete/free`，稍有不慎就会导致内存泄漏。
> * **栈的动态分配**：分配效率极速（O(1)），无外部碎片，自动回收。但空间极小，仅适用于生命周期仅在函数内部、且尺寸较小的动态内存需求。千万要做好防御性拦截，小心 **Stack Overflow（栈溢出）** 导致程序被操作系统当场击毙！



---
# 二、 C/C++内存管理方式


## 1.C语言中动态内存管理方式：malloc/calloc/realloc/free
我们在[【C/C++底层修炼】拆解动态内存管理：四大动态内存函数、六大错误与柔性数组](https://blog.csdn.net/2502_93089837/article/details/159463465?spm=1011.2415.3001.5331)这一篇已经进行讲解，大家可以看一下
这里我们之间进入问题
```c
void Test()
{
    // 1.malloc/calloc/realloc的区别是什么？
    int* p2 = (int*)calloc(4, sizeof(int));
    int* p3 = (int*)realloc(p2, sizeof(int)*10);

    // 这里需要free(p2)吗？
    free(p3);
}
```
1.  **`malloc/calloc/realloc` 区别**
    - `malloc(n)`：分配 `n` 字节内存，**不初始化**，内容随机。
    - `calloc(count, size)`：分配 `count * size` 字节内存，**全部初始化为0**。
    - `realloc(ptr, new_size)`：调整已有内存块 `ptr` 的大小到 `new_size`，可能原地扩容或分配新块并拷贝数据。

2.  **是否需要 `free(p2)`？**
    - 不需要。`realloc` 成功后，原指针 `p2` 已经失效（要么内存块原地扩展，要么系统自动释放旧块并返回新地址 `p3`）。
    - 只需要 `free(p3)` 即可，重复 `free(p2)` 会导致**未定义行为**（程序崩溃或内存 corruption）。

3. **malloc的实现原理**？
malloc 不是直接每次找 OS 要内存，而是先预申请一大块堆内存，自己管理分配与释放，高效复用

大家可以通过这个视频来进行详细了解
>[malloc的实现原理](https://www.bilibili.com/video/BV117411w7o2/?spm_id_from=333.788.videocard.0&vd_source=e55f16c87dbfaa5f553f0a07bb61817e)

## 2.C++内存管理方式
C语言内存管理方式在C++中可以继续使用，但有些地方就无能为力，而且使用起来比较麻烦，因此C++又提出了自己的内存管理方式：==通过new和delete操作符进行动态内存管理。==
### 2.1 new/delete的操作内置类型
```cpp
void Test()
{
    // 动态申请一个int类型的空间
    int* ptr4 = new int;

    // 动态申请一个int类型的空间并初始化为10
    int* ptr5 = new int(10);

    // 动态申请10个int类型的空间（代码里是3个）
    int* ptr6 = new int[3];

    // 动态申请9个int类型的空间,并初始化前4个
    int* ptr7 = new int[9]{1,2,3,4};
    

    delete ptr4;
    delete ptr5;
    delete[] ptr6;
    delete[] ptr7;
}
```

- `new int`：申请单个int空间，**不初始化**
- `new int(10)`：申请单个int空间，**初始化为10**
- `new int[3]`：申请3个int的数组空间，**不初始化**
-  `new int[9]{1,,2,3,4}`：申请9个int的数组空间，**部分初始化**
- 对应释放规则：单个对象用`delete`，数组用`delete[]`，必须匹配使用，否则会导致未定义行为

图片便于理解
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b84b59210ce04b06b19a79224e2a4bfe.png)
>**注意：申请和释放单个元素的空间，使用new和delete操作符，申请和释放连续的空间，使用new[]和delete[]，注意：匹配起来使用。**

### 2.2  new 和delete操作自定义类型
#### 2.2.1 创建单个对象+对比new/delete相对于malloc的优势
```cpp
#include <iostream>
using namespace std;

class A
{
public:
    A(int a = 0)
        : _a(a)
    {
        cout << "A():" << this << endl;
    }

    ~A()
    {
        cout << "~A():" << this << endl;
    }

private:
    int _a;
};

int main()
{
    // new/delete 和 malloc/free 最大区别是 new/delete 对于【自定义类型】除了开空间
    // 还会调用构造函数和析构函数

    A* p1 = (A*)malloc(sizeof(A));
    A* p2 = new A(1);
    free(p1);
    delete p2;

    // 内置类型是几乎是一样的
    int* p3 = (int*)malloc(sizeof(int)); // C
    int* p4 = new int;
    free(p3);
    delete p4;

    A* p5 = (A*)malloc(sizeof(A) * 10);
    A* p6 = new A[10];
    free(p5);
    delete[] p6;

    return 0;
}
```

注意：在申请自定义类型的空间时，new会调用构造函数，delete会调用析构函数，而malloc与free不会。——==这也就是new/delete 和 malloc/free最大区别==

- 对于我们**数据结构学过的链表**而言，new/delete对比malloc/free的**优势会更加突出**

==我们可以通过下面代码来看出来ew/delete对比malloc/free的优势==

```cpp
#include<iostream>
using namespace std;

struct ListNode
{
	int val;
	ListNode* next;

	// 💡 C++ 专属灵魂：构造函数
	ListNode(int x)
		:val(x)
		, next(nullptr)
	{
	}
};

int main()
{
	// ---------------------------------------------------------
	// 🤮 史前时代：如果是 C 语言的 malloc，写个链表节点简直是受刑：
	// ListNode* n1 = (ListNode*)malloc(sizeof(ListNode));
	// if (n1 == NULL) return -1; // 还得检查空指针
	// n1->val = 1;               // 手动赋值
	// n1->next = nullptr;        // 手动置空
	// 每次申请都要强转、算大小、手动塞数据，极其繁琐！
	// ---------------------------------------------------------

	// ✨ 现代 C++：见证 new 的降维打击！
	// new 的底层逻辑：1. 在堆上开辟空间； 2. 【自动调用该对象的构造函数】！
	// 内存申请 + 完美初始化，一行代码直接优雅起飞：
	ListNode* n1 = new ListNode(1);
	ListNode* n2 = new ListNode(2); // (为了链表看起来真实点，我把值改成了 1,2,3,4)
	ListNode* n3 = new ListNode(3);
	ListNode* n4 = new ListNode(4);

	// 节点链接，干净利落
	n1->next = n2;
	n2->next = n3;
	n3->next = n4;

	// 🚨 黄金开发习惯：有 new 必有 delete！
	// 永远别做内存泄漏的罪人。delete 会先调用对象的析构函数，再释放堆空间。
	delete n1;
	delete n2;
	delete n3;
	delete n4;

	return 0;
}
```
#### 2.2.2 创建对象数组

 **==动态对象数组的初始化：3 种姿势对比==**

在堆区动态申请对象数组（如 `new A[3]`）时，如何给这几个对象优雅地赋初值？C++ 演进了 3 种写法，按现代工程标准，推荐程度依次递增：

* **🥉 写法 1：有名对象拷贝（最啰嗦）**。先在栈上创建局部对象，再拷贝进堆数组。代码行数多，且存在多余的拷贝开销。（❌ 极不推荐）
* **🥈 写法 2：匿名对象（进阶）**。直接在 `new` 语句中通过 `A(x, y)` 构造匿名对象。省去了栈变量，编译器通常会优化掉拷贝，直接原地构造。（✅ 推荐）
* **🥇 写法 3：隐式类型转换（极致优雅）**。C++11 引入的王炸特性，直接用 `{}` 传参匹配对应的构造函数。连类名都省了，代码极简，性能拉满！（🔥 强烈推荐）

代码演示

```cpp
#include<iostream>
using namespace std;
class A
{
public:
	A(int a1 = 0, int a2 = 0)
		:_a1(a1)
		, _a2(a2)
	{
		cout << "A(int a1 = 0, int a2 = 0)" << endl;
	}

	A(const A& aa)
		:_a1(aa._a1)
	{
		cout << "A(const A& aa)" << endl;
	}

	A& operator=(const A& aa)
	{
		cout << "A& operator=(const A& aa)" << endl;
		if (this != &aa)
		{
			_a1 = aa._a1;
		}
		return *this;
	}

	~A()
	{
		//delete _ptr;
		cout << "~A()" << endl;
	}

	void Print()
	{
		cout << "A::Print->" << _a1 << endl;
	}

	A& operator++()
	{
		_a1 += 100;

		return *this;
	}
private:
	int _a1 = 1;
	int _a2 = 1;
};

int main()
{
	// 🥉 写法 1：有名对象拷贝（性能拉跨）
	A aa1(1, 1);
	A aa2(2, 2);
	A aa3(3, 3);
	A* p3 = new A[3]{aa1, aa2, aa3};

	// 🥈 写法 2：匿名对象（编译器优化原地构造）
	A* p4 = new A[3]{ A(1,1), A(2,2), A(3,3)};

	// 🥇 写法 3：隐式类型转换 + 初始化列表（现代 C++ 极致优雅）
	A* p5 = new A[3]{ {1,1}, {2,2}, {3,3} };

	// ---------------------------------------------------------
	// 🚨 擦屁股环节：只要申请的是数组（有 []），释放必须带 delete[]
	// ---------------------------------------------------------
	delete[] p3; 
	delete[] p4;
	delete[] p5;

	return 0;
}
```
### 2.3 new申请空间失败的处理
我们之前学C语言时**malloc**申请失败返回**NULL**，，而**new**申请失败会返回**抛异常**

当我们申请过大失败时，程序会直接崩溃异常
```cpp
#include<iostream>
using namespace std;
int main()
{
	void* p1 = new char[1024 * 1024 * 1024];
	void* p2 = new char[1024 * 1024 * 1024];
	return 0;
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b2dcb345ec8343a49dfc5b06983df963.png)


我们既然提到了“抛异常”，很多习惯了 C 语言 `if (ptr == NULL)` 的小伙伴可能会一脸懵：抛异常到底是个啥现象？如果抛了异常我们不管，会发生什么？

答案极其暴躁：**如果 `new` 失败抛出了异常而你没有去拦截它，操作系统为了自保，会直接把你的程序强杀崩溃！**

但在真实的商业项目（比如微信后台服务器）里，系统绝对不能因为某个瞬间内存不够就直接宕机！那么，我们该如何优雅地拦截这个致命错误，让程序安全存活呢？

这就必须请出 C++ 的专属保命神技：**`try-catch` 异常捕获机制**。

* **`try { }`（高危监控区）**：把有可能“炸雷”（如 `new` 极大内存）的代码放在这里面。意思是：“你且大胆去试，我给你盯着。”
* **`catch () { }`（兜底安全网）**：如果 `try` 里面的代码真的抛出了异常，程序**绝对不会崩溃**！它会瞬间中断当前逻辑，精准落入 `catch` 的怀抱里，让你有机会输出报错信息或执行清理工作。

口说无凭，我们直接来写一段**暴力榨干堆内存**的极限测试代码，带你直观感受一下 `try-catch` 是如何完美兜底的：

```cpp
#include<iostream>
using namespace std;

// 模拟一个极端吃内存的业务逻辑
void func()
{
	int n = 1;
	while (1) // 💀 死循环，直到物理内存被彻底榨干
	{
		// 每次疯狂申请 1MB 的空间 (1024 * 1024 Byte = 1MB)
		void* p1 = new char[1024 * 1024]; 
		
		// 打印申请成功的地址和次数
		cout << p1 << "->"<< n<<endl;
		++n;
	}
}

int main()
{
	// 🛡️ 开启异常监控
	try
	{
		func(); // 执行极度危险的内存分配函数
	}
	// 🕸️ 兜底安全网：精准捕获 new 抛出的标准异常
	catch (const exception& e)
	{
		// e.what() 会打印出系统底层的具体死因（通常输出：bad allocation）
		cout << e.what() << endl; 
	}

	
	return 0;
}
```
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d54d5d60130240ba90e2d5bcea456b4a.png)

> **💡 黄金开发经验：**
> 这里的 `catch (const exception& e)` 是 C++ 异常捕获的标准姿势。`exception` 是所有标准库异常的“老祖宗”。而 `e.what()` 就像是法医的验尸报告，能直接告诉你底层到底是因为什么死的（比如跑完这段代码，它一定会打印出 `bad allocation`，也就是内存分配失败）。
> **牢记：** 以后在 C++ 里处理 `new` 可能带来的失败，彻底忘掉 `NULL`，拥抱 `try-catch` 才是真正的大厂思维！

---
# 三、operator new与operator delete函数(重点)


**new和delete**是用户进行动态内存申请和释放的**操作符**，**operator new**和**operator delete**是系统提供的**全局函数**，**new**在底层调用**operator new**全局函数来申请空间，**delete**在底层通过**operator delete**全局函数来释放空间。
<br>


<center><strong>我们来看一下底层代码</strong></center>

## 1.operator new 函数

```cpp
/*
operator new: 该函数实际通过malloc来申请空间，当malloc申请空间成功时直接返回；申请空间
失败，尝试执行空间不足应对措施，如果改应对措施用户设置了，则继续申请，否则抛异常。
*/
void *__CRTDECL operator new(size_t size) _THROW1(_STD bad_alloc)
{
    // try to allocate size bytes
    void *p;
    while ((p = malloc(size)) == 0)
        if (_callnewh(size) == 0)
        {
            // report no memory
            // 如果申请内存失败了，这里会抛出bad_alloc 类型异常
            static const std::bad_alloc nomem;
            _RAISE(nomem);
        }

    return (p);
}
```


## 2.operator delete 函数

```cpp
/*
operator delete: 该函数最终是通过free来释放空间的
*/
void operator delete(void *pUserData)
{
    _CrtMemBlockHeader * pHead;

    RTCCALLBACK(_RTC_Free_hook, (pUserData, 0));

    if (pUserData == NULL)
        return;

    _mlock(_HEAP_LOCK);  /* block other threads */
    __TRY
        /* get a pointer to memory block header */
        pHead = pHdr(pUserData);

        /* verify block type */
        _ASSERTE(_BLOCK_TYPE_IS_VALID(pHead->nBlockUse));

        _free_dbg( pUserData, pHead->nBlockUse );

    __FINALLY
        _munlock(_HEAP_LOCK);  /* release other threads */
    __END_TRY_FINALLY

    return;
}

/*
free的实现
*/
#define   free(p)               _free_dbg(p, _NORMAL_BLOCK)
```
通过上述两个全局函数的实现知道，**operator new实际也是通过malloc**来申请空间，如果malloc申请空间成功就直接返回，否则执行用户提供的空间不足应对措施，如果用户提供该措施就继续申请，否则就抛异常。**operator delete最终是通过free**来释放空间的。

## 3.使用类型不匹配的后果

> **🚨 C++ 内存管理黄金铁律：必须严格配套使用！**
> * **`malloc`** 必须配 **`free`**
> * **`new`** 必须配 **`delete`**
> * **`new[]`** 必须配 **`delete[]`**
> 
> **切记：绝对不能交叉混用，否则极易引发隐蔽的内存泄漏或程序直接崩溃！**

我们常说 `new/delete` 和 `malloc/free` 绝对不能混用，但很多头铁的新手会抬杠：“我混用了也没见程序崩溃啊？”

今天，我们就来写一段极其作死的代码，分 4 个层次，带你亲眼看看“乱点鸳鸯谱”到底会引发什么级别的灾难：

#### 🥉 级别一：`new` 一个内置类型，用 `free` 释放
```cpp
void Test1()
 {
    int* p1 = new int(10);
    free(p1); // 表面上风平浪静，没崩溃也没泄漏
}
```
* **后果解析**：对于 `int` 这种内置类型，它没有构造函数也没有析构函数。`new` 在底层本来就是去调 `malloc` 申请空间的。所以你用 `free` 把空间还回去，编译器也就睁一只眼闭一只眼了。
* **致命误区**：别以为没报错就是对的！这属于典型的**未定义行为（UB）**，强依赖于编译器的底层实现，换个环境分分钟教你做人。

#### 🥈 级别二：`new` 一个自定义类型，用 `free` 释放（隐蔽的内伤）
```cpp
class A 
{
public:
    A(int a = 0) { _ptr = new int[100]; cout << "A()" << endl; }
    ~A() { delete[] _ptr; cout << "~A()" << endl; }
private:
    int* _ptr;
};

void Test2()
 {
    A* p2 = new A(1);
    free(p2); // 🚨 灾难：没有调用析构函数！
}
```
* **后果解析**：`free` 是个没有感情的 C 语言机器，它不管对象死活，直接把 `p2` 指向的这块 `A` 对象的躯壳给回收了。
* **致命一击**：因为它没有调用 `~A()` 析构函数，导致 `A` 对象内部那个指向 100 个 `int` 的 `_ptr` 永远留在了堆区！**程序不崩溃，但你的内存正在疯狂泄漏！**

#### 🥇 级别三与终极核爆：数组的错配与 44 字节之谜！

前方高能预警！这是面试最爱考的底层深度题。仔细看下面这段代码：

```cpp
#include<iostream>
using namespace std;
class A 
{
public:
	A(int a = 0)
		:_a(a)
	{
		cout << "A();" << this << '\n';
	}
	~A()
	{
		cout << "~A();" << this << '\n';
	}
private:
	int _a;
};
 
int main()
{
	// A* p2 = (A*)operator new(sizeof(A)); // 可以显式调用底层函数

	A* p1 = new A(1);
	delete p1; // 单个对象，正常释放

	// ---------------------------------------------------
	// 💀 终极作死：动态申请 10 个对象的数组，却用普通的 delete 释放
	// ---------------------------------------------------
	A* p2 = new A[10]; 
	
	// 🚨 此时如果直接 delete p2; 程序会当场崩溃！
	delete p2;      

	return 0;
}
```

**🔍 灵魂拷问：为什么 `delete p2` 位置错了，程序会直接崩溃？**

我们通过查看底层的反汇编代码和内存监视器，来揭开这个惊天大秘密：


假设我们的类 `A` 大小是 4 个字节（里面只有一个 `int` 成员）。我们 `new A[10]`，理所应当觉得操作系统应该给我们分配 **`40 个字节`** 的空间，对吧？
但是，如果你去底层看底层的 `malloc` 调用，你会震惊地发现，**它实际申请了 44 个字节！**
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5f1e19fa9df24c4991d61c6f99c418b6.png)
按f11
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bf02a2c568e74940adfe0aef06202f73.png)
**多出来的这 4 个字节是干嘛的？**
因为你申请的是对象数组，且类 `A` **有析构函数**。编译器必须得记住你到底申请了几个对象，不然它等会儿用 `delete[]` 释放的时候，怎么知道要调用几次析构函数呢？
于是，编译器偷偷在给你分配的内存**最头部，塞了一个数字 `10`**（占 4 个字节，专业术语叫 Cookie）。然后，把往后偏移了 4 个字节的地址，交给了你的指针 `p2`！

> **💡 黄金反向印证：**
> 如果你把类 `A` 的析构函数 `~A()` 给**注释掉**，你再去底层看，你会发现申请的空间又神奇地**变回了 40 个字节**！
> 因为编译器很聪明：既然你没有析构函数需要调用，那我记那个数字（10）还有什么用？直接优化掉！

**💥 崩溃真相大白：**
当你写下 `delete p2;` （漏了 `[]`）时：
1. 编译器以为这只是一个单体对象，所以**只调用了 1 次析构函数**。
2. 最致命的是，底层的 `free` 机制会直接从 `p2` 当前指向的地址开始回收内存！但我们刚才说了，真正的内存起始块其实在 `p2` **往前退 4 个字节**的地方！
3. 把一个偏移过的错误地址交给操作系统去释放，操作系统会直接判定为**堆内存被破坏（Heap Corruption）**，当场强杀你的程序！

> **🚨 极限避坑总结：**
> 不要管内置类型会不会崩溃，也不要管有没有析构函数！
> 只要在 C++ 里写代码，把这条铁律焊死在键盘上：**`new` 必须配 `delete`，`new[]` 必须配 `delete[]`！错一个符号，就是万丈深渊！**

---
# 四、 new和delete的实现原理

## 1. 内置类型
如果申请的是内置类型的空间，`new` 和 `malloc`，`delete` 和 `free` 基本类似。
**不同的地方是：**
* `new/delete` 申请和释放的是单个元素的空间，`new[]` 和 `delete[]` 申请的是连续空间。
* 而且 `new` 在申请空间失败时会**抛异常**，`malloc` 会返回 **`NULL`**。

## 2.自定义类型

* **`new` 的原理**
  1. 调用 `operator new` 函数申请空间。
  2. 在申请的空间上执行构造函数，完成对象的构造。

* **`delete` 的原理**
  1. 在空间上执行析构函数，完成对象中资源的清理工作。
  2. 调用 `operator delete` 函数释放对象的空间。

* **`new T[N]` 的原理**
  1. 调用 `operator new[]` 函数，在 `operator new[]` 中实际调用 `operator new` 函数完成 N 个对象空间的申请。
  2. 在申请的空间上执行 N 次构造函数。

* **`delete[]` 的原理**
  1. 在释放的对象空间上执行 N 次析构函数，完成 N 个对象中资源的清理。
  2. 调用 `operator delete[]` 释放空间，实际在 `operator delete[]` 中调用 `operator delete` 来释放空间。

>💡注意
>new和malloc都是保留字,需要头文件malloc.h,只是平时这个头文件已经被其他头文件所包含了，用的时候很少单独引入
>
# 五、 定位new表达式(placement-new) （了解）

在 C++ 的深水区，还有一个极其硬核、甚至看起来有些“违背常理”的语法——**定位 new（placement-new）**。这也是大厂在考察“高并发内存池开发”时必问的顶级八股文。

我们前面学过，普通的 `new` 是个“一条龙服务”：先向操作系统申请内存，再调用构造函数。
但**定位 new** 的作用是：**“只管杀不管埋”——哦不对，是“只管造不管批”**！它允许你在**已经分配好的一块纯净内存（生肉）上，强行调用构造函数去初始化一个对象（把生肉做成牛排）。**

==📌 语法格式：==
```cpp
new (place_address) type; 
// 或者带参数：
new (place_address) type(initializer-list);
```
> **注：** `place_address` 必须是一个有效的内存指针。

==💡 为什么要费这么大劲“脱裤子放屁”？==
很多新手会问：直接用普通的 `new` 不香吗？为什么非要先自己搞一块内存，再用 `placement-new` 去构造？

**答案只有两个字：性能！**
在开发高并发服务器时，如果频繁地使用普通的 `new/delete` 向操作系统申请和释放内存，会产生巨大的开销和内存碎片。
所以，大牛们会自己写一个**内存池（Memory Pool）**：一次性向系统批发一大块毫无生机的“纯净内存”。当业务需要创建对象时，直接从池子里切一块内存出来，然后用 **`placement-new`** 在这块内存上显式调用构造函数，赋予它“灵魂”。这就极大提升了程序的运行效率！

==⚔️实战代码：==

来看这段极具画面感的代码，注意看里面**显式调用析构函数**的骚操作：

```cpp
#include<iostream>
using namespace std;

class A {
public:
	A(int a = 0) : _a(a) { cout << "A(): 赋予灵魂 -> " << this << endl; }
	~A() { cout << "~A(): 剥夺灵魂 -> " << this << endl; }
private:
	int _a;
};

int main()
{
	// ------------------------------------------------------------------
	// 玩法 1：配合 malloc 食用
	// ------------------------------------------------------------------
	// 1. 批发“生肉”：申请一块和 A 对象一样大的纯净内存，此时它还不是对象！
	A* p1 = (A*)malloc(sizeof(A));
	
	// 2. 注入灵魂（定位 new）：在这块指定的内存 p1 上，强行调用构造函数
	new(p1)A;  // 如果有参数就是 new(p1)A(10);
	
	// 🚨 3. 致命踩坑点：必须手动显式调用析构函数清理资源！
	// 这是 C++ 极其罕见的【允许显式调用析构函数】的场景
	p1->~A();  
	
	// 4. 退还生肉
	free(p1);

	// ------------------------------------------------------------------
	// 玩法 2：配合 operator new 食用 (更纯正的 C++ 风味)
	// ------------------------------------------------------------------
	A* p2 = (A*)operator new(sizeof(A));
	
	new(p2)A(10); // 定位 new 传参初始化
	
	p2->~A();             // 手动析构
	operator delete(p2);  // 释放底层的纯净空间

	return 0;
}
```

> **🚨 极限防雷总结：**
> 使用 `placement-new` 就像是你在手动驾驶一架没有自动驾驶仪的战斗机。
> 因为内存是你自己手动批发的（`malloc` 或 `operator new`），所以对象销毁时，**编译器绝对不会自动帮你调用析构函数！** 你必须像代码中那样，亲手写下 `p1->~A();` 来清理对象内部的资源，然后再去释放外层的内存躯壳。少做一步，就是万劫不复的内存泄漏！


# 六、`malloc/free` 和 `new/delete` 的区别

`malloc/free` 和 `new/delete` 的**共同点**是：都是从堆上申请空间，并且都需要用户手动释放。

**不同的地方是（划重点）：**

1. **语法属性**：`malloc` 和 `free` 是**函数**，`new` 和 `delete` 是**操作符**。
2. **初始化能力**：`malloc` 申请的空间不会初始化，`new` 可以进行初始化。
3. **大小计算**：`malloc` 申请空间时，需要手动计算空间大小并传递；`new` 只需在其后跟上空间的类型即可，如果是多个对象，在 `[]` 中指定对象个数即可。
4. **返回值类型**：`malloc` 的返回值为 `void*`，在使用时必须强转；`new` 不需要，因为 `new` 后面跟的就是空间的类型。
5. **失败处理机制**：`malloc` 申请空间失败时，返回的是 `NULL`，因此使用时必须判空；`new` 不需要判空，但是 `new` 失败会抛出异常，需要捕获异常。
6. **底层核心逻辑（最重要的一点）**：申请自定义类型对象时，`malloc/free` 只会单纯地开辟和回收空间，**不会**调用构造函数与析构函数；而 `new` 在申请空间后会**调用构造函数**完成对象的初始化，`delete` 在释放空间前会**调用析构函数**完成空间中资源的清理释放。


# 结束语
嗨！本篇到这里就结束啦！动态内存管理是 C++ 最自由、也是最危险的利刃。从底层四大战区，到 `operator new` 源码剥丝抽茧，再到极易踩坑的“交叉释放”与 placement-new 黑科技，希望这篇文章能帮你彻底扫清大厂面试和日常开发中的“段错误”阴霾。
**如果这篇干货对你有帮助，别忘了点赞、收藏、关注一键三连支持一下！**，感谢大家的支持啦！

 ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5b5b3d0d33964232919cdb8cd3aac38b.jpeg)![请添加图片描述](https://i-blog.csdnimg.cn/direct/490b1a39ccb54b5b89600e5605cd9574.jpeg)
 



