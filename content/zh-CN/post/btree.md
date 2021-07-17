+++
title = "手撸一棵B树"
date = "2021-06-07T15:28:15+08:00"
categories = [
    "数据结构"
]
tags = [
    "数据结构",
    "B树"
]
toc = true

+++


本文教大家如何手撸一棵B树。

我在网上看到的大多数教程都是上来就丢给你一堆B树的性质特点，简直劝退，本文我会尝试用人能理解的思路介绍一下B树，有代码干货。

<!--more-->


>百度百科：
>
>在计算机科学中，B树（英语：B-tree）是一种**自平衡**的树，能够保持数据有序。这种数据结构能够让查找数据、顺序访问、插入数据及删除的动作，都在**对数时间**内完成。与自平衡二叉查找树不同，B树为系统大块数据的读写操作做了优化。B树减少定位记录时所经历的中间过程，从而加快存取速度。B树这种数据结构可以用来描述外部存储。这种数据结构常被应用在**数据库和文件系统**的实现上。


顺便，B树的英文是B-tree（B杠tree），有些人以为B树和B-树（B减树）是两回事，其实都是误会,并没有B减树。

&emsp;&ensp;
&emsp;&ensp;
## B树的构成和结构

![Snipaste_2021-06-07_15-42-05](../btreePic/Snipaste_2021-06-07_15-42-05.png)
&emsp;&ensp;
&emsp;&ensp;
### B树的节点构成

上图展示的就是一棵b树。我们先来看看b树**节点**的组成。

b树的节点包含以下内容：

1. 本节点存放的<font style="background:Lavender">数据（关键字）数组</font>（图中蓝底部分），编号从1开始
2. 指向子节点的指针数组（图中白底部分）
3. 指向父节点的指针（图中未标出）

&emsp;&ensp;

下图红框框 框出来的就是**一个节点**。构建b树前需要先给出b树的**阶数m**，一个节点中最多有**m-1个数据**和**m个子树**。为了方便代码实现，我们规定**关键字的编号从1开始，子节点的编号从0开始**。

![Snipaste_2021-06-07_15-46-54](../btreePic/Snipaste_2021-06-07_15-46-54.png)

```c++
class BTNode {
public:
	int keyNum; // 关键字个数
	BTNode* parent; // 指向父节点
	int* keys; // 关键字数组，编号从1开始
	BTNode** ptr; // 子树指针数组
	BTNode();
	~BTNode();
};
```

&emsp;&ensp;

### B树的数据存储结构

![Snipaste_2021-06-07_15-56-12](../btreePic/Snipaste_2021-06-07_15-56-12.png)

我们看存放**32**这个数据的节点。在**节点内**，数据从左到右是**由小到大排列**。

在**32**这个数据**左侧的子树**，其包含的所有数据都**小于32**，对于**右侧的子树**，其包含的数据都**大于32**。实际上每个节点都满足，**节点内关键字由小到大排列，当前关键字左侧子树的关键字都小于当前关键字，右侧的则大于**。

&emsp;&ensp;

## B树的操作

B树常见的操作有，搜索，插入，分裂，删除/* todo */。

```c++
class BTree {
private:
	BTNode* root;
     // 将k和ap分别插入t->key[i+1]和t->ptr[i + 1]
	void insert(BTNode* t, int i, int k, BTNode* ap);
     // 在p->key[1...keynum]中查找 k
	int search(BTNode* t, int k);
     // 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
	void split(BTNode* q, int s, BTNode*& ap);
     // 生成含信息(t, x, ap)的新根结点t*ap, 原t和 ap为子树指针
	void newRoot(BTNode*& t, BTNode* q, int x, BTNode* ap);
     // 在结点t的子结点q的key[i]与key[i+1]之间插入k
	void insertBT(BTNode*& t, int k, BTNode* q, int i);
	Result searchBT(BTNode* t, int k); // 在结点t中查找k
public:
	BTree();
	~BTree();
	void insertBT(int key); // B-树插入操作
	void searchBT(int key); // B-树查找操作
	void levelOrder(); // B-树层次遍历
};
```

&emsp;&ensp;

### 手撸一棵B树

讲具体的操作之前我们先**手动实现**一棵B树，这样方便大家理解B树的数据分布。B树的构建其实就是使用插入操作不断插入数据。给出一组数据**21, 45, 65, 25, 31, 52, 55**，构建**3阶**b树。

前面提到过，m阶b树的节点最多包含**m-1个数据和m个子树**，所以前三个数据只需按顺序插入到第一个节点。

<img src="../btreePic/Snipaste_2021-06-07_16-25-00.png" alt="Snipaste_2021-06-07_16-25-00" style="zoom: 50%;" />

插入第三个数据后，节点的数据超过了m-1=2个，这时候就需要对这个节点进行**分裂操作**。分裂的位置我们取**m/2然后向上取整**（可以有不同的取法），在这里就是第2个数据**45**。

分裂操作：

1. 新建节点，将<font style="background:LightPink">分裂位置</font>右侧的全部数据移入新节点
2. 分裂位置右侧的<font style="background:BlanchedAlmond">子树指针</font>指向这个新结点
3. 将<font style="background:LightPink">分裂位置</font>的数据和<font style="background:BlanchedAlmond">子树指针</font>插入父节点，如果没有父节点则新建根节点

<img src="../btreePic/Snipaste_2021-06-07_16-48-26.png" alt="Snipaste_2021-06-07_16-48-26" style="zoom: 50%;" />

&emsp;&ensp;

继续插入下一个数据**25**，从根节点进行**搜索操作**：

**45 > 21**，进入左子树搜索，**21 < 25**，插入在右侧。同样地插入数据**31**。

<img src="../btreePic/Snipaste_2021-06-07_16-52-58.png" alt="Snipaste_2021-06-07_16-52-58" style="zoom:50%;" />

&emsp;&ensp;

发现节点中数据超过2个，进行分裂，同样的分裂操作。

1. 新建节点，将<font style="background:LightPink">分裂位置</font>右侧的全部数据移入新节点
2. 分裂位置右侧的<font style="background:BlanchedAlmond">子树指针</font>指向这个新结点
3. 将<font style="background:LightPink">分裂位置</font>的数据和<font style="background:BlanchedAlmond">子树指针</font>插入父节点，如果没有父节点则新建根节点

<img src="../btreePic/Snipaste_2021-06-07_16-57-50.png" alt="Snipaste_2021-06-07_16-57-50" style="zoom:50%;" />

同样的道理插入最后两个数据，最后得到的数据结构是下图这样的（省略最后一层的空指针）。

<img src="../btreePic/Snipaste_2021-06-07_17-05-32.png" alt="Snipaste_2021-06-07_17-05-32" style="zoom:50%;" />

&emsp;&ensp;

各位可以尝试手撸一下这个样例巩固一下。4阶b树，按顺序插入45, 24, 53, 90, 46, 47。

结果是这样的。

<img src="../btreePic/Snipaste_2021-06-07_17-15-58.png" alt="Snipaste_2021-06-07_17-15-58" style="zoom:50%;" />

&emsp;&ensp;

### 搜索操作

传入根节点t和需要查询的关键字k，返回类Result。

tag为0或1表示是否查询成功。如果查询成功，pt为k所在的节点，i为k在pt中的位置。如果查询失败，pt为k需要插入的节点，i + 1为k插入的位置。

```c++
class Result {
public:
	BTNode* pt; // 指向找到的结点
	int i;
	int tag;
	Result(BTNode* p, int m, int t) {
		pt = p;		i = m;		tag = t;
	}
	Result(const Result& R) {
		pt = R.pt;	i = R.i;	tag = R.tag;
	}
	~Result() {
		pt = nullptr;	i = 0; 	tag = 0;
	}
};
```

从根结点开始搜索关键字k `(line 17)`

1. 如果节点中存在>=k的关键字，则返回该关键字的前一个位置i - 1`(line 5)`
   - 如果这个关键字==k `(line 18)`，则查询成功，返回节点信息`(line 26)`
   - 如果这个关键字>k，查询失败，进入关键字左侧的子树搜寻`(line 20)`。如果子树不存在，则返回k的插入位置。
2. 如果节点中不存在>=k的关键字，说明k比节点中所有关键字都大，所以返回最后一个子树指针的位置`(line 8)`，进入最右侧子树搜索`(line 20)`

```c++
// 在p->key[1...keynum]中查找 k
int BTree::search(BTNode* t, int k) {
	for (int i = 1; i <= t->keyNum; i++)
		if (t->keys[i] >= k)
			return i - 1;
    //找到关键字>=k，如果=则找到，如果>则在左边的子树找，或在i处插入

	return t->keyNum;//所有关键字都<k，去最右侧边的子树找，或在keyNum处插入
}

// 遍历所有节点查询key关键字
Result BTree::searchBT(BTNode* t, int k) {
	BTNode* p = t, * q = nullptr; // p指向当前查询节点，q指向当前节点的父节点
	bool found = false;
	int i = 0;
	while (p && !found) {
		i = search(p, k);	// 在当前节点中查找k
		if (p->keys[i + 1] == k)
			found = true;
		else {
			q = p;
			p = p->ptr[i];	// 转到下一个节点
		}
	}
	if (found)
		return Result(p, i + 1, 1);	// 查找成功，返回节点信息
	else
		return Result(q, i, 0);	// 查找失败，返回插入位置信息
}
```

&emsp;&ensp;

### 插入和分裂操作

传入key，在树中搜索key，如果返回的rusult.tag == 1，则关键字已存在，如果result.tag == 0，则执行插入操作。

```c++
// B-树插入操作
void BTree::insertBT(int key) {
	Result r = searchBT(root, key);
	if (!r.tag) {
		insertBT(root, key, r.pt, r.i);
	}
}
```

将key插入节点的对应位置

1. 插入后节点关键字数量 < m - 1 ，插入成功，退出`(line 14)`
2. 插入后节点关键字数量 > m - 1 ，执行分裂操作`(line 16)`
   - 当前节点存在父节点，执行分裂操作③的插入父节点`(line 21)`
   - 当前节点不存在父节点，执行分裂操作③的创建新根节点`(line 23)`

```c++
void BTree::insertBT(BTNode*& t, int k, BTNode* q, int i) {
	int x = k;
	BTNode* ap = nullptr;
	bool finished = false;
	if (!q) {	// 创建第一个节点
		q = new BTNode;
		q->keyNum = 1;
		q->keys[1] = k;
		this->root = q;
		return;
	}
	while (!finished) {
		insert(q, i, x, ap);	// 将k和ap分别插入t->key[i+1]和t->ptr[i + 1]
		if (q->keyNum < m)	// 插入完成（keys从1开始，所以判断keyNum < m）
			finished = true;
		else {	// 分裂节点*q
			int s = (m + 1) / 2;	// 向上取整
			split(q, s, ap);	// 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
			x = q->keys[s];
			q = q->parent;
			if (q)	// 将ap插入父节点对应位置
				i = search(q, x);	//查找ap在父节点的插入位置
			else {
				newRoot(t, q, x, ap);	//创建新的根节点
				finished = true;
			}
		}
	}
}

void BTree::split(BTNode* q, int s, BTNode*& ap) {
	ap = new BTNode;
	int j = 1;
	// 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
	for (int i = s + 1; i <= q->keyNum; i++) {
		ap->keys[j] = q->keys[i];
		ap->ptr[j] = q->ptr[i];
		j++;
	}
	ap->ptr[0] = q->ptr[s];
	ap->keyNum = j - 1;
	q->keyNum -= j;
	ap->parent = q->parent;
	for (int i = 0; i <= ap->keyNum; i++)
		if (ap->ptr[i])
			ap->ptr[i]->parent = ap;
}

// 生成含信息(t, x, ap)的新根结点t, 原t和 ap为子树指针
void BTree::newRoot(BTNode*& t, BTNode* q, int x, BTNode* ap) {
	q = new BTNode;
	q->keyNum = 1;
	q->keys[1] = x;
	q->ptr[0] = t;
	q->ptr[1] = ap;
	t->parent = q;
	ap->parent = q;
	this->root = q;
}
```

&emsp;&ensp;

### 删除操作

todo

&emsp;&ensp;

## 完整代码和测试样例

#### 输入

```
第一行输入t，表示有t个数据序列
第二行输入m, 表示要构建m阶B-树
第三行输入n，表示首个序列包含n个数据
第四行输入n个数据，都是自然数且互不相同，数据之间用空格隔开
```

#### 输出

```
输出B-树的关键字
同一个结点的关键字之间用:间隔
不同结点之间的关键字用一个空格间隔
对B-树进行层次遍历可以得到。
```

#### 样例输入

```
3
3
3
21 45 65
3
7
21 45 65 25 31 52 55
4
6
45 24 53 90 46 47
```

#### 样例输出

```
45 21 65
45 25 55 21 31 52 65
45:47 24 46 53:90
```

#### 代码示例

```c++
#include <iostream>
#include <queue>
using namespace std;

int m;

class BTNode {
public:
	int keyNum; // 关键字个数
	BTNode* parent; // 指向双亲结点
	int* keys; // 关键字向量
	BTNode** ptr; // 子树指针向量
	BTNode() {
		keyNum = 0;
		parent = nullptr;
		keys = new int[m + 2];
		ptr = new BTNode * [m + 2];
		for (int i = 0; i < m + 2; i++)
			ptr[i] = nullptr;
	}

	~BTNode() {
		delete[] keys;
		delete[] ptr;
	}
};

class Result {
public:
	BTNode* pt; // 指向找到的结点
	int i;
	int tag;
	Result(BTNode* p, int m, int t) {
		pt = p;
		i = m;
		tag = t;
	}

	Result(const Result& R) {
		pt = R.pt;
		i = R.i;
		tag = R.tag;
	}

	~Result() {
		pt = nullptr;
		i = 0;
		tag = 0;
	}
};

class BTree {
private:
	BTNode* root;
	void insert(BTNode* t, int i, int k, BTNode* ap); // 将k和ap分别插入t->key[i+1]和t->ptr[i + 1]
	int search(BTNode* t, int k); // 在p->key[1...keynum]中查找 k
	void split(BTNode* q, int s, BTNode*& ap); // 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
	void newRoot(BTNode*& t, BTNode* q, int x, BTNode* ap); // 生成含信息(t, x, ap)的新根结点t*ap, 原t和 ap为子树指针
	void insertBT(BTNode*& t, int k, BTNode* q, int i); // 在结点t的子结点q的key[i]与key[i+1]之间插入k
	Result searchBT(BTNode* t, int k); // 在结点t中查找k
public:
	BTree();
	~BTree();
	void insertBT(int key); // B-树插入操作
	void searchBT(int key); // B-树查找操作
	void levelOrder(); // B-树层次遍历
};

// 将k和ap分别插入t->key[i+1]和t->ptr[i + 1]
void BTree::insert(BTNode* t, int i, int k, BTNode* ap) {
	t->keyNum++;
	for (int j = t->keyNum; j > i + 1; j--) {	//把[i+1, keyNum]的数据后移一位
		t->keys[j] = t->keys[j - 1];
		t->ptr[j] = t->ptr[j - 1];
	}
	t->keys[i + 1] = k;	//插入数据
	t->ptr[i + 1] = ap;
}

// 在p->key[1...keynum]中查找 k
int BTree::search(BTNode* t, int k) {
	for (int i = 1; i <= t->keyNum; i++)
		if (t->keys[i] >= k)
			return i - 1;	//找到关键字>=k，如果=则找到，如果>则在左边的子树找，或在i处插入
	return t->keyNum;	//所有关键字都<k，去最右侧边的子树找，或在keyNum处插入
}

// 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
void BTree::split(BTNode* q, int s, BTNode*& ap) {
	ap = new BTNode;
	int j = 1;
	// 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
	for (int i = s + 1; i <= q->keyNum; i++) {
		ap->keys[j] = q->keys[i];
		ap->ptr[j] = q->ptr[i];
		j++;
	}
	ap->ptr[0] = q->ptr[s];
	ap->keyNum = j - 1;
	q->keyNum -= j;
	ap->parent = q->parent;
	for (int i = 0; i <= ap->keyNum; i++)
		if (ap->ptr[i])
			ap->ptr[i]->parent = ap;
}

// 生成含信息(t, x, ap)的新根结点t, 原t和 ap为子树指针
void BTree::newRoot(BTNode*& t, BTNode* q, int x, BTNode* ap) {
	q = new BTNode;
	q->keyNum = 1;
	q->keys[1] = x;
	q->ptr[0] = t;
	q->ptr[1] = ap;
	t->parent = q;
	ap->parent = q;
	this->root = q;
}

// 遍历所有节点查询key关键字
Result BTree::searchBT(BTNode* t, int k) {
	BTNode* p = t, * q = nullptr; // p指向当前查询节点，q指向当前节点的父节点
	bool found = false;
	int i = 0;
	while (p && !found) {
		i = search(p, k);	// 在当前节点中查找k
		if (p->keys[i + 1] == k)
			found = true;
		else {
			q = p;
			p = p->ptr[i];	// 转到下一个节点
		}
	}
	if (found)
		return Result(p, i + 1, 1);	// 查找成功，返回节点信息
	else
		return Result(q, i, 0);	// 查找失败，返回插入位置信息
}

// 参考课本244页算法9.14
void BTree::insertBT(BTNode*& t, int k, BTNode* q, int i) {
	int x = k;
	BTNode* ap = nullptr;
	bool finished = false;
	if (!q) {	// 创建第一个节点
		q = new BTNode;
		q->keyNum = 1;
		q->keys[1] = k;
		this->root = q;
		return;
	}
	while (!finished) {
		insert(q, i, x, ap);	// 将k和ap分别插入t->key[i+1]和t->ptr[i + 1]
		if (q->keyNum < m)	// 插入完成（keys从1开始，所以判断keyNum < m）
			finished = true;
		else {	// 分裂节点*q
			int s = (m + 1) / 2;	// 向上取整
			split(q, s, ap);	// 将q->key[s+1...m],q->ptr[s...m]移入新结点ap
			x = q->keys[s];
			q = q->parent;
			if (q)
				i = search(q, x);	//查找在父节点的插入位置
			else {
				newRoot(t, q, x, ap);
				finished = true;
			}
		}
	}
}

// B树初始化
BTree::BTree() {
	root = nullptr;
}

BTree::~BTree() {
	root = nullptr;
}

// B-树插入操作
void BTree::insertBT(int key) {
	Result r = searchBT(root, key);
	if (!r.tag) {
		insertBT(root, key, r.pt, r.i);
	}
}

// B-树查找操作
void BTree::searchBT(int key) {
	Result r = searchBT(root, key);
	if (!r.tag) {
		cout << "-1" << endl;
	}
	else {
		BTNode* p = r.pt;
		cout << p->keys[1];
		for (int i = 2; i <= p->keyNum; i++) {
			cout << ':' << p->keys[i];
		}
		cout << ' ' << r.i << endl;
	}
	return;
}

// B-树层次遍历输出关键字
void BTree::levelOrder() {
	queue<BTNode*> tq;
	BTNode* p = root;
	// 首结点入队
	if (p) {
		tq.push(p);
	}
	// 层次遍历树
	while (!tq.empty()) {
		p = tq.front();
		tq.pop();
		// 输出结点p的key
		cout << p->keys[1];
		for (int i = 2; i <= p->keyNum; i++) {
			cout << ':' << p->keys[i];
		}
		cout << ' ';
		// 自结点入栈
		for (int i = 0; i <= p->keyNum; i++) {
			if (!p->ptr[i]) {
				break;
			}
			tq.push(p->ptr[i]);
		}
	}
	return;
}

int main(void) {
	int t;
	cin >> t;
	while (t--) {
		cin >> m;
		int n, k, key;
		// 构建B-树
		cin >> n;
		BTree bTree;
		while (n--) {
			cin >> key;
			bTree.insertBT(key);
		}
		// 按层次遍历输出B-树
		bTree.levelOrder();
		cout << endl;
		// 查找B-树结点
		//cin >> k;
		//while (k--) {
		//	cin >> key;
		//	bTree.searchBT(key);
		//}
	}
	return 0;
}
```

## 参考资料

https://www.yiibai.com/data_structure/b-tree.html

https://blog.csdn.net/alzzw/article/details/97663352

《数据结构（C语言版）》清华大学出版社

&emsp;&ensp;

&emsp;&ensp;