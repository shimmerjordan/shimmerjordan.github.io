---
title: 关于ios::sync_with_stdio(false)和cin.tie(nullptr)
date: 2020-02-21 10:47:50
tags: 
	- C++效率
	- Lambda表达式
	- C++特性
categories: C++效率
id: iosSyncWithStdio
---
# 前言
我们可以发现C++高效率解决的源码中有很多以下代码段：
```C++
static const auto io_sync_off = []() {
    // turn off sync  
    std::ios::sync_with_stdio(false);  
    // untie in/out streams  
    std::cin.tie(nullptr);  
    return nullptr;  
}();  
```
那么ios::sync_with_stdio(false);和cin.tie(nullptr);到底咋C++效率问题上起什么作用呢？这篇文章简要说明其起源与用法。
<!--more-->

# 起源
ios::sync_with_stdio(false)和cin.tie(nullptr)均来自Lambda表达式：
```C++
static const auto io_sync_off = []() {
    ... ...
}();
```
这个乍一看很像是函数，但是前后又是[]又是()的，是Lambda表达式即匿名函数。

>Lambda表达式是C++11引入的特性，是一种描述函数对象的机制，它的主要应用是描述某些具有简单行为的函数。Lambda也可以称为匿名函数。

Lambda表达式完整的声明格式如下：
```C++
[capture list] (params list) mutable exception-> return type { 
	function body 
}
```
各项具体含义如下:

- capture list：捕获外部变量列表  
- params list：形参列表  
- mutable指示符：用来说用是否可以修改捕获的变量  
- exception：异常设定  
- return type：返回类型  
- function body：函数体  

这里不再赘述其详细内容，总而言之等价为：

```C++
static const auto function() {
    ... ...
}
static const auto io_sync_off = function();
```
# std::ios::sync_with_stdio(false)

iostream默认是与stdio关联在一起的，以使两者同步，因此消耗了iostream不少性能。C++中的std :: cin和std :: cout为了兼容C，保证在代码中同时出现std :: cin和scanf或std :: cout和printf时输出不发生混乱，所以C++用一个流缓冲区来同步C的标准流。通过std :: ios_base :: sync_with_stdio函数设置为false后可以解除这种同步，让std :: cin和std :: cout不再经过缓冲区，iostream的性能就会提高了很多倍。因此，当解除同步之后，注意不要与scanf和printf混用以免出现问题。

# std::cin.tie(nullptr)
nullptr是c++11中的关键字，表示空指针

NULL是一个宏定义，在C和C++中的定义不同，c中NULL为（void\*) 0，而C++中NULL为整数0
nullptr是一个字面值常量，类型为std::nullptr_t，空指针常数可以转换为任意类型的指针类型。在c++中（void\*)不能转化为任意类型的指针，即 int \*p=(void\*)是错误的，但int \*p=nullptr是正确的。原因是对于函数重载：若C++中 （void\*）支持任意类型转换，函数重载时将出现问题。下列代码中fun(NULL)将不能判断调用哪个函数
```C++
void fun(int i)  {cout<<"1";};
void fun(char *p)  {cout<<"2";};
int main() {
	fun(NULL);        //输出1，c++中NULL为整数0
	fun(nullptr);     //输出2，nullptr 为空指针常量。是指针类型
}
```
tie是将两个stream绑定的函数，空参数的话返回当前的输出流指针。

std :: cin默认是与std :: cout绑定的，所以每次操作的时候都要调用fflush，这样增加了IO的负担，通过tie(nullptr)来解除std :: cin和std :: cout之间的绑定，进一步加快执行效率。
