## 基本类型

- bool		**1Byte**
  - false  0
  - true 1
- char    'a','c,' '      **1Byte**
- int     -2147483648-2147483647    **4Byte**
- float     1.23,2.5,     6-7位有效数字    **4Byte**
- double    15-16有效数字   **8Byte**
- long long (long long int)    -2^63~  -2^63  -1   **8Byte**
- long double    18-19位有效数字   **16Byte**



18KB

8Mb

1Byte=8bit

level 1 语法课 主要面向C ++
level 2 基础课 主要是写裸题 理解遇到的算法和其他知识点，是学算法模板的，但是基础课不等于简单课
level 3 提高课 这个是对应基础课学到的算法模板，学会了写模板提，不代表能应用，提高课就是应用课
level 4 进阶课 这个就是面向省选难度的了

浮点数比较

```c++
// 浮点数之间的比较 相差在一定精度之内 就算他们相等
const double eps=1e-6;
fabs(a-b) <= eps{
	相等
}
```



## C语言输入

- int %d
- float %f
- double %lf
- char %c
- long long %lld

## 输出



```c++
//保留5位小数
printf("%.5lf",double)
    
//整数格式化 不足在前边补上空格
printf("%5d",int);

//在右边补上空格
printf("%-5d",int);

//在高位补上0
printf("%05d",int);

//宽度为5 后边保留一位小数
printf("%5.1f",double);
```



## 判断

```c++
a > b
a < b
a >= b
a <= b
a == b
a != b
```

## 循环

```c++
// if版的循环
while

//先做一次  之后在进行判断
do{
    
}while();

//for循环
 for(init语句； 条件语句; 表达式){
     
 }
```

## 字符串

每一个字符都对应一个-128-127之间的数字，两者的关系通过ASCII表来进行查看。

常见ASCII  'A'-'Z' 65-90

'a'-'z' 97-122

```c++
//可以直接进行转化
cout << (int)char << endl;
cout << (char)int << endl;

//读取到空格或者换行为止
scanf("%s",s);
cin >> s;

//读一整行 把标准读入当成文件读入

char s[100];
fgets(s,1000,stdin);
cin.getline(s,100);

string s;
getline(cin,s);

//输出字符串
puts(s);
printf("%s\n",s);

//常用函数
#include<string.h>
//求长度 不包括\0
strlen(s)  
// stringcompare
strcmp(a,b)
// 把后一个复制给前一个
strcpy(a,b)

    
// 遍历字符串 要把字符串当成字符数组来进行操作 
```

## 指针

```c++
//输出一个地址
cout << (void*)&c << endl;

int a=10;
int* p=&a;
*p=12;
cout << *p << endl;

//引用 别名
int& po=p;
```

## STL

### Vector

#include<vector>

- size 实际长度
- empty 是否为空
- clear 清空
- begin 返回第一个元素
  - *a.begin()与a[0]
- end 返回尾部
- front 第一个元素
- back 最后一个元素
- push_back()  插入到尾部
-  pop_back() 删除最后一个元素

### Queue

#include'<'queue'>'

- push 从队尾插入
- pop 从队头弹出
- front 返回队头元素
- back 返回队尾元素

### Set

set和multiset两个容器，分别是“有序集合”和“有序多重集合”，

即前者的元素不能重复，而后者可以包含若干个相等的元素。

- insert 插入 logn
- find 查找等于x的元素

### Map

#include <map>

size/empty/clear/begin/end

- Insert/erase

​		与set类似，但其参数均是pair<key_type, value_type>。

- find

​		h.find(x) 在变量名为h的map中查找key为x的二元组。

## 常用库函数

- reverse 翻转

翻转一个vector：

reverse(a.begin(), a.end());

翻转一个数组，元素存放在下标1~n：

reverse(a + 1, a + 1 + n);

 

- unique 去重

返回去重之后的尾迭代器（或指针），仍然为前闭后开，即这个迭代器是去重之后末尾元素的下一个位置。该函数常用于离散化，利用迭代器（或指针）的减法，可计算出去重后的元素个数。

把一个vector去重：

int m = unique(a.begin(), a.end()) – a.begin();

把一个数组去重，元素存放在下标1~n：

int m = unique(a + 1, a + 1 + n) – (a + 1);

 

-  random_shuffle 随机打乱

用法与reverse相同

 

- sort

对两个迭代器（或指针）指定的部分进行快速排序。可以在第三个参数传入定义大小比较的函数，或者重载“小于号”运算符。

 

把一个int数组（元素存放在下标1~n）从大到小排序，传入比较函数：

 

int a[MAX_SIZE];

bool cmp(int a, int b) {return a > b; }

sort(a + 1, a + 1 + n, cmp);

 

把自定义的结构体vector排序，重载“小于号”运算符：

 

struct rec{ int id, x, y; }

vector<rec> a;

bool operator <(const rec &a, const rec &b) {

​		return a.x < b.x || a.x == b.x && a.y < b.y;

}

sort(a.begin(), a.end());

 

-  lower_bound/upper_bound  二分

lower_bound 的第三个参数传入一个元素x，在两个迭代器（指针）指定的部分上执行二分查找，返回指向第一个大于等于x的元素的位置的迭代器（指针）。

upper_bound 的用法和lower_bound大致相同，唯一的区别是查找第一个大于x的元素。当然，两个迭代器（指针）指定的部分应该是提前排好序的。

 

在有序int数组（元素存放在下标1~n）中查找大于等于x的最小整数的下标：

int I = lower_bound(a + 1, a + 1 + n,. x) – a;

 

在有序vector<int> 中查找小于等于x的最大整数（假设一定存在）：

int y = *--upper_bound(a.begin(), a.end(), x);