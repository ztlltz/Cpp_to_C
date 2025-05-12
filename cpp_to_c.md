[toc]

# CPP to C

---



## 引言

如果你曾经被c语言的繁琐操作劝退到cpp，那么当你的cpp大成后，你可能会想知道一件事：

cpp那些好用的stl，在c语言里面能不能也被很优美的实现。

比如像vector,map这种不可能不用的数据结构，或者像sort这种的函数。

作者虽然目前cpp并没有大成，但是由于最近工作上的事情不得不重新品味c的语法。伴随着学习笔记的性质本文也就诞生了。

本文将分为很多小章节，每个章节将介绍一些cpp到c的转换。

---

##  排序函数

说到排序函数，在cpp中我们往往只有一个选择，那就是使用sort函数。sort函数的构成分为两大块，

* 一是cmp函数，即小于号的定义

  ```c++
  int cmp(int a,int b){
      return a<b;
  }
  ```

  

* 二是sort函数的使用

  ```c++
  sort(v.begin(),v.end(),cmp);
  ```

那么在c语言里面有一个类似的实现，就是qsort函数，它同样由两部分构成，

* cmp函数

  ```c
  int cmp(const void* a,const void* b){
      return *((int*)a)-*((int*)b);
  }
  ```

* qsort使用(假设v是int类型的数组，vSize是v的长度)

  ```c
  qsort(v,vSize,sizeof(int),cmp);
  ```

  