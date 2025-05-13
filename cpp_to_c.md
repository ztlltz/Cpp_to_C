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


复杂度如下表：

| 操作  | 复杂度 |
| ----- | ------ |
| qsort | nlogn  |
| sort  | nlogn  |

---

## 哈希表

哈希表在cpp中我们有两种类似的实现，一种是map，另一个是unordered_map,其中map的底层实现是红黑树，而unordered_map是纯纯的哈希表实现。

对于cpp的哈希表我们常用的操作有以下几种：

* 初始化

  ```c++
  unordered_map<int,int>mp;
  ```

* 插入

  ```c++
  // 方法一
  mp[key]=value;
  // 方法二
  mp.insert({key,value});
  ```

* 查询key对应的value

  ```c++
  //查询key存不存在(flag = 0 or 1)
  int flag = mp.count(key);
  // 获取对应的value
  auto value = mp[key];
  ```

* 删除key的键值对

  ```c++
  mp.erase(key);
  ```

c语言并没有原生的哈希表实现，但是有一个开源库，uthash

* 初始化

  ```c
  #include "uthash.h"
  typedef struct {
      int key;
      UT_hash_handle hh;
  } HashItem; 
  HashItem * mp = NULL;
  ```

* 插入

  ```c
  // 查询key存不存在（pEntry = NULL or key的地址）
  HashItem *hashFindItem(HashItem **obj, int key) {
      HashItem *pEntry = NULL;
      HASH_FIND_INT(*obj, &key, pEntry);
      return pEntry;
  }
  // 插入元素
  // 支持插入INT,STR,PTR ,也可以HASH_ADD支持任意元素
  bool hashAddItem(HashItem **obj, int key) {
      if (hashFindItem(obj, key)) {
          return false;
      }
      HashItem *pEntry = 
          (HashItem *)malloc(sizeof(HashItem));
      pEntry->key = key;
      HASH_ADD_INT(*obj, key, pEntry);
      return true;
  }
  ```

* 删除元素

  ```c
  void hashFree(HashItem **obj) {
      HashItem *curr = NULL, *tmp = NULL;
      HASH_ITER(hh, *obj, curr, tmp) {
          HASH_DEL(*obj, curr);  
          free(curr);
      }
  }
  //删除所有元素
  HASH_CLEAR(hh,users);
  ```

* 杂项

  ```c
  //获取元素个数
  int size = HASH_COUNT(mp);
  //遍历
  for (HashItem *pEntry = mp; 
       pEntry; 
       pEntry = pEntry->hh.next) {
          res[pos++] = pEntry->key;
  }
  //排序哈希表
  HASH_SORT(mp, cmp);
  ```

  