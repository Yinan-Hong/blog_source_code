+++
title = "AVL树和Trie树"
date = "2021-06-26"
categories = [
    "数据结构","算法"
]
tags = [
    "AVL树",
    "平衡二叉树",
    "数据结构",
	"Trie"
]
toc = true

+++

本文介绍AVL树和Trie树。Trie树是因为正好做到所以就顺便记录一下。

文章里插入了一些gif，虽然我已经压缩到了3、4m的样子，但加载起来还是卡卡的，同学们可以等加载好之后刷新一下再看就流畅了。

<!--more-->

## AVL树

> Wiki: 在计算机科学中，AVL树是最早被发明的自平衡二叉查找树。在AVL树中，任一节点对应的两棵子树的最大高度差为1，因此它也被称为高度平衡树。查找、插入和删除在平均和最坏情况下的时间复杂度都是O(logn)。
>
> AVL树由前苏联的数学家 **A**delse-**V**elskil 和 **L**andis 在1962 年提出。

AVL树就是在插入和删除的操作时，动态调整树的结构，使AVL树的任意两颗子树的高度差<=1。平衡二叉树查询时，每进行一次判断能排除一半的节点，所以复杂度是O(log2n)。如果在插入和删除中不进行动态调整，那树结构有可能会退化成链表，或者会存在很长的直链，导致查询的复杂度最坏会到O(n)。

<img src="../AVL&TriePic/Snipaste_2021-06-24_10-47-13.png" alt="Snipaste_2021-06-24_10-47-13" style="zoom:60%;" />

### 平衡因子

**平衡因子（Balance Factor）**是节点的左右子树高度差，二叉平衡树中，平衡因子只能取0, 1, -1。有些书上规定平衡因子是左高减右高，有些则是反过来，其实影响不大，只需在对应情况下进行正确的旋转操作就行。本文中平衡因子是取左子树高度减右子树高度。

### 节点结构

```c++
class BiNode {
public:
	int key; // 关键值
	int bf; // 平衡因子
	BiNode* lChild, * rChild;
	BiNode(int kValue, int bValue) { /*略*/ }
	~BiNode() { /*略*/ }
};
```

### 常见操作

```c++
// 二叉排序树
class BST {
private:
	BiNode* root; // 根结点指针
	void rRotate(BiNode*& p); //p为根的右旋操作
	void lRotate(BiNode*& p); //p为根的左旋操作
	void leftBalance(BiNode*& t); //左平衡处理
	void rightBalance(BiNode*& t); //右平衡处理
	int insertAVL(BiNode*& t, int key, bool& taller); // 插入元素并做平衡处理
	void del(BiNode*& t, key); //删除操作，考完试补 todo
public:
	BST();
	void insertAVL(int key); // 二叉排序树插入元素
    void del(int key); // 删除节点
	~BST();
	void inOrder(); // 中序遍历
};
```

### 节点的插入和与平衡

![Snipaste_2021-06-26_16-52-49](../AVL&TriePic/Snipaste_2021-06-26_16-52-49.png)

上图左侧是平衡二叉树，节点上方数字为高度，当插入key为99的节点时，key为66的节点的平衡因子变为-2，发生了失衡。这时需要对二叉树进行调整，来保证所有节点满足平衡因子的绝对值<=1。

### 最小失衡树

在新插入的结点向上查找，以第一个平衡因子的绝对值超过 1 的结点为根的子树称为最小不平衡子树。也就是说，一棵失衡的树，是有可能有多棵子树同时失衡的。而这个时候，我们**只要调整最小的不平衡子树**，就能够将不平衡的树调整为平衡的树。

**平衡二叉树的失衡调整主要是通过旋转最小失衡子树来实现的**。根据旋转的方向有两种处理方式，**左旋** 与 **右旋** 。

旋转的目的就是减少高度，通过降低整棵树的高度来平衡。**哪边的树高，就把那边的树向上旋转**。

### 左旋

1. <font style="background:LightPink">失衡节点</font>的<font style="background:BlanchedAlmond">右孩子</font>变为根节点
2. <font style="background:BlanchedAlmond">右孩子</font>的**左子树**变为<font style="background:LightPink">失衡节点</font>的**右子树**
3. <font style="background:LightPink">失衡节点</font>变为<font style="background:BlanchedAlmond">右孩子</font>的**左子树**

<img src="../AVL&TriePic/left.gif" alt="left" style="zoom:50%;" />

```c++
void BST::lRotate(BiNode*& p) {
	BiNode* rc = p->rChild;
	p->rChild = rc->lChild;
	rc->lChild = p;
	p = rc;
}
```



### 右旋

1. <font style="background:LightPink">失衡节点</font>的<font style="background:BlanchedAlmond">左孩子</font>变为根节点
2. <font style="background:BlanchedAlmond">左孩子</font>的**右子树**变为<font style="background:LightPink">失衡节点</font>的**左子树**
3. <font style="background:LightPink">失衡节点</font>变为<font style="background:BlanchedAlmond">左孩子</font>的**右子树**

<img src="../AVL&TriePic/right.gif" alt="right" style="zoom:50%;" />

```c++
void BST::rRotate(BiNode*& p) {
	BiNode* lc = p->lChild;
	p->lChild = lc->rChild;
	lc->rChild = p;
	p = lc;
}
```

### 先左旋后右旋

图先放，下面紧接着就解释为什么要有二次旋转的操作。

1. 对失衡节点的左节点进行左旋
2. 对失衡节点进行右旋

<img src="../AVL&TriePic/leftright.gif" alt="leftright" style="zoom:40%;" />

### 先右旋后左旋

1. 对失衡节点的右节点进行右旋
2. 对失衡节点进行左旋

<img src="../AVL&TriePic/rightleft.gif" alt="rightleft" style="zoom:50%;" />



### 四种插入对应的旋转

<img src="../AVL&TriePic/Snipaste_2021-06-26_17-56-45.png" alt="Snipaste_2021-06-26_17-56-45" style="zoom: 33%;" />

这棵树插入节点55后，失衡的节点是60，而且是左侧高。如果对66进行左旋处理如下图，发现**树依旧失衡**。

<img src="../AVL&TriePic/wrong.gif" alt="wrong" style="zoom: 20%;" />

所以我们需要判断，**失衡节点是左高还是右高，失衡节点的子节点是左高还是右高**，来区分不同的情况。像上图的，失衡节点是**左侧高**，失衡节点的左子树为**右侧高**，此时需要进行两次旋转。

实际上有四种插入，分别对应四种旋转

1. 插入在失衡节点的**左子树的左子树**，进行**左旋**
2. 插入在失衡节点的**右子树的右子树**，进行**右旋**
3. 插入在失衡节点的**左子树的右子树**，进行**先左旋后右旋**
4. 插入在失衡节点的**右子树的左子树**，进行**先右旋后左旋**

在代码实现时，先判断失衡节点需要进行左旋还是右旋，然后在平衡处理的函数中，判断子节点是否需要进行旋转。

```c++
// t为根的二叉排序树作左平衡旋转处理
void BST::leftBalance(BiNode*& t) {
	BiNode *l = t->lChild;
	switch (l->bf) {
	case LH: t->bf = l->bf = EH; rRotate(t); break;	// 左子节点为左高，对应第一种插入
	case RH:	// 左子节点为右高，对应第三种插入
		BiNode *lr = l->rChild;
		switch (lr->bf) {	// 调整平衡因子
		case LH: t->bf = RH; l->bf = EH; break;
		case EH: t->bf = l->bf = EH; break;
		case RH: t->bf = EH; l->bf = LH; break;
		}
		lr->bf = EH;
		lRotate(t->lChild);	// 进行先左旋后右旋
		rRotate(t);
	}
}

// t为根的二叉排序树作右平衡旋转处理
void BST::rightBalance(BiNode*& t) {
	BiNode* r = t->rChild;
	switch (r->bf) {
	case RH: t->bf = r->bf = EH; lRotate(t); break;	// 右子节点为右高，对应第二种插入
	case LH:	// 右子节点为左高，对应第四种插入
		BiNode* rl = r->lChild;
		switch (rl->bf) { // 调整平衡因子
		case RH: t->bf = LH; r->bf = EH; break;
		case EH: t->bf = r->bf = EH; break;
		case LH: t->bf = EH; r->bf = RH; break;
		}
		rl->bf = EH;
		rRotate(t->rChild);	// 进行先右旋后左旋
		lRotate(t);
	}
}
```

### 删除操作

todo...

### 完整样例代码

输入

```
第一行输入测试数据组数t；
每组测试数据，第一行输入结点数n, 第二行输入n个结点值。
```

输出

```
对每组测试数据，按中序遍历的顺序输出树中，结点值及平衡因子（测试数据没有空树）
即输出"结点值:平衡因子"，不同结点之间间隔一个空格。
```

样例输入

```
8
3
64 5 1
3
64 5 13
6
64 78 5 1 13 15
6
64 78 5 1 13 10
3
64 78 100
3
64 80 70
6
64 30 80 90 70 68
6
64 30 80 90 70 75
```

样例输出

```
1:0 5:0 64:0
5:0 13:0 64:0
1:0 5:1 13:0 15:0 64:0 78:0
1:0 5:0 10:0 13:0 64:-1 78:0
64:0 78:0 100:0
64:0 70:0 80:0
30:0 64:0 68:0 70:0 80:-1 90:0
30:0 64:1 70:0 75:0 80:0 90:0
```

AC代码

```c++
#include <iostream>
using namespace std;

#define LH 1 // 左高
#define EH 0 // 等高
#define RH -1 // 右高

class BiNode {
public:
	int key; // 关键值
	int bf; // 平衡因子
	BiNode* lChild, * rChild;
	BiNode(int kValue, int bValue) {
		key = kValue;
		bf = bValue;
		lChild = NULL;
		rChild = NULL;
	}
	~BiNode() {
		key = 0;
		bf = 0;
		lChild = NULL;
		rChild = NULL;
	}
};

// 二叉排序树
class BST {
private:
	BiNode* root; // 根结点指针
	void rRotate(BiNode*& p);
	void lRotate(BiNode*& p);
	void leftBalance(BiNode*& t);
	void rightBalance(BiNode*& t);
	int insertAVL(BiNode*& t, int key, bool& taller); // 插入元素并做平衡处理
	void inOrder(BiNode* p);
public:
	BST();
	void insertAVL(int key); // 二叉排序树插入元素
	~BST();
	void inOrder(); // 中序遍历
};

// 以p为根的二叉排序树作右旋处理
void BST::rRotate(BiNode*& p) {
	BiNode* lc = p->lChild;
	p->lChild = lc->rChild;
	lc->rChild = p;
	p = lc;
}

// 以p为根的二叉排序树作左旋处理
void BST::lRotate(BiNode*& p) {
	BiNode* rc = p->rChild;
	p->rChild = rc->lChild;
	rc->lChild = p;
	p = rc;
}

// t为根的二叉排序树作左平衡旋转处理
void BST::leftBalance(BiNode*& t) {
	BiNode *l = t->lChild;
	switch (l->bf) {
	case LH: t->bf = l->bf = EH; rRotate(t); break;
	case RH:
		BiNode *lr = l->rChild;
		switch (lr->bf) {
		case LH: t->bf = RH; l->bf = EH; break;
		case EH: t->bf = l->bf = EH; break;
		case RH: t->bf = EH; l->bf = LH; break;
		}
		lr->bf = EH;
		lRotate(t->lChild);
		rRotate(t);
	}
}

// t为根的二叉排序树作右平衡旋转处理
void BST::rightBalance(BiNode*& t) {
	BiNode* r = t->rChild;
	switch (r->bf) {
	case RH: t->bf = r->bf = EH; lRotate(t); break;
	case LH:
		BiNode* rl = r->lChild;
		switch (rl->bf) {
		case RH: t->bf = LH; r->bf = EH; break;
		case EH: t->bf = r->bf = EH; break;
		case LH: t->bf = EH; r->bf = RH; break;
		}
		rl->bf = EH;
		rRotate(t->rChild);
		lRotate(t);
	}
}


int BST::insertAVL(BiNode*& t, int key, bool& taller) {
	if (!t) {
		t = new BiNode(key, 0);
		taller = true;
	}
	else if (key == t->key) { //存在相同key节点，不再插入
		taller = false;
		return 0;
	}
	else if (key < t->key) { //搜寻左子树
		if (!insertAVL(t->lChild, key, taller)) //未插入
			return 0;
		if (taller)	//插入成功则树“长高”
			switch (t->bf) {
			case LH: leftBalance(t); taller = false; break;	//原本左侧高，左平衡处理
			case EH: t->bf = LH; taller = true; break; //原本等高，现在左侧高
			case RH: t->bf = EH; taller = false; break; //原本右侧高，现在等高
			}
	}
	else {	//搜寻右子树
		if (!insertAVL(t->rChild, key, taller))
			return 0;
		if(taller)
			switch (t->bf) {
			case LH: t->bf = EH; taller = false; break;
			case EH: t->bf = RH; taller = true; break;
			case RH: rightBalance(t); taller = false; break;
			}
	}
	return 1;
}

void BST::inOrder(BiNode* p) {
	if (p) {
		inOrder(p->lChild);
		cout << p->key << ':' << p->bf << ' ';
		inOrder(p->rChild);
	}

	return;
}

// 二叉排序树初始化
BST::BST() {
	root = NULL;
}

BST::~BST() {
	root = NULL;
}

// 插入元素并作平衡处理
void BST::insertAVL(int key) {
	bool taller = false;
	insertAVL(root, key, taller);
}


// 中序遍历
void BST::inOrder() {
	inOrder(root);
}

int main(void) {
	int t;
	cin >> t;
	while (t--) {
		// 构建二叉平衡树，并在插入元素时做平衡处理
		int n, elem;
		cin >> n;
		BST tree;
		while (n--) {
			cin >> elem;
			tree.insertAVL(elem);
		}
		tree.inOrder();
		cout << endl;
	}
	return 0;
}
```
#### 参考资料

https://zhuanlan.zhihu.com/p/56066942

https://www.jianshu.com/p/65c90aa1236d

## Trie树

Trie树，字典树，前缀树。是一种哈希树的变种。典型应用是用于统计，排序和保存大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：利用字符串的公共前缀来节约存储空间，最大限度地减少无谓的字符串比较，查询效率比哈希表高。

<img src="../AVL&TriePic/Snipaste_2021-06-21_10-31-59.png" alt="Snipaste_2021-06-21_10-31-59" style="zoom:50%;" />

因为题目没有要求，这里只给出建树和搜索的实现。有兴趣的朋友可以自己是现实一下大小写区分、输入查重、搜索单词等Trie树常见操作。代码注释非常详细，这里就不多做解释了。

### 完整样例代码

题目描述

```
输入的一组单词，创建Trie树。输入字符串，计算以该字符串为公共前缀的单词数。
```

输入

```
测试数据有多组
每组测试数据格式为：
第一行：一行单词，单词全小写字母，且单词不会重复，单词的长度不超过10
第二行：测试公共前缀字符串数量t
后跟t行，每行一个字符串
```

输出

```
每组测试数据输出格式为：
第一行：创建的Trie树的层次遍历结果
第2~t+1行：对每行字符串，输出树中以该字符串为公共前缀的单词数。
```

样例输入

```
5 abd abcd bcd efg hig
3
ab
bc
abcde
```

样例输出

```
abehbcficddggd
2
1
0
```

AC代码

```c++
#include <iostream>
#include <queue>
using namespace std;

class Node {
public:
	char data;
	Node** next;
	bool is_leaf;
	Node(char data) {
		this->data = data;
		next = new Node * [26];
		for (int i = 0; i < 26; i++)
			next[i] = nullptr;
		is_leaf = true;
	}
	~Node() {
		delete next;
	}
};

class Trie {
private:
	Node* root;
	void insert(Node* node, char* str);
	void search(Node* node, char* str, int& cnt);
	void search(Node* node, int& cnt);
	void destroy(Node* node);
public:
	Trie();
	void insert(char* str);
	int search(char* str);
	void level_order();
	~Trie();
};

Trie::Trie() {
	root = new Node('\0');
}

Trie::~Trie() {
	destroy(root);
	delete root;
}

//深度优先，删除Trie
void Trie::destroy(Node* node) {
	for (int i = 0; i < 26; i++) {
		if (node->next[i]) {
			if (!node->next[i]->is_leaf)
				destroy(node->next[i]);	//从叶子结点开始删
			delete node->next[i];
		}
	}
	node->is_leaf = true;
}

void Trie::insert(char* str) {
	insert(root, str);
}

void Trie::insert(Node* node, char* str) {
	if (*str == '\0')	//字符串结束
		return;
	int pos = *str - 'a';	//取子节点位置
	if (!node->next[pos]) {
		node->next[pos] = new Node(*str);	//新建子树
		node->is_leaf = false;
	}
	insert(node->next[pos], str + 1);	//插入字符串下一位
}

int Trie::search(char* str) {
	int cnt = 0;	//记录公共前缀单词数
	search(root, str, cnt);
	return cnt;
}

//匹配前缀
void Trie::search(Node* node, char* str, int& cnt) {
	if (*str == '\0') {	//前缀匹配成功，搜索以当前节点为根的叶子数
		search(node, cnt);
		return;
	}
	int pos = *str - 'a';	//取当前字符位置
	if (!node->next[pos])	//前缀匹配失败
		return;
	search(node->next[pos], str + 1, cnt);
}

//搜索公共前缀单词数，即叶子数，深度优先
void Trie::search(Node* node, int& cnt) {
	if (node->is_leaf) {	//找到叶子，返回
		cnt++;
		return;
	}
	for (int i = 0; i < 26; i++)	//子节点存在则向下搜索
		if (node->next[i])
			search(node->next[i], cnt);
}

void Trie::level_order() {
	if (!root)
		return;
	queue<Node*>q;
	q.push(root);
	while (!q.empty()) {
		Node* n;
		n = q.front();
		q.pop();
		for (int i = 0; i < 26; i++)	//将当前节点的所有子节点存入队列
			if (n->next[i]) {
				cout << n->next[i]->data;
				q.push(n->next[i]);
			}
	}
	cout << endl;
}

int main() {
	int	t;
	const int MAX_LETTER = 11;
	while (cin >> t) {
		Trie trie;
		char* word = new char[MAX_LETTER];
		for (int i = 0; i < t; i++) {
			cin >> word;
			trie.insert(word);
		}
		delete[] word;
		trie.level_order();

		char* prefix;
		cin >> t;
		for (int i = 0; i < t; i++) {
			prefix = new char[MAX_LETTER];
			cin >> prefix;
			int cnt = trie.search(prefix);
			cout << cnt << endl;
			delete[] prefix;
		}
	}
}
```

