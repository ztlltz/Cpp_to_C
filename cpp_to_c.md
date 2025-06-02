[toc]

# CPP to C

---



## 0 引言

如果你曾经被c语言的繁琐操作劝退到cpp，那么当你的cpp大成后，你可能会想知道一件事：

cpp那些好用的stl，在c语言里面能不能也被很优美的实现。

比如像vector,map这种不可能不用的数据结构，或者像sort这种的函数。

作者虽然目前cpp并没有大成，但是由于最近工作上的事情不得不重新品味c的语法。伴随着学习笔记的性质本文也就诞生了。

本文将分为很多小章节，每个章节将介绍一些cpp到c的转换。

---

##  1 指针和字符串

### 1.1 内存管理

大多数情况下，我们都想在堆上开辟一个内存。在cpp中，基本上我们不用担心这个事情，因为cpp的stl都是开辟在堆上的。但是如果我们要在c里面实现这个东西呢，事情就变得不太简约起来。

以开辟一个二维空间举例：

对于cpp下面三行就能实现

```c++
int rows = 10;
int cols = 10;
vector<vector<int>>nums(rows,vector<int>(cols));
```

而c语言则如下

```c
int rows = 10;
int cols = 10;
int** nums = malloc(rows * sizeof(int*));
for(int i=0;i<rows;i++){
    nums[i] = malloc(cols * sizeof(int));
    memset(nums[i],0,col * sizeof(int)); //初始化为0
}
```

如果在cpp里面这就结束了，后面也不用管nums了，但是在c里面我们还得记得把他的空间释放了。

```c
for(int i=0;i<rows;i++){
   free(nums[i]);
}
free(nums);
```

总之写起来还是比较繁琐的。

同样还有个用起来比较繁琐的，就是字符串

由于c风格的字符串其实就是字符数组，所以定义字符串和定义数组是一样的。

需要注意的是，c语言默认字符串的结尾是'\0'，所以内存的分配要多预留一个空间。

---

那么我们来看看对于操作c字符串的一些常用的函数:

### 1.2 strdup

有时候我们想创建一个字符串来对另一个字符串进行深拷贝

这时候如果我们可以重新分配一段空间，然后赋值成和另一个字符串一样的值。

c语言的strdup将这套操作简化成了函数strdup，大致工作方式如下

```c
char  * destString = malloc(sizeof(char) * (strlen(sourceString)+1)); //记得放'\0'
for(...){
	destString[i] = sourceString[i];
}
return destString;
```

观察他的工作方式我们发现strdup的复杂度是O(n)的。

使用起来如下:

```c
char  * destString = strdup(sourceString);
```

需要注意的是，这个函数要求sourceString非NULL，同时如果内存分配失败，函数会返回NULL。

当然，还有一件事，记得最后把他手动free掉

```c
free(destString);
```

---

### 1.3 strlen

strlen有点类似string.size()，不同点在于它的复杂度是O(n)，而且他会不计算'\0'的空间。

所以要注意在内存分配时string的真实大小是strlen+1，但是我们谈论字符串长度时往往不加1。

```c
int stringSize = strlen(string);
```

---

### 1.4 memcpy_s

memcpy不是一个专门操作字符串的函数，但是在字符串中出现比较频繁。它支持我们将一个内存的值复制给另一个内存

```c
memcpy_s(dest,lenthDest,source,lenthSource); // lenth为内存大小
```

还有一个和他长得比较像的是memset

### 1. 5 memset

这个字符串就不太常用，它是把每个字节赋值成一个值

```c
memset(ptr,0,lenth); // lenth为内存大小
```

往往我们用它来将内存赋值为0。

### 例题 1

>给定一个字符串A，请返回一个新的字符串B，满足B = A + A' ,其中A'为A的反转字符串。
>
>比如，当A为“string”时，A'为“gnirts”，B为“stringgnirts”

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
// 要传入指针B和长度lenOfB的地址，否则实际上更改的是参数的副本
void concatWithReverse(char* A, int lenOfA, char** B, int* lenOfB) {
	*lenOfB = lenOfA * 2;
	*B = (char*)malloc(sizeof(char) * ((*lenOfB) + 1)); // 预留一个‘\0’;
	(*B)[*lenOfB] = '\0';
	memcpy_s((*B), (*lenOfB) * sizeof(char), A, lenOfA * sizeof(char));
	for (int i = 0; i < lenOfA; ++i) {
		(*B)[i+lenOfA] = A[lenOfA - i - 1];
	}
}
int main() {
	char *A ,*B;
	int lenOfA = 10;
	int lenOfB = 0;
	A = strdup("12345abcde");
	concatWithReverse(A, lenOfA, &B, &lenOfB);
	printf("%d\n%s", lenOfB, B);
    
    free(A);
    free(B);
	return 0;
}

```



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

例题 2

> 给定一个包含数字和字母的字符串，请返回他的升序版本，其中规定，大写字母>数字>小写字母
>
> tip: ASCII里小写字母>大写字母>数字
>
> 比如 
>
> 输入:
>
> ThisIs1Test  
>
> 输出：
>
> ITTehissst1

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
int getType(char a){
	if(a>='0'&&a<='9'){
		return 2;
	}
	if(a>='a'&&a<='z'){
		return 1;
	}
	if(a>='A'&&a<='Z'){
		return 0;
	}
}
int cmp(const void *a, const void *b){
	char aa= *((char*)a);
	char bb= *((char*)b);
	int typeA = getType(aa);
	int typeB = getType(bb);
	if(typeA!=typeB){
		return typeA-typeB;
	}
	else{
		return aa - bb; 
	}
}
void SortStringWithNewRule(char *string){
	qsort(string,strlen(string),sizeof(char),cmp);
	for(int i=0;i<strlen(string);i++){
		printf("%c",string[i]);
	}
	printf("\n");
}
int main(){
	char * inputString = strdup("ThisIs1Test");
	SortStringWithNewRule(inputString);
	free(inputString);
}
```



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
  for (HashItem *pEntry = mp; pEntry; pEntry = pEntry->hh.next) {
          res[pos++] = pEntry->key;
  }
  //排序哈希表
  HASH_SORT(mp, cmp);
  ```
  
  ---

