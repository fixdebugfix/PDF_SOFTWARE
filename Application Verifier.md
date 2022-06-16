@[TOC]

> 应用程序验证器（Application Verifier）这个工具是做什么的？ 
>
> Application Verifier是来自微软官方的一款应用程序验证工具，主要用于帮助用户检测和调试内存损坏、危险的安全漏洞、Run-time检测等；是一款辅助开发工具，不用修改源码；在程序退出时报告未释放的资源等，程序正常退出才会有报告
>
- 获取：可以通过windows software development Kit 管理器来下载最新版
- 推荐：WinDbg和应用程序验证器在很多场景下配合使用、参考中的文章也可以都读一下
- 原理：hook掉分配和释放资源的API，做一些检查和记录
- 前置知识：最好会使用WinDbg、gflag.exe、懂得堆的基本知识...，不懂也没有关系，百度一下就知道了
> 使用WinDbg这么长时间居然不知道`AV`，还是在看《软件调试》第2版，第19章时发现的，下面是对当时的笔记进行的整理

# 0x01 Introdution

> 如果第一次接触这个工具，建议读一下`appverif.chm`的`How to use Application Verifier`部分，可以当成应用程序验证器的介绍文档，包括应用时机、安装和配置、选择64还是32位版本、基本测试、结果分析等

当一个Release版本软件出现bug时，可以使用操作系统提供的验证机制；校验机制中有2个常用工具：

- 驱动程序验证器（`verifier.exe`）：用户态工具
- 应用程序验证器（`appverif.exe`）：内核态工具

应用程序验证器（`appverif.exe`）是windows本地代码的`运行时`分析工具，依赖于操作系统的支持来捕获常见的系统API误使用，工具可以自动捕获很多难以重现的代码错误；**是一个设置应用验证机制参数的工具**

工具主要包括2个部分：实现在`verifier.dll`等dll中的`avf`开头的函数和一个设置工具（`appverif.exe`）

应用程序验证器（`appverif.exe`）本质上就是一个工具，下面通过问答和一个示例来讲解

# 0x02 常见问题

> 提示：`appverif.chm`的`Frequently Asked Questions about the  Application Verifier`章节里面有很多新手遇到的问题解答

下面通过问答的形式说明工具的一些使用疑惑（主要是第一次使用会有），其实多操作几次自然就记住了

## Q1：使用时机？

什么情况下可以使用应用程序验证机制呢？通常有2种情况：

- 1.在开发中为所有相关进程启动，缺点是影响程序执行效率
- 2.修改产品中的bug时，可以确保修改错误不会引入新的问题

## Q2：怎么开始和停止？

下面给出了开始和停止的方法

```csharp
=========================Start==================================
#Start Testing an Application
To test an application you must first add the Application to the verifier tool. 
You can then select the tests you want enabled, save your test setup, and run your application.

1.On the File menu, select Add Application.Browse to the application and click Open
2.select the tests to run
3.When the tests you want enabled have all been selected, click Save

Note：设置好后，不论appvrif.exe是否运行（主要是修改注册表中某项，继续运行程序会读取注册表），应用程序每次运行都会执行相应的校验 

=========================Stop==================================
#To remove an application from Application Verifier
1.Select the name of the application
2.On the File menu, select Delete Application .
3.Click Save . 

Note: 即使删除了相应的程序, 以前产生log文件也不会删除
```

## Q3：怎么查看测试设置项？

每个测试项都是有一些属性的，可以通过下面的方式详细进行查看

> 1.显示属性设置区

```cpp
#To view the Property window
On the View menu, select Property Window . A check mark while appear next to the Property Window selection.
```

属性设置区**默认是关闭的**，可以用下面方式开启；里面会显示具体某个测试项目的详细参数设置和描述信息

![在这里插入图片描述](https://img-blog.csdnimg.cn/3a3b790ef3994bd296eb9887891c504e.png#pic_center)


> 2.鼠标右键停在测试项上

- 选择`Properties`：允许控制测试设置的属性，注意并不是所有的测试设置都有

![在这里插入图片描述](https://img-blog.csdnimg.cn/eacfbe46f9184faba989315926b554da.png#pic_center)


- 选择`Verifier Stop Options`：设置验证停顿相关属性

![在这里插入图片描述](https://img-blog.csdnimg.cn/51648241bb8648649c0c80f42d54446b.png#pic_center)


其中：`Not Continuable`表示验证停顿被触发时，将中断到调试器中且不会从验证停顿中恢复过来（**效果就是停在调试器里**）

> 提示：其他具体属性不用详细介绍，一看就明白；具体每一个暂停码的相关信息可以参考`appverif.chm的Stop Codes and Definitions`章节
>
> 下面是`验证停顿码7`的详细信息，包含产生的可能原因和产生结果的格式
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/b36c984c2b244514a36346154cc606db.png#pic_center)


## Q4：测试结果怎么查看？

正常应用程序验证器使用流程：对某个程序用验证器（`appverif.exe`）启动相应设置，启动程序，此时验证器（`appverif.exe`）会在后台运行；观察测试结果有2种方式：

- 1.查看相应的日志文件

`View`菜单下面的`Logs`可以查看

![在这里插入图片描述](https://img-blog.csdnimg.cn/016457705d854de88f0ee2dfe44ce5de.png#pic_center)


- 2.调试器查看：即配合WinDbg等工具进行调试，在WinDbg里查看；这个就先不说了，后面会看到调试器中的错误信息发生了变化

> 提示：鼠标直接停在相应测试设置上查看简短描述，会看到测试结果需要`log查看`还是`调试器查看`
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/6ec374e5b05d4af881213de72dbfc9c3.png#pic_center)


## Q5：完全页堆和普通页堆？

> **注意：默认是开启完全页堆的**，所以需要说明一下页堆，页堆是什么？
>
> 作用：就是可以`在堆被破坏时立刻发现问题`，而不是等到下次调用堆函数时才发现
>
> 原理：简单理解是在堆块后面增加专门用于检测的栅栏页（fense page），一旦用户数据区溢出并触发到栅栏页（fense page）就会立刻触发异常，而不是等到下次调用堆函数才发现问题，会立刻发现问题；堆块前面也可以添加栅栏页
>
> 资料：如果想详细了解一下页堆的知识，可以看[这里](文档已经写完，整理好后，后续在这里贴上链接)
>
> 文档：`appverif.chm`的`Debugging Heap Errors`章节有介绍页堆，页堆实际上有2种：完全页堆和普通页堆，下面是《软件调试》中的截图中两者的区别
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/d9bcd9ff12b3468da2655d220c434438.png#pic_center)


## Q6：修改被进程捕获时机？

下面是应用程序验证配置后，程序的的大致流程：

- 1.进程加载模块

进程加载器会先将可执行程序（exe）和`ntdll.dll`这个2个模块进行加载，`ntdll.dll`中`ldr`开头的函数负责主要加载工作

- 2.加载器初始化

会从下面的注册表键下寻找当前程序的可执行选项（**`捕获位置`**）

`\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\程序名.exe`

- 3.根据可执行设置，决定动态修改IAT
- 4.找到EP处执行程序

# 0x03 释放同一块内存2次示例

下面使用1个堆内存释放二次的示例来讲解应用程序验证器的使用，将代码build成名字为`heap_test`的可执行文件

```cpp
//两次释放同一块内存
//直接运行现象：单独运行，程序会崩溃；如果指定了JIT调试器会触发
//调试器中运行现象：如果被应用程序验证器配置过，会发现错误的准确类型（双重释放）
#include "StdAfx.h"
#include <stdio.h>
void main(int argc, char* args[]) {
	char *psz = NULL;
	psz = new char[10];
	delete[] psz;
	delete[] psz;			//二次释放

	return;
}
```

下图是没有开启应用程序验证器相关验证功能的调试信息，可以看到是因为指令异常中断到调试器（错误码（`0x80000003`）和提示信息没有什么有价值的信息）

> 提示：这个截图是使用WinDbg查bug的经典截图，先记住它；和后面的开启应用程序校验后的WinDbg截图对比查看，看看哪个效果更好？

![在这里插入图片描述](https://img-blog.csdnimg.cn/410811769bd748f699cc10b4e6ebb72c.png#pic_center)


`想要进一步定位没有思路怎么办？`通常都是`kb`等一系列WinDbg，这里就不介绍了

如果你对WinDbg也不是非常熟悉，有没有什么傻瓜式的操作可以获得程序崩溃时更多的信息呢？下面介绍的应用程序验证器或许你可以试试

---

> 提示：下面用2种方法来介绍应用程序校验
>
> - 1.应用程序验证器（`appverif.exe`）方法：主要侧重对应用程序校验的初步认识
> - 2.`gflags.exe`方法：主要为了扩展应用程序校验的配置方法和理解校验器钩子（原理）

> **直接先总结2种方法的区别**：应用程序验证器（`appverif.exe`）设置一个程序后，注册表项会出现`GlobalFlag`和`VerifierFlags`等值；`gflags.exe`只会影响`GlobalFlag`这一个注册表值
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/4727b537c7064f4f9f739dafb729a126.png#pic_center)


# 0x04 应用程序验证器应用

先简单介绍一下工具的基本使用

## step 1.配置

### 生效

对源码生成的执行文件（`heap_test.exe`）进行默认配置截图如下

> 提示：第一次使用的话，一般将Basics配置上就行，后续熟练了根据自己的要求进行配置，此时页堆也会被默认打开

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9ea53f8101d430b8d71478c9acaa1d5.png#pic_center)


### 注册表影响

配置实际上会影响全局标志，注册表中查看的截图如下，可以看到应用程序验证器中设置的测试项目和注册表都是一一对应的

![在这里插入图片描述](https://img-blog.csdnimg.cn/72a2d7b5a187488fa5ba703411419870.png#pic_center)


其中的部分信息使用WinDbg在全局变量中也可以查到

![在这里插入图片描述](https://img-blog.csdnimg.cn/dd148a288b874dd0beb81e8bb9f12d74.png#pic_center)


### 取消

想要使某一个进程的设置失效，有下面简单的2种方式：

- 在`appverif.exe`中使用`Ctrl + D` 

  > 注意：此时注册表中还会有一点残留，最好也一起删除，防止后续使用WinDbg开启进程出现问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/68f8aecd87c141be944929a057f0201b.png#pic_center)


- 直接在注册表中删除对应的项（因为工具实际就是影响了注册表中的项而已）

## step 2.WinDbg调试

配置好验证的项后，WinDbg下重新运行一下程序，发现页堆和应用程序都使能了

```cpp
0:000> vercommand
command line: '"C:\Program Files (x86)\Windows Kits\10\Debuggers\x86\windbg.exe" '

#查看全局标志
0:000> !gflag
Current NtGlobalFlag contents: 0x02000100
    vrf - Enable application verifier					#应用程序验证器生效，0x00000100
    hpa - Place heap allocations at ends of pages		#页堆生效，0x02000000
```

继续让程序运行，**这次看看出问题时的提示信息是否会有变化？**

`错误提示信息更加具体了`，再也不是错误码（0x80000003）这种没啥有用信息的提示了，就冲这一点，就一定要学习应用程序验证器使用

```csharp
#运行，会产生异常中断到调试器，检测出现问题的信息更全面
0:000> g
=======================================
VERIFIER STOP 00000007: pid 0x16630: Heap block already freed. 					#堆块已经被释放过了
	07F71000 : Heap handle for the heap owning the block.
	07F72BFC : Heap block being freed again.				
	0000000A : Size of the heap block.											#堆块大小
	00000000 : Not used
=======================================
This verifier stop is not continuable. Process will be terminated 
when you use the `go' debugger command.
=======================================
(16630.16634): Break instruction exception - code 80000003 (first chance)		#指令异常
eax=7c4e03a0 ebx=00000000 ecx=000001a1 edx=006ff2a1 esi=07f71000 edi=7c4ddda0
eip=7c4d3bf8 esp=006ff58c ebp=006ff794 iopl=0         nv up ei pl nz na po nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000202
vrfcore!VerifierStopMessageEx+0x5c8:
7c4d3bf8 cc              int     3
```

- **解释**：VERIFIER STOP `00000007`，停顿码7代表重复释放的堆块；**如果对同一个堆块释放2次，就会触发这个停顿（这也太直接了吧...）**

- **根因**：应用程序验证器已经将哪个堆块（07F72BFC），什么原因（如果对同一个堆块释放2次）产生的指令异常显示出来

> 推荐：如果这个二次释放是在特殊场景下才触发且程序代码量很大，VS调试可执行程序时是很难发现的，因此推荐使用应用程序验证器配合调试器查问题的bug

> 提示：都提示出二次释放导致程序崩溃了，那剩下的操作就是review代码里能引起二次释放的代码；如果有源码可以使用`k`等命令查看栈帧
>
> ```csharp
> 0:000> k
> # ChildEBP RetAddr  
> 00 006ff794 7c4d83c0 vrfcore!VerifierStopMessageEx+0x5c8	        #停止消息
> 01 006ff7b8 7935df34 vrfcore!VfCoreRedirectedStopMessage+0x80
> 02 006ff80c 7935bb4f verifier!VerifierStopMessage+0x84
> 03 006ff878 793589dd verifier!AVrfpDphReportCorruptedBlock+0x1cf	#堆块异常
> 04 006ff8dc 79358b35 verifier!AVrfpDphFindBusyMemoryNoCheck+0x7d
> 05 006ff900 7935cde1 verifier!AVrfpDphFindBusyMemory+0x15
> 06 006ff918 79362572 verifier!AvrfpDphCheckPageHeapAllocation+0x41
> 07 006ff928 7c526123 verifier!VerifierCheckPageHeapAllocation+0x12				
> 08 006ff970 00b61314 vfbasics!AVrfpHeapFree+0x53					#调用vfbasics.dll版本的释放函数
> 09 006ff984 00b61016 heap_test!free+0x1c [f:\dd\vctools\crt_bld\self_x86\crt\src\free.c @ 51] 
> 0a 006ff998 00b61298 heap_test!main+0x16 [c:\...\heap_test.cpp @ 11] 
> ```

---

**扩展：验证结果在调试器中输出的格式**

```csharp
#验证结果在调试器中输出的基本：其中的参数列表取决于被执行的验证类型
=======================================
VERIFIER STOP <stop-code>: <porcess-ID>: 对故障的简单描述 
	parameter -1 : 描述
	parameter -2 : 描述			
	parameter -3 : 描述	
	parameter -4 : 描述
=======================================
```

---

## step 3.扩展命令(!avrf)

下面介绍一个WinDbg中非常有用的扩展命令

NT全局标志可以使用扩展命令`!gflag`进行查看，WinDbg中命令`!avrf`可以用来查看所有调试进程的校验标志

```csharp
#官网中关于!avrf的部分摘要
The !avrf extension controls the settings of Application Verifier and displays a variety of output produced by Application Verifier.

#语法：
!avrf
The !avrf command without any parameters shows the Application Verifier settings and information about the current and previous Application Verifier breaks if any.
!avrf -hp { Length | -a Address }
Displays the heap operation log. Address specifies the heap address. Records of the heap operations containing this heap address will be displayed.
...

#Comments
#展示验证配置
When the !avrf extension is used with no parameters, it displays the current Application Verifier options.
#获取验证停顿码和相关原因
If an Application Verifier Stop has occurred, the !avrf extension with no parameters will reveal the nature of the stop and what caused it.
#缺少符号信息
If symbols for ntdll.dll and verifier.dll are missing, the !avrf extension will generate an error message. 
```

> 提示：如果`!avrf`命令显示失败，大概率是因为`vfbasics.dll`的路径问题（老版本微软的符号服务器有这个dll的公共符号，但是没有函数扩展所需要的类型，因此会解析失败，新版没有遇到这个问题）；因为`vfbasics.dll`放入符号会一同安装在system32目录下，所以解决的办法是设置符号路径时，使本地system32的路径出现在微软的符号服务器前

下面是命令使用的截图，省去了还要查找错误码的过程，直接给出了简要信息

```csharp
#查看NT全局标志
0:000> !gflag
Current NtGlobalFlag contents: 0x02000100
    vrf - Enable application verifier
    hpa - Place heap allocations at ends of pages

#查看所有调试进程的校验标志（为了显示全面，下面是使用appverif.exe默认检查项设置的结果）
0:000> !avrf
Verifier package version >= 3.00 
Application verifier settings (81643027):
   - full page heap						#与appverif.exe默认检查项一一对应
   - Handles
   - Locks
   - Memory
   ...
   - SRWLock
*******************************************************************************
*                        Exception Analysis                                   *
*******************************************************************************
APPLICATION_VERIFIER_HEAPS_DOUBLE_FREE (7)								#原因：堆被释放二次
Heap block already freed.
This situation happens if the block is freed twice. Freed blocks are marked in a
special way and are kept around for a while in a delayed free queue. ...

Arguments:
Arg1: 08051000, Heap handle for the heap owning the block. 				#得到的详细信息
Arg2: 08052c64, Heap block being freed again. 
Arg3: 0000000a, Size of the heap block. 
Arg4: 00000000, Not used 

NTGLOBALFLAG:  2000100
ExceptionAddress: 799e3bf8 (vrfcore!VerifierStopMessageEx+0x000005c8)
   ExceptionCode: 80000003 (Break instruction exception)
FAILURE_BUCKET_ID:  BREAKPOINT_AVRF_80000003_vrfcore.dll!VerifierStopMessageEx
```

# 0x05 系统支持的校验器

下面主要介绍一下通过`gflags.exe`设置程序的校验器

> 提示：gflags也是一个很强大的工具，有时间最好也要学习一下

## step 1.配置

`gflags.exe`工具为示例程序启动操作系统支持的默认校验器

![在这里插入图片描述](https://img-blog.csdnimg.cn/0449d353060d4a0d9c2025c267172243.png#pic_center)


上面实际上是设置了进程映像的NT全局标志为0x100（`vrf`，应用程序验证器），即告诉操作系统我们想要进程后续实例启动`公共应用程序校验器钩子`（hook，简单理解就是会hook掉一部分函数）

上面设置会影响2个地方：

- 1.全局标志GlobalFlag映像文件执行选项（`Image File Execution Options`）

![在这里插入图片描述](https://img-blog.csdnimg.cn/cbd0049d83ae483b8e7afa01b75d90f8.png#pic_center)


- 2.一个叫`verifier.dll`的dll紧随`ntdll.dll`被加载（使用`appverif.exe`设置也会影响）

当在用户态调试器下运行目标进程，`verifier.dll`紧随`ntdll.dll`被加载，并映射到进程的地址空间中；这个`verifier.dll`模块包含了操作系统的核心校验引擎

```csharp
#查看NT全局标志
0:000> !gflag
Current NtGlobalFlag contents: 0x02000100
    vrf - Enable application verifier
    hpa - Place heap allocations at ends of pages
#列出模块    
0:000> lm
start    end        module name
00b70000 00b8d000   heap_test   (deferred)             
75590000 7578f000   KERNELBASE   (deferred)             
76900000 769e0000   KERNEL32   (deferred)             
77510000 776aa000   ntdll      (pdb symbols)   c:\symbols\...\wntdll.pdb
79350000 793b3000   verifier   (deferred)  				#这个模块 应用程序校验器 相关模块          

#查看verifier.dll模块的详细信息
0:000> lmv m verifier
Browse full module list
start    end        module name
79350000 793b3000   verifier   (deferred)             
    Image path: C:\WINDOWS\SysWOW64\verifier.dll
    Image name: verifier.dll
    FileDescription:  Standard application verifier provider dll
```

> 页堆默认设置？
>
> - 提示
>
> 细心的话会发现，`hpa（页堆）`也被打开了，但是全局标志`GlobalFlag`映像文件执行选项（`Image File Execution Options`）和`gflags /p`查询都没有开启hpa的进程
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/7952f8dabaa44ad19029814657d45fa0.png#pic_center)

>
> - 关联
>
> 实际是应用程序验证器的默认测试设置中的heap里面默认选取了页堆
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/b1b12651569c468ebeb86b6eb1ab5540.png#pic_center)


## step 2.系统默认启动项

当设置NT全局标志中应用程序校验位时，操作系统会默认启动相关的校验位，处理检查就是系统默认的启动项

![在这里插入图片描述](https://img-blog.csdnimg.cn/29153d2a2fa64e46b5d64e86f27ce6a2.png#pic_center)


## step 3.校验器检查原理

在调试器下继续运行程序，会命中一个校验中断，显示的类型是` block already freed`的错误

```csharp
0:000> g
===========================================================
VERIFIER STOP 00000007: pid 0x18424: block already freed 
	066E1000 : Heap handle
	06893968 : Heap block
	0000000A : Block size
	00000000 : 
===========================================================
(18424.183a4): Break instruction exception - code 80000003 (first chance)
verifier!VerifierBreakin+0x42:
7935dd72 cc              int     3
0:000> k
 # ChildEBP RetAddr      
00 007ef84c 7935de70     verifier!VerifierBreakin+0x42	#中断是verifier.dll产生
01 007efb74 7935e16d     verifier!VerifierCaptureContextAndReportStop+0xf0
02 007efbb8 7935bb4f     verifier!VerifierStopMessage+0x2bd
03 007efc24 7935c09a     verifier!AVrfpDphReportCorruptedBlock+0x1cf
04 007efc80 7935ce0b     verifier!AVrfpDphCheckNormalHeapBlock+0x11a
05 007efca0 79362572     verifier!AvrfpDphCheckPageHeapAllocation+0x6b
06 007efcb0 79377cd6     verifier!VerifierCheckPageHeapAllocation+0x12
07 007efd08 00b71314     verifier!AVrfpHeapFree+0x56	#free被改变转向verifier.dll的同名函数
08 007efd1c 00b71016     heap_test!free+0x1c [f:\...\free.c @ 51] 
09 007efd30 00b71298     heap_test!main+0x16 [c:...\heap_test.cpp @ 8] 
```

**校验器检查原理**：主要是更新部分常见win32 API函数调用对应的导入表（`IAT`）项来截取对API的调用，验证是否符合约定的使用方法，因此会有应用程序校验钩子的说法

下面是运行时的截图，可以看到`free`由原本调用`HeapFree函数`，被hook到调用`verifier.dll的AVrfpHeapFree函数`

![在这里插入图片描述](https://img-blog.csdnimg.cn/cde7b1e415094491bc83fbd0f3724af9.png#pic_center)


> 提示：这些hook方法只针对用户态代码上下文，内核驱动程序则与另一个工具相关（驱动校验工具`verifier.exe`）

# 0x06 总结

- 1.hook常见API

应用程序验证工具将常见操作系统API的导入表替换成特定调用来实现，在执行真实功能前，会先执行额外的校验检查来发现API误使用

- 2.本地特性

应用程序验证器（`appverif.exe`）只在本地用户态运行时分析使用，内核态要用驱动校验工具（`verifier.exe`）

- 3.目标程序最好运行在调试器中

应用程序校验中断作为运行时断言（SEH异常）提出，如果没有运行在调试器下且没有设置JIT调试器，异常`不会`被处理只会简单挂掉；因此尽量使目标程序最好运行在调试器中，这样可以捕获到任何检验而停止

# 0x07 参考

- 1.《Windows编程调试技术内幕》6.2章节
- 2.《Windows高级调试》中相关部分
- 3.《软件调试》第2版，第19章
- 4.应用验证器的帮助文档（appverif.chm）
