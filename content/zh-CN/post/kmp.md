+++
title = "KMP算法详解"
date = "2021-07-03T16:50:29+08:00"
categories = [
    "算法"
]
tags = [
    "KMP算法",
    "字符串匹配"
]
toc = true

+++

期末没时间，水一篇KMP（Knuth-Morris-Pratt）算法。

KMP算法是一种字符串匹配算法，其实字符串匹配都是可以暴力求解的，KMP算法是进行了优化，减少了无效的匹配，从而节省时间。

<!--more-->

## 一道题目

题目描述

```
给定一个字符串（模式串）和一些待查找的字符串，求每个待查找字符串在模式串中出现的次数（可重叠）
若使用C++只能include一个头文件iostream；若使用C语言只能include一个头文件stdio
程序中若include多过一个头文件，不看代码，作0分处理
```

输入

```
测试数据有多组（测试组数 <= 5），
第一行包括一个字符串P，长度不超过105，且非空串
第二行包括一个整数N，代表待查找的字符串数量 (1 <= N <= 5)
接下来的N行，每一行包括一个待查找的字符串，其长度不超过50，且非空串
```

输出

```
对于每组测试数据，
输出每个待查找字符串出现的次数，
具体输出见样例
```

样例输入

```
aabbcc
3
aa
bb
cc
ababab
1
aba
```

样例输出

```
aa:1
bb:1
cc:1
aba:2
```

题目解析

这道题就是一个典型的在模式串中匹配子串的题目，我在上课时想快点敲完，就直接用暴力求解了。假设模式串长度为n1，子串长度为n2，暴力求解就是从模式串的s[0]到s[n1 - n2]，挨个对模式串进行匹配。

<img src="../kmpPic/Snipaste_2021-07-03_18-00-22.png" alt="Snipaste_2021-07-03_18-00-22" style="zoom:60%;" />

这样需要进行 n1 - n2 次匹配，然后每次匹配中，要匹配 n2 个字符，所以时间复杂度是O[(n1-n2)*n2]

额这样做在普通课程的oj上是可以通过的，但如果是ACM题的话是妥妥的时间超限。是蓝桥杯的话，也是不能通过后几个样例的。

AC代码

```c++
#include <iostream>
using namespace std;

const int LEN = 100001;

int main() {
	char* P = new char[LEN];
	char* sub = new char[LEN];
	int n;
	while (cin >> P >> n) {
		int  SIZE = 0;	//模式串长度
		for (; P[SIZE] != '\0'; SIZE++);	//不给引用库函数，自己求一下strlen

		for (int i = 0; i < n; i++) {	//输入子串
			cin >> sub;
			int size = 0;	//子串长度
			for (; sub[size] != '\0'; size++);

			int cnt = 0;
			for (int j = 0; j < SIZE - size + 1; j++) {
				int matched = 0;	//记录匹配的字符数
				for (int k = 0; k < size; k++)	//模式串片段与子串进行匹配
					if (P[j + k] == sub[k])
						matched++;
					else
						break;
				if (matched == size)
					cnt++;
			}
			cout << sub << ":" << cnt << endl;
		}
	}
	delete[] sub;
	delete[] P;
}
```

## KMP算法

KMP算法就是会省去很多无效的匹配。其实算法的过程是模拟了人的思维，大家可以体会一下。

用KMP算法进行字符串匹配时，我们用两个指针i，j分别指向<font style="background:BlanchedAlmond">**模式串当前匹配到的位置**</font>和<font style="background:BlanchedAlmond">**目标串当前匹配到的位置**</font>。在暴力求解中，i指针在每一round都要回溯到main_str[round]的位置，而在KMP算法中是不需要的。

<img src="../kmpPic/Snipaste_2021-07-04_00-22-08.png" alt="Snipaste_2021-07-04_00-22-08"  />

### 失配后，下一round从第几位开始匹配？

下图的例子，在round n+1发生了失配。

![Snipaste_2021-07-04_00-22-39](../kmpPic/Snipaste_2021-07-04_00-22-39.png)

此时i指针不动，j指针回溯，进行round n+2的匹配。

![Snipaste_2021-07-04_00-22-56](../kmpPic/Snipaste_2021-07-04_00-22-56.png)

因为在round n匹配s[2]时，可以理解成同时匹配了round n+2的s[0]字符，所以round n+2中，匹配只需从s[3]开始。

在对应位置匹配失败后，**j指针要回溯到目标串的位置取决于目标串中的重复串**。通常将回退位置储存在next数组里面。

## next数组

用KMP算法求解，第一步就是计算next数组，这个数组存的是，**子串的某一位发生失配后，需要从子串的第几位开始重新匹配，即失配后子串指针退回到的位置**。

### 求next数组代码

下面是求next数组的代码，非常抽象，下面解释代码为什么要这样子写。（因为几乎所有教程里，进行匹配时用的指针起名为i，j，求next时候的指针也起名为i，j，实在让新手看得一头雾水。这里为了说明这两对指针没有半毛钱关系，下面我用x，y来命名。）

```c++
void getNext(char* s, int*& next) {
	int len = strlen(s);
	int x = 0, y = -1;	//x为当前字符下标，y为寻找重复串的辅助指针
	next[0] = -1; //为方便求next和匹配时判断
	while (x < len - 1) {
		if (y == -1 || s[x] == s[y])
				next[++x] = ++y; //有重复串，则记录退回位置
		else
			y = next[y]; //不构成重复串，将y指针退回（具体看下面）
	}
}
```

再说一遍，求next数组就是**在子串中找重复串**，让失配后回退到对应的位置，减少无效的匹配。

<img src="../kmpPic/Snipaste_2021-07-03_21-47-44.png" alt="Snipaste_2021-07-03_21-47-44" style="zoom: 50%;" />

前三个字符没有构成重复串，所以失配时子串下标均退回到0。`line 9`保证了，没有构成重复串时，y指针是一直指向首位置的，因为`line 7`所以next[1]到next[3]的值都是0。

计算到x == 3时，s[x] == s[y]，即s[3]与s[0]构成重复串，所以如果在s[4]处发生失配，在失配前，s[3]是匹配成功的，说明s[0]也是会匹配成功的，所以下一round的匹配只需从s[1]开始。

同理，如果在s[5]发生失配，在下一round中只需从s[2]开始进行匹配。

<img src="../kmpPic/Snipaste_2021-07-03_22-45-00.png" alt="Snipaste_2021-07-03_22-45-00" style="zoom: 67%;" />

让这条字符串再多一位，当计算到x == 5时，y == 2，是不会构成重复串的，那`line 9`就能确保next[6]的值为0。对应的在匹配中的情况是，在s[6]处发生失配，j指针需要退回到0。

![Snipaste_2021-07-03_22-52-46](../kmpPic/Snipaste_2021-07-03_22-52-46.png)

## KMP代码

```c++
int KMP(char* s_main, char* s_aim) {
	int len1 = strlen(s_main);
	int len2 = strlen(s_aim);
	int* next = new int[len2];
	getNext(s_aim, next);
	int i = 0, j = 0; //模式串位置，目标串位置
	while (i < len1 && j < len2) {
		//j == -1,即s[0]匹配不成功，i后移，j回到位置0，或
		//匹配成功，两个指针均后移
		if (j == -1 || s_main[i] == s_aim[j])
			i++, j++;
		else
			j = next[j]; //失配，回溯j
	}
	if (j == len2) //匹配成功，返回模式串中目标串的位置
		return i - j;
	else
		return -1;
}
```

## 上面求next的缺陷

如果模式串是这样的，那目标串中的**每个a都要和模式串中的b匹配一下**，那round 2 到round 6的匹配都是无效的。因为在round 1 中已经确定了，目标串需要至少往后挪6位才有可能匹配到。

<img src="../kmpPic/Snipaste_2021-07-03_18-11-23.png" alt="Snipaste_2021-07-03_18-11-23" style="zoom:60%;" />

更一般的情况是下面这样的，在round n匹配失败后，回退到第一个`c`显然是没用的，应该再往前回退。

<img src="../kmpPic/Snipaste_2021-07-03_23-37-21.png" alt="Snipaste_2021-07-03_23-37-21" style="zoom:60%;" />

### 改进getNext

其实只用加个判断。如果求next时发现前面有一样的字符，则取前一个字符的next。

```c++
void getNext(char* s, int*& next) {
	int len = strlen(s);
	int x = 0, y = -1;	//x为当前字符下标，y为寻找重复串的辅助指针
	next[0] = -1;
	while (x < len - 1) {
		if (y == -1 || s[x] == s[y]) {
			if (s[++x] == s[++y]) //s[y]==s[x]字符相同，往回取next，避免无效匹配
				next[x] = next[y]; //意思是避免匹配了s[x],又倒回去匹配s[y]了（上图的c和c）
			else
				next[x] = y; //有重复串，则记录退回位置
		}
		else
			y = next[y]; //不构成重复串，将y指针退回
	}
}
```

往回取next不会影响y的值，这样遇到下面这种情况，求得的next数组是{-1, -1, -1, -1, -1, 5}。进行匹配时，在'b'处失配，后面的round也只会匹配最后两个字符。

<img src="../kmpPic/Snipaste_2021-07-03_19-18-51.png" alt="Snipaste_2021-07-03_19-18-51" style="zoom: 67%;" />

## 二道算法题——KMP算法

题目描述

```
学习KMP算法，给出主串和模式串，求模式串在主串的位置
```

输入

```
第一个输入t，表示有t个实例
第二行输入第1个实例的主串，第三行输入第1个实例的模式串
以此类推
```

输出

```
第一行输出第1个实例的模式串的next值
第二行输出第1个实例的匹配位置，位置从1开始计算，如果匹配成功输出位置，匹配失败输出0
以此类推
```

样例输入

```
3
qwertyuiop
tyu
aabbccdd
ccc
aaaabababac
abac
```

样例输出

```
-1 0 0
5
-1 0 1
0
-1 0 0 1
8
```

题目解析

通过观察样例，这道题是没有用优化过的next求法的。注意输出从1开始，搜索失败输出0。

AC代码

```c++
#include <iostream>
#include <string>
using namespace std;

void getNext(char* s, int*& next) {
	int len = strlen(s);
	int x = 0, y = -1;	//x为当前字符下标，y为寻找重复串的辅助指针
	next[0] = -1; //为方便求next和匹配时判断
	while (x < len - 1) {
		if (y == -1 || s[x] == s[y])
			next[++x] = ++y; //有重复串，则记录退回位置
		else
			y = next[y]; //不构成重复串，将y指针退回（具体看下面）
	}
}

void print(int* next, int len) {
	for (int i = 0; i < len; i++)
		cout << next[i] << ' ';
	cout << endl;
}

int KMP(char* s_main, char* s_aim) {
	int len1 = strlen(s_main);
	int len2 = strlen(s_aim);
	int* next = new int[len2];
	getNext(s_aim, next);
	print(next, len2);

	int i = 0, j = 0; //模式串位置，目标串位置
	while (i < len1 && j < len2) {
		//j == -1,即s[0]匹配不成功，移动i，j回到位置0
		//或匹配成功，两个指针均后移
		if (j == -1 || s_main[i] == s_aim[j])
			i++, j++;
		else
			j = next[j]; //失配，回溯j
	}
	if (j == len2) //匹配成功，返回模式串中目标串的位置
		return i - j;
	else
		return -1;
}

int main() {
	int t;
	const int MAX_LEN = 10001;
	char* s_main = new char[MAX_LEN];
	char* s_aim = new char[MAX_LEN];
	cin >> t;
	while (t--) {
		cin >> s_main >> s_aim;
		cout << KMP(s_main, s_aim) + 1 << endl;
	}
}
```

## 三道算法题——特殊的语言

题目描述

```
某城邦的语言，每个字是由两个字母构成的。
考古学家发现把他们的文字数字化之后，当想搜索特定的句子时，总会匹配到错误的地方。
比如一段文字是aabcdaabcdef，想要搜索abcd，应当搜到的是aabcda abcd ef
却会得到额外的一个并不符合该语言语法的结果a abcd aabcdef
（因为每个字由两个字符组成，这样匹配就把正确的“字”拆开了）。
请你帮他实现正确的匹配算法。
```

输入

```
每组数据两行，第一行为该语言的主串，第二行为模式串，
都由大写或小写英文字母组成，长度都不超过 10000，且一定为偶数个。
```

输出

```
每组数据输出正确匹配的次数
```

样例输入

```
abcdaabbab
ab
AbdcAbdcAbqAbdcAbdcAbp
AbdcAb
```

样例输出

```
2
2
```

题目解析

因为两个字母为一个字，第二个样例中

**AbdcAb**dcAbqAbdcAbdcAbp和

Abdc**AbdcAb**qAbdcAbdcAbp

均符合，有重叠，注意不要漏掉。

AC代码

```c++
#include <iostream>
#include <string.h>
using namespace std;

void getNext(char* s, int*& next) {
	int len = strlen(s);
	int x = 0, y = -1;	//x为当前字符下标，y为寻找重复串的辅助指针
	next[0] = -1;
	while (x < len - 1) {
		if (y == -1 || s[x] == s[y]) {
			if (s[++x] == s[++y])
				next[x] = next[y]; //后一个字符相同，往回取next，避免无效匹配
			else
				next[x] = y; //有重复串，则记录退回位置
		}
		else
			y = next[y]; //不构成重复串，将y指针退回
	}
}

int KMP(char* str, char* key) {
	int len1 = strlen(str);
	int len2 = strlen(key);
	int* next = new int[len2];
	getNext(key, next);

	int i = 0, j = 0, cnt = 0;
	while (i < len1) {
		if (j == -1 || str[i] == key[j])
			i++, j++;
		else
			j = next[j];
		if (j == len2) { //匹配成功
			if (i % 2 == 0) //判断匹配位置是否为2的倍数
				cnt++;
			i = i - len2 + 2; //回溯i，否则漏掉重叠
			j = 0; //回退j
		}
	}
	delete[] next;
	return cnt;
}

int main() {
	const int MAX = 10086;
	char str[MAX], key[MAX];
	while (cin >> str >> key) {
		int t = KMP(str, key);
		cout << t << endl;
	}
}
```

## 写在后面

额本来想水一下，然后不小心写详细了...

kmp算法有点难理解，我举得例子几乎涵盖了所有情况，如果还是看不懂的可以来问我=.=

## 参考资料和推荐阅读

有参考：https://www.cnblogs.com/dusf/p/kmp.html

有参考：http://data.biancheng.net/view/180.html

推荐阅读：https://zhuanlan.zhihu.com/p/83334559