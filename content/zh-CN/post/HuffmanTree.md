+++
author = "兔楠"
title = "最优二叉树和赫夫曼编码"
date = "2021-05-21"
description = "testing..."
tags = [
    "最优二叉树","二叉树","算法"
]
categories = [
    "算法","数据结构"
]
toc = true
+++




节点的**带权路径长度**是指**根节点到该节点的路径长度**与**节点权值**的**乘积**。最优二叉树（赫夫曼树or哈夫曼树）是指**带权路径和**最小的二叉树。构建赫夫曼树时，要使带权路径和最小，需要遵循**权重大的节点**离根节点更近。

<img src="../HuffmanTreePic/1f01e27c9ea87c9c3a214fc36a7dbfa.png" alt="1f01e27c9ea87c9c3a214fc36a7dbfa" style="zoom: 33%;" />

<!--more-->

## 构建赫夫曼树

有给定权值的n个节点：

1. 在n个节点中选出两个权值最小的节点，将两个节点组成一个二叉树，根结点的权值为左右子节点权值的和。

   <img src="../HuffmanTreePic/00900ca89da8eda12b48ade759aa85f.png" alt="00900ca89da8eda12b48ade759aa85f" style="zoom:33%;" />

2. 将上一步的根节点放入剩下的节点中，进行 1 。

<img src="../HuffmanTreePic/6f1b275d9cf0f38d3c6ea9bfbe89512.png" alt="6f1b275d9cf0f38d3c6ea9bfbe89512" style="zoom:33%;" />



## 赫夫曼编码

**问题**： 请设计变长的前缀码，对消息DEAACAAAAABA进行编码

对消息**DEAACAAAAABA**进行编码的前缀码编码方案很多，比如用ascii码之类的但是会很长因，因为每个字母的ascii码都要占**1个byte**。

消息中字符 A 出现了8次，字符 B、C、D 和 E 均只出现了1次；字符 A 用一个bit位表示，**A=0**，其他字符的编码均不能以0开头；

**B=10、C=11、D=110**和 **E=111**

**DEAACAAAAABA=110111001100000100**。这样得到比较短的编码。



### 编码规则

[<font color=CornflowerBlue>原文链接：哈夫曼编码详解——图解真能看了秒懂_Young_IT的博客-CSDN博客</font>](https://blog.csdn.net/Young_IT/article/details/106730343)


#### 直接上题目:
```
已知字符集  { a, b, c, d, e, f }，
若各字符出现的次数分别为{ 6, 3, 8, 2, 10, 4 }，
则对应字符集中各字符的哈夫曼编码可能是：(2分)

A. 00, 1011, 01, 1010, 11, 100

B. 00, 100, 110, 000, 0010, 01

C. 10, 1011, 11, 0011, 00, 010

D. 0011, 10, 11, 0010, 01, 000
```


#### 步骤一：
1.找最小两个次数（这里是2和3）

<img src="../HuffmanTreePic/a1da8a80f5cdb28737236178731dd0c.png" alt="a1da8a80f5cdb28737236178731dd0c" style="zoom:50%;" />

2.把他们放进树中（小左大右）

3.每次组合都多一个父节点（即2+3=5）

<img src="../HuffmanTreePic/256d05cf81bf7184d98084c27015bee.png" alt="256d05cf81bf7184d98084c27015bee" style="zoom:33%;" />

#### 步骤二：
1.再选出2个最小的数（排除上面已经选了的）——选出了4和6

<img src="../HuffmanTreePic/2ab1a369dd7fc2d2a2f33beb4ccb7de.png" alt="2ab1a369dd7fc2d2a2f33beb4ccb7de" style="zoom:50%;" />

2.因为4<5 , 6>5（5为步骤一中组合后的父节点）

3.单独拿4来跟5组合（小左大右）        【如果拿出的2个数都比5小，则这2个数自己组合后跟5组合，下面提到】

<img src="../HuffmanTreePic/d1518edb4ca8272ae3612ef3dcc042c.png" alt="d1518edb4ca8272ae3612ef3dcc042c" style="zoom:33%;" />

####　步骤三：
1.因为步骤二用掉了4，还没用6。现在取最小2个数

<img src="../HuffmanTreePic/ecef694185aaadcab7549796280694a.png" alt="ecef694185aaadcab7549796280694a" style="zoom:50%;" />

2.因为6 < 9 , 8< 9 所以6和8自己组合（小左大右） （组合后先放一边）

<img src="../HuffmanTreePic/7f96a2fd57bf82e2d292a34e8a6311a.png" alt="7f96a2fd57bf82e2d292a34e8a6311a" style="zoom:33%;" />

#### 步骤四：
1.取出最后10

<img src="../HuffmanTreePic/ee3a8a0e62c067da7080e5d13fd8ee8.png" alt="ee3a8a0e62c067da7080e5d13fd8ee8" style="zoom:50%;" />

2.10要和这两个子树根节点最小的组合（9<14,所以和9组合）（小左大右）

<img src="../HuffmanTreePic/e36eaf93beebd513b9e5fcdc88fc81c.png" alt="e36eaf93beebd513b9e5fcdc88fc81c" style="zoom:33%;" />

3.然后把14的子树组合上去（小左大右）  所以放左边

<img src="../HuffmanTreePic/1004e88577251220e377d5d8ffe7572.png" alt="1004e88577251220e377d5d8ffe7572" style="zoom:33%;" />

#### 步骤五：
组合完哈夫曼树后,将对应的字符填上去

<img src="../HuffmanTreePic/5fc05eacd8affd7ee64e2d8f372fc65.png" alt="5fc05eacd8affd7ee64e2d8f372fc65" style="zoom:33%;" />

#### 步骤六：
从根节点开始向下走往左为0，往右1。走到对应的字符的路径就是该字符的哈夫曼编码（左0右1）

<img src="../HuffmanTreePic/b49eedfff8e6f9893bcba38e3328ea2.png" alt="b49eedfff8e6f9893bcba38e3328ea2" style="zoom:50%;" />

#### 最后结果：

| 字符 | 赫夫曼编码 |
| ---- | ---------- |
| a    | 00         |
| b    | 1011       |
| c    | 01         |
| d    | 1010       |
| e    | 11         |
| f    | 100        |

```
所以最后答案 A

已知字符集{ a, b, c, d, e, f }，
若各字符出现的次数分别为{ 6, 3, 8, 2, 10, 4 }，
则对应字符集中各字符的哈夫曼编码可能是：(2分)

A. 00, 1011, 01, 1010, 11, 100 ✔

B. 00, 100, 110, 000, 0010, 01

C. 10, 1011, 11, 0011, 00, 010

D. 0011, 10, 11, 0010, 01, 000
```
`
版权声明：本文为CSDN博主「Young_IT」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。原文链接：https://blog.csdn.net/Young_IT/article/details/106730343
`


### 参考代码

#### 输入

第一行输入t，表示有t个测试实例
第二行先输入n，表示第1个实例有n个权值，接着输入n个权值，权值全是小于1万的正整数
依此类推

#### 输出

逐行输出每个权值对应的编码，格式如下：权值-编码
即每行先输出1个权值，再输出一个短划线，再输出对应编码，接着下一行输入下一个权值和编码。
以此类推

#### 样例输入

```
1
5 15 4 4 3 2
```

#### 样例输出

```
15-1
4-010
4-011
3-001
2-000
```

#### 实现代码

```c++
const int MaxW = 9999999;  // 假设结点权值不超过9999999
// 定义huffman树结点类
class HuffNode
{
public:
    int weight;     // 权值
    int parent;     // 父结点下标
    int leftchild;  // 左孩子下标
    int rightchild; // 右孩子下标
};
// 定义赫夫曼树类
class HuffMan
{
private:
    void MakeTree();    // 建树，私有函数，被公有函数调用
    void SelectMin(int pos, int* s1, int* s2);  // 从 1 到 pos 的位置找出权值最小的两个结点，把结点下标存在 s1 和 s2 中
public:
    int len;    // 结点数量
    int lnum;   // 叶子数量
    HuffNode* huffTree; // 赫夫曼树，用数组表示
    string* huffCode;   // 每个字符对应的赫夫曼编码
    void MakeTree(int n, int wt[]); // 公有函数，被主函数main调用
    void Coding();  // 公有函数，被主函数main调用
    void Destroy();
};
// 构建huffman树
void HuffMan::MakeTree(int n, int wt[])
{
    // 参数是叶子结点数量和叶子权值
    // 公有函数，对外接口
    int i;
    lnum = n;
    len = 2 * n - 1;
    huffTree = new HuffNode[2 * n];
    huffCode = new string[lnum + 1];    // 位置从 1 开始计算
    // huffCode实质是个二维字符数组，第 i 行表示第 i 个字符对应的编码
    // 赫夫曼树huffTree初始化
    for (i = 1; i <= n; i++)
        huffTree[i].weight = wt[i - 1]; // 第0号不用，从1开始编号
    for (i = 1; i <= len; i++)
    {
        if (i > n) huffTree[i].weight = 0;  // 前n个结点是叶子，已经设置
        huffTree[i].parent = 0;
        huffTree[i].leftchild = 0;
        huffTree[i].rightchild = 0;
    }
    MakeTree();  // 调用私有函数建树
}
void HuffMan::SelectMin(int pos, int* s1, int* s2)
{
    // 找出最小的两个权值的下标
    // 函数采用地址传递的方法，找出两个下标保存在 s1 和 s2 中
    int w1, w2, i;
    w1 = w2 = MaxW;  // 初始化w1和w2为最大值，在比较中会被实际的权值替换
    *s1 = *s2 = 0;
    for (i = 1; i <= pos; i++)
    {
        // 比较过程如下：
        // 如果第 i 个结点的权值小于 w1，且第 i 个结点是未选择的结点，提示：如果第 i 结点未选择，它父亲为 0
        // 把第 w1 和 s1 保存到 w2 和 s2，即原来的第一最小值变成第二最小值
        // 把第 i 结点的权值和下标保存到 w1 和 s1，作为第一最小值
        // 否则，如果第 i 结点的权值小于 w2，且第 i 结点是未选择的结点
        // 把第 i 结点的权值和下标保存到 w2 和 s2，作为第二最小值
        if (w1 > huffTree[i].weight && !huffTree[i].parent) {
            w2 = w1;
            *s2 = *s1;

            w1 = huffTree[i].weight;
            *s1 = i;
        }
        else if (w2 > huffTree[i].weight && !huffTree[i].parent) {
            w2 = huffTree[i].weight;
            *s2 = i;
        }
    }
}
void HuffMan::MakeTree()
{
    // 私有函数，被公有函数调用
    int i, s1, s2;
    for (i = lnum + 1; i <= len; i++)
    {
        SelectMin(i - 1, &s1, &s2);  // 找出两个最小权值的下标放入 s1 和 s2 中
        huffTree[s1].parent = huffTree[s2].parent = i;
        huffTree[i].leftchild = s1;
        huffTree[i].rightchild = s2;
        huffTree[i].weight = huffTree[s1].weight + huffTree[s2].weight;
        // 将找出的两棵权值最小的子树合并为一棵子树，过程包括
        // 结点 s1 和结点 s2 的父亲设为 i
        // 结点 i 的左右孩子分别设为 s1 和 s2
        // 结点 i 的权值等于 s1 和 s2 的权值和
    }
}
// 赫夫曼编码
void HuffMan::Coding()
{
    char* cd;
    int i, c, f, start;
    // 求 n 个结点的赫夫曼编码
    cd = new char[lnum];    // 分配求编码的工作空间
    cd[lnum - 1] = '\0';    // 编码结束符
    for (i = 1; i <= lnum; ++i)
    {
        // 逐个字符求赫夫曼编码
        start = lnum - 1;   // 编码结束符位置
        // 参考课本P147算法6.12 HuffmanCoding代码
        for (c = i, f = huffTree[i].parent; f != 0; c = f, f = huffTree[f].parent)
            if (huffTree[f].leftchild == c)
                cd[--start] = '0';
            else
                cd[--start] = '1';
        huffCode[i].assign(&cd[start]); // 把cd中从start到末尾的编码复制到huffCode中
    }
    delete[]cd;    // 释放工作空间
}
// 销毁赫夫曼树
void HuffMan::Destroy()
{
    len = 0;
    lnum = 0;
    delete[]huffTree;
    delete[]huffCode;
}
// 主函数
int main()
{
    int t, n, i, j;
    int wt[800];
    cin >> t;
    HuffMan myHuff;
    for (i = 0; i < t; i++)
    {
        cin >> n;
        for (j = 0; j < n; j++)
            cin >> wt[j];

        myHuff.MakeTree(n, wt);
        myHuff.Coding();
        for (j = 1; j <= n; j++)
        {
            cout << myHuff.huffTree[j].weight << '-';   // 输出各权值
            cout << myHuff.huffCode[j] << endl; // 输出各编码
        }
        myHuff.Destroy();
    }
    return 0;
}
```





### 解码参考代码

#### 输入

第一行输入t，表示有t个测试实例
第二行先输入n，表示第1个实例有n个权值，接着输入n个权值，权值全是小于1万的正整数
第三行输入n个字母，表示与权值对应的字符
第四行输入k，表示要输入k个编码串
第五行起输入k个编码串
以此类推输入下一个示例

#### 输出

每行输出解码后的字符串，如果解码失败直接输出字符串“error”，不要输出部分解码结果

#### 样例输入

```
2
5 15 4 4 3 2
A B C D E
3
11111
10100001001
00000101100
4 7 5 2 4
A B C D
3
1010000
111011
111110111
```

#### 样例输出

```
AAAAA
ABEAD
error
BBAAA
error
DCD
```

#### 实现代码

只需在上文赫夫曼树实现代码中加入 Coding 方法和 Decoding 方法的实现。

```c++
//解码方法
int HuffMan::Decode(const string codestr, char txtstr[]) {
    int i, k, c;
    char ch;
    c = len;
    k = 0;
    for (i = 0; i < codestr.length(); i++) {
        ch = codestr[i];
        if (ch == '0') {
            c = huffTree[c].leftchild;
        }
        else if (ch == '1') {
            c = huffTree[c].rightchild;
        }
        else {
            return -1;
        }
        if (huffTree[c].leftchild == 0 && huffTree[c].rightchild == 0) {
            txtstr[k++] = huffTree[c].cha;
            c = len;
        }
        else {
            ch = '\0';
        }
    }
    if (ch == '\0') return -1;
    else txtstr[k] = '\0';
    return 1;
}
//主函数
int main()
{
    int t, n, i, j, m;
    int wt[800];
    char ct[800];
    HuffMan myHuff;
    cin >> t;
    for (i = 0; i < t; i++) {
        cin >> n;
        string codestr;
        char txtstr[800];
        for (j = 0; j < n; j++) {
            cin >> wt[j];
        }
        for (j = 0; j < n; j++) {
            cin >> ct[j];
        }
        myHuff.MakeTree(n, wt, ct);
        myHuff.Coding();
        cin >> m;
        while (m--) {
            cin >> codestr;
            if (myHuff.Decode(codestr, txtstr) == 1) {
                cout << txtstr << endl;
            }
            else {
                cout << "error" << endl;
            }
        }
        myHuff.Destroy();
    }
    return 0;
}
```



## 一道算法题



### 赫夫曼编码长度

#### 题目描述

每行一个大小写英文字母组成的字符串，长度不大于 1000，通过前缀编码后最短的编码长度。

#### 输入

第一行输入一个整数t，表示有t组测试数据；

接下来输入t组测试数据，每组数据一行，大小写英文字母。

#### 输出

每组数据输出赫夫曼编码长度

#### 样例输入

```
4
AABBCCDEEEE
AAABCCC
BBACB
tPvlQHFbPN
```

#### 样例输出

```
25
11
7
32
```

#### 题目解析
`
本题如果真要用哈夫曼来建树，计算值会非常复杂。

首先要能够发现规律：哈夫曼树的编码长度等于各个叶节点权值与路径长度乘积之和，同时这个值等于非叶节点之和。

采用优先队列模拟哈夫曼树的建立。采用map记录字符与出现的次数，将每个字符的次数依次加入优先队列（数值小的在队头），每一次从队列中出队最小的两个，相加后再加入队列中。用ans记录每一次相加和temp值之和，当队列中剩下一个元素时，ans的值即为所求

知识点：
`priority_queue` 优先队列：

`priority_queue<int>q;` 默认为数字（字典序）大的值在队首top，等价于`priority_queue<int, vector<int>, less<int> >q;`

`priority_queue<int, vector<int>, greater<int> >q;` 表示数字（字典序）小的在队首

`
版权声明：本文为CSDN博主「julia7_」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_35093872/article/details/88055475
`
#### AC代码

```c++
#include <iostream>
#include <string>
#include <cstring>
#include <queue>
#include <map>
using namespace std;
const int maxn = 100;

int main() {
    int t;
    cin >> t;
    while (t--) {
        string str;
        map<char, int> mp;
        priority_queue<int, vector<int>, greater<int> >q;
        cin >> str;
        int len = str.length();
        for (int i = 0; i < len; i++) {
            if (mp.find(str[i]) == mp.end()) {
                mp[str[i]] = 1;
            }
            else {
                mp[str[i]]++;
            }
        }
        for (map<char, int>::iterator it = mp.begin(); it != mp.end(); it++) {
            q.push(it->second);
        }
        int ans = 0;
        while (q.size() != 1) {
            int a, b, temp;
            a = q.top();
            q.pop();
            b = q.top();
            q.pop();
            temp = a + b;
            ans += temp;
            q.push(temp);
        }
        cout << ans << endl;
    }
    return 0;
}
```

