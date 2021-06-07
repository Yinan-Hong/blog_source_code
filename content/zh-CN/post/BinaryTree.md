+++
title = "二叉树从0到1"
date = "2021-05-15T15:18:48+08:00"
categories = [
    "算法",
	"数据结构"
]
tags = [
    "二叉树",
    "算法"
]
toc = true
+++

本文介绍关于二叉树的一些基本概念，典型操作，和我做过的比较入门的算法题。所以本文比较适合初学算法的同学，或者用来快速复习什么的。如果是对二叉树的算法比较熟悉的同学，建议可以直接去刷相关的算法题。

<!--more-->

## 二叉树概念

二叉树就是每个节点最多有两个子节点的树。每个节点的内存大小包括【要存储的数据】、【指向子节点的两个指针】。

![](../BinaryTreePic/1.png)

需要了解的几个概念，图中Tree 1就是一般的二叉树，Tree 2是**完全二叉树**，Tree 3是**满二叉树**。**完全二叉树**除最下面一层，其余层的节点个数必须达到最大，最底层节点全部靠左排列。**满二叉树**则是所有层的结点个数都需达到最大，显然满二叉树都是完全二叉树。

![](../BinaryTreePic/2.png)

像上图的数据结构中，发现6、8、9号位置都没有存到数据，会有空间的浪费。而用**完全二叉树**存储就可以避免这样的问题。实际上所有的树结构都是可以转化成完全二叉树的。




## 二叉树的遍历操作

遍历就是逐个访问所有节点，而根据**访问根节点和左右子节点顺序**的不同，二叉树的遍历分为**前序、中序、后序遍历**。如果把一个运算式的字符串存入二叉树，按照中序输出得到"1+2*3-4"这样的中缀表达式，那按照前序输出就会得到它的**波兰式**，按后序输出就会得到它的**逆波兰式**。

<img src="../BinaryTreePic/3.png" alt="4e71b9f29ad47929213242aa8683a42" style="zoom:67%;" />



### 前序遍历

![](../BinaryTreePic/4.jpg)

前序遍历就是先访问根节点，再访问左子节点，再访问右子节点。上图的输出结果为ABDEGHCF。一些简单的算法题会给出前序遍历的数据，需要**通过前序遍历的数据建立二叉树**，然后进行其他二叉树的访问操作。

前序遍历的代码（递归实现）很简单：

```c++
void pre_order(Node* n) {
	if (n != NULL) {
		cout << n->data << ' '; //访问本节点数据
		pre_order(n->left);
		pre_order(n->right);
	}
}
```



### 中序遍历

中序遍历是先访问左子节点，再访问根节点，再访问右子节点。按上面同一棵树的输出结果应为DBGEHACF，图我就不画了太麻烦了...代码的区别其实就是换一下顺序。

```c++
void in_order(Node* n) {
    if (n != NULL) {
        pre_order(n->left);
        cout << n->data << ' '; //访问本节点数据
        pre_order(n->right);
    }
}
```



### 后序遍历

后序遍历是先访问左右节点，最后访问根节点。上面那棵树的后序输出为DGHEBFCA。

```c++
void in_order(Node* n) {
    if (n != NULL) {
        pre_order(n->left);
        pre_order(n->right);
        cout << n->data << ' '; //访问本节点数据
    }
}
```



### 逐层遍历

逐层遍历的意思是从左到右访问每一层的每一个节点。还是上面那棵树，逐层输出就是ABCDEFGH。逐层输出要用到**队列**，每一层的节点会先被放入队列，访问每一个节点时，将子节点放入队尾。每次循环取出队首的节点，就能实现从上到下，从左到右访问所有节点。

```c++
#include <queue>
void BinaryTree::level_order() {
	if (root == NULL)
		return;
	queue<Node*> q;
	q.push(root);
	while (!q.empty()) {
		Node* n;
		n = q.front();	//取出队首的节点
		q.pop();
		cout << n->data << ' '; //访问节点数据
		if (n->left != NULL)
			q.push(n->left);	//将当前节点的左右子节点放入队列
		if (n->right != NULL)
			q.push(n->right);
	}
}
```



###  我写的一个二叉树模板类

因为想着这样不用改很多代码就可以实现存不同数据类型二叉树，所以就无聊写了个模板。这个类实现了二叉树中后序遍历，层序遍历，计算高度这些常用操作。

```c++
#include <iostream>
#include <string>
#include <queue>
using namespace std;

template <typename T>
class Node {
public:
	T data;
	Node* left;		//左子节点
	Node* right;	//右子节点
};

template <typename T>
class BinaryTree {
private:
	Node<T>* root;		//根节点
	int height;		//高度

	Node<T>* create_tree(const T* s, int& pos, int s_len);
	void pre_order(Node<T>* n);
	void in_order(Node<T>* n);
	void post_order(Node<T>* n);
	void level_order(Node<T>* n);
	void get_height(Node<T>* n, int h);	//计算高度

public:
	BinaryTree();
	void create_tree(const T* s, int s_len);
	void pre_order();		//前序遍历
	void in_order();		//中序遍历
	void post_order();		//后序遍历
	void level_order();		//层序遍历
	int get_height();		//获取高度
	void ancestor(char A, char B);	//todo求两个节点的最大公共祖先
};

template <typename T>
BinaryTree<T>::BinaryTree() {
	root = NULL;
	height = -1;
}

template <typename T>
Node<T>* BinaryTree<T>::create_tree(const T* s, int& pos, int s_len) {
	++pos;
	Node<T>* n;
	if (pos >= s_len)
		return NULL;
	else {
		if (s[pos] == '#')		//该位置没有节点
			n = NULL;
		else {
			n = new Node<T>;
			n->data = s[pos];
			n->left = create_tree(s, pos, s_len);
			n->right = create_tree(s, pos, s_len);
		}
		return n;
	}
}

template <typename T>
void BinaryTree<T>::create_tree(const T* s, int s_len) {
	int pos = -1;
}

template <typename T>
void BinaryTree<T>::pre_order() {	//在公有接口中访问数据，保证私有数据的封装
	pre_order(root);
	cout << endl;
}

template <typename T>
void BinaryTree<T>::pre_order(Node<T>* n) {
	if (n != NULL) {
		//do sth
		cout << n->data << ' ';
		pre_order(n->left);
		pre_order(n->right);
	}
}

template <typename T>
void BinaryTree<T>::in_order() {
	in_order(root);
	cout << endl;
}

template <typename T>
void BinaryTree<T>::in_order(Node<T>* n) {
	if (n != NULL) {
		in_order(n->left);
		//do sth
		cout << n->data << ' ';
		in_order(n->right);
	}
}

template <typename T>
void BinaryTree<T>::post_order() {
	post_order(root);
	cout << endl;
}

template <typename T>
void BinaryTree<T>::post_order(Node<T>* n) {
	if (n != NULL) {
		post_order(n->left);
		post_order(n->right);
		//do sth
		cout << n->data << ' ';
	}
}

//用队列进行层序遍历
template <typename T>
void BinaryTree<T>::level_order() {
	if (root == NULL)
		return;
	queue<Node<T>*>q;
	q.push(root);
	while (!q.empty()) {
		Node<T>* n;
		n = q.front();
		q.pop();
		//do sth
		cout << n->data << ' ';
		if (n->left != NULL)
			q.push(n->left);
		if (n->right != NULL)
			q.push(n->right);
	}
	//do sth
	cout << endl;
}

template <typename T>
int BinaryTree<T>::get_height() {
	if (height == -1)
		get_height(root, 0);
	return height;
}

template <typename T>
void BinaryTree<T>::get_height(Node<T>* n, int h) {
	if (n != NULL) {
		++h;
		if (h > height)
			height = h;
		get_height(n->left, h);
		get_height(n->right, h);
	}
}

int main() {
//数据类型为char，#表示此处没有节点
	string s;
	s = "ABD##E#F##C##";
	BinaryTree<char> a;

//数据类型为int，0表示没有
	//int s[] = { 1,2,3,0,0,4,0,5,0,0,6,0,0 };
	//BinaryTree<int> a;

	a.create_tree(s.c_str(), s.size());
	//a.create_tree(s, 13);
	cout << "前序遍历：" << endl;
	a.pre_order();
	cout << "中序遍历：" << endl;
	a.in_order();
	cout << "后序遍历：" << endl;
	a.post_order();
	cout << "层序遍历：" << endl;
	a.level_order();
	cout << "树高：" << endl;
	cout << a.get_height() << endl;
	return 0;
}
```



## 一些比较常规的算法题

大部分是学校数据结构课做的。挑一些比较有意思的放出来，会尽量注释详细。



### 二叉树结点的最大距离

#### 题目描述

二叉树两个结点的距离是一个结点经过双亲结点，祖先结点等中间结点到达另一个结点经过的分支数。二叉树结点的最大距离是所有结点间距离的最大值。例如，下图所示二叉树结点最大距离是3，C和D的距离。
二叉树用先序遍历顺序创建，#表示空树。计算二叉树结点最大距离和最大距离的两个结点(假设二叉树中取最大距离的两个结点唯一）。

<img src="../BinaryTreePic/5.png" alt="" style="zoom:50%;" />

#### 输入

测试次数T
第2行之后的T行，每行为一棵二叉树先序遍历结果（#表示空树）

#### 输出

 对每棵二叉树，输出树的结点最大距离和最大距离的结点，输出格式见样例。

#### 样例输入

```
3
A##
ABC##EF#G###D##
ABEH###F#K###
```

#### 样例输出

```
0:
5:G D
4:H K
```

#### 题目解析

![](../BinaryTreePic/6.jpg)

对每个节点，子树的最长路径有两种可能，一种是从根节点到叶子结点（左），一种是从叶子结点到另一个叶子节点（右），而leaf to leaf最大距离 = 根to左侧最深 + 根to右侧最深。所以只用递归采用后序遍历，对每个节点计算最长的根叶子距离和叶子到叶子距离，取较大的那个作为max distance返回。这样到根节点时，就能得到最大的节点路径。

#### AC代码

```c++
#include <iostream>
#include <string.h>
#include <queue>
using namespace std;

template <typename T>
class Node {
public:
	T data;
	Node* left;		//左子节点
	Node* right;	//右子节点
};

template <typename T>
class BinaryTree {
private:
	Node<T>* root;		//根节点
	Node<T>* create_tree(const T* s, int& pos, int s_len);
	int post_order(Node<T>* n, int* record);

public:
	BinaryTree();
	void create_tree(const T* s, int s_len);
	void post_order();		//后序遍历
	void max_distance();
};

template <typename T>
BinaryTree<T>::BinaryTree() {
	root = NULL;
	height = -1;
	leaves = 0;
}

template <typename T>
Node<T>* BinaryTree<T>::create_tree(const T* s, int& pos, int s_len) {
	++pos;
	Node<T>* n;
	if (pos >= s_len)
		return NULL;
	else {
		if (s[pos] == '#')		//该位置没有节点
			n = NULL;
		else {
			n = new Node<T>;
			n->data = s[pos];
			n->left = create_tree(s, pos, s_len);
			n->right = create_tree(s, pos, s_len);
		}
		return n;
	}
}

template <typename T>
void BinaryTree<T>::create_tree(const T* s, int s_len) {
	int pos = -1;
	root = create_tree(s, pos, s_len);
}

template <typename T>
void BinaryTree<T>::pre_order() {
	pre_order(root);
	cout << leaves << endl;
}

template <typename T>
void BinaryTree<T>::max_distance() {
	int* record = new int;
	int num = post_order(root, record);
	if (num)
		cout << num << ":" << root->l_deepest << " " << root->r_deepest << endl;
	else
		cout << 0 << ":" << endl;
	delete record;
}


//最大距离可能为
//1.node左子树上最大距离
//2.node右字数上最大距离
//3.node左子树深度+右子树深度

template <typename T>
int BinaryTree<T>::post_order(Node<T>* n, int* record) {
	if (n == NULL) {
		*record = 0;	//存深度
		return 0;
	}
	int l_max = post_order(n->left, record);	//左子树上最大距离
	int l_depth = *record;						//左子树深度
	int r_max = post_order(n->right, record);	//右子树上最大距离
	int r_depth = *record;						//右子树深度
	int cur_node_depth = l_depth + r_depth;		//左子树深度+右子树深度

	*record = max<int>(l_depth, r_depth) + 1;	//当前节点深度
	return max<int>(max<int>(l_max, r_max), cur_node_depth); //返回当前节点的最大距离
}				//也就是当前节点作为根节点，其子节点的最大距离

int main() {
	int t;
	cin >> t;
	while (t--) {
		char* s = new char[100];
		cin >> s;
		BinaryTree<char> bt;
		bt.create_tree(s, strlen(s));
		bt.max_distance();
		delete[] s;
	}
	return 0;
}
```



### 求根到叶子结点路径上权值的和

#### 题目描述

给定一棵二叉树的逻辑结构（先序遍历的结果，空树用字符‘0’表示，例如AB0C00D00），建立该二叉树的二叉链式存储结构
二叉树的每个结点都有一个权值，从根结点到每个叶子结点将形成一条路径，每条路径的权值等于路径上所有结点的权值和。编程求出二叉树的最大路径权值。如下图所示，共有4个叶子即有4条路径

路径1权值=5 + 4 + 11 + 7 = 27          路径2权值=5 + 4 + 11 + 2 = 22
路径3权值=5 + 8 + 13 = 26                路径4权值=5 + 8 + 4 + 1 = 18
可计算出最大路径权值是27

该树输入的先序遍历结果为ABCD00E000FG00H0I00，各结点权值为：
A-5，B-4，C-11，D-7，E-2，F-8，G-13，H-4，I-1

<img src="../BinaryTreePic/7.png" alt="995ec06a8ed943fc06b350fc620126c" style="zoom:50%;" />



#### 输入

第一行输入一个整数t，表示有t个测试数据
第二行输入一棵二叉树的先序遍历，每个结点用字母表示
第三行先输入n表示二叉树的结点数量，然后输入每个结点的权值，权值顺序与前面结点输入顺序对应
以此类推输入下一棵二叉树

#### 输出

每行输出每棵二叉树的最大路径权值，如果最大路径权值有重复，只输出1个

#### 样例输入

```
2
AB0C00D00
4 5 3 2 6
ABCD00E000FG00H0I00
9 5 4 11 7 2 8 13 4 1
```

#### 样例输出

```
11
27
```



#### 题目解析

先序建树，然后后序遍历节点，记录访问过的节点的数据，访问根节点时比较取权值较大的子树。比如图中7>2，访问11时就取7。递归到最后得到的就是权值最大的数据。

其实可以优化，**边建树边计算数据**，效率会更高，但是做的时候没想到，就不写了。

#### AC代码

```c++
#include <iostream>
using namespace std;
int cnt;
int* dt;

template <typename T>
class Node {
public:
	T data;
	Node* left;		//左子节点
	Node* right;	//右子节点
};

template <typename T>
class BinaryTree {
private:
	Node<T>* root;		//根节点
	int height;		//高度
	Node<T>* create_tree(const char* s, int& pos, int s_len);
	int post_order(Node<T>* n);

public:
	BinaryTree();
	void create_tree(const char* s, int s_len);
	void post_order();		//后序遍历
};

template <typename T>
BinaryTree<T>::BinaryTree() {
	root = NULL;
	height = -1;
}

template <typename T>
Node<T>* BinaryTree<T>::create_tree(const char* s, int& pos, int s_len) {
	++pos;
	Node<T>* n;
	if (pos >= s_len)
		return NULL;
	else {
		if (s[pos] == '0')		//该位置没有节点
			n = NULL;
		else {
			n = new Node<T>;
			n->data = dt[cnt];
			cnt++;
			n->left = create_tree(s, pos, s_len);
			n->right = create_tree(s, pos, s_len);
		}
		return n;
	}
}

template <typename T>
void BinaryTree<T>::create_tree(const char* s, int s_len) {
	int pos = -1;
	root = create_tree(s, pos, s_len);
}

template <typename T>
void BinaryTree<T>::post_order() {
	cout << post_order(root) << endl;
}

template <typename T>
int BinaryTree<T>::post_order(Node<T>* n) {
	if (n == NULL)
		return 0;
	int max1, max2;
	max1 = post_order(n->left);
	max2 = post_order(n->right);
	return max1 > max2 ? max1 + n->data : max2 + n->data;	//返回权值较大的子树
}

int main() {
	int t;
	cin >> t;
	while (t--) {
		string s;
		cin >> s;	//s存的节点名称
		int num;
		cin >> num;
		dt = new int[num];
		for (int i = 0; i < num; i++)
			cin >> dt[i];	//存节点的数据
		BinaryTree <int> btree;
		btree.create_tree(s.c_str(), s.size());
		btree.post_order();
		delete[] dt;
	}
	return 0;
}
```



### 后序遍历的非递归算法

因为递归的实现是通过栈，递归到下一层的时候，把当前的函数压栈。所以如果有一棵树特别长，就会发生**栈溢出**。所有的递归，都是可以通过非递归的方式实现的，这就是这道算法题的背景知识...

#### 输入

第一行输入一个整数t，表示有t个测试数据
第二行起输入二叉树先序遍历的结果，空树用字符‘0’表示，输入t行

#### 输出

逐行输出每个二叉树的后序遍历结果

#### 样例输入

```
3
AB0C00D00
ABC00D00EF000
ABCD0000E0F00
```



#### 样例输出

```
CBDA
CDBFEA
DCBFEA
```



#### 直接上代码

```c++
#include <iostream>
#include <string>
#include <stack>
using namespace std;

template <typename T>
class Node {
public:
	T data;
	Node* left;		//左子节点
	Node* right;	//右子节点
};

template <typename T>
class BinaryTree {
private:
	Node<T>* root;		//根节点
	Node<T>* create_tree(const T* s, int& pos, int s_len);
	void post_order(Node<T>* n);

public:
	BinaryTree();
	void create_tree(const T* s, int s_len);
	void post_order();		//后序遍历
};

template <typename T>
BinaryTree<T>::BinaryTree() {
	root = NULL;
}

template <typename T>
Node<T>* BinaryTree<T>::create_tree(const T* s, int& pos, int s_len) {
	++pos;
	Node<T>* n;
	if (pos >= s_len)
		return NULL;
	else {
		if (s[pos] == '0')		//该位置没有节点
			n = NULL;
		else {
			n = new Node<T>;
			n->data = s[pos];
			n->left = create_tree(s, pos, s_len);
			n->right = create_tree(s, pos, s_len);
		}
		return n;
	}
}

template <typename T>
void BinaryTree<T>::create_tree(const T* s, int s_len) {
	int pos = -1;
	root = create_tree(s, pos, s_len);
}

template <typename T>
void BinaryTree<T>::post_order() {
	post_order(root);
	cout << endl;
}

template <typename T>
void BinaryTree<T>::post_order(Node<T>* n) {
	Node<T>* p;
	stack<Node<T>*> s1;
	stack<int> s2;
	int tag;
	p = n;
	do {
		if (p != NULL) {
			tag = 0;
			s1.push(p);
			s2.push(tag);
			p = p->left;
		}
		if (s1.empty())
			break;
		if (p == NULL) {
			tag = s2.top();
			if (tag == 0) {
				s2.pop();
				s2.push(1);
				p = s1.top()->right;
			}
			if (tag == 1) {
				p = s1.top();
				cout << p->data;
				s1.pop();
				s2.pop();
				p = NULL;
			}
		}
	} while (!s1.empty());
}

int main() {
	int t;
	cin >> t;
	while (t--) {
		string s;
		cin >> s;
		BinaryTree<char> a;
		a.create_tree(s.c_str(), s.size());
		a.post_order();
	}
		return 0;
}

```



### 用中后序输出的数据建树

#### 题目描述

按中序遍历和后序遍历给出一棵二叉树，求这棵二叉树中叶子节点权值的最小值。
输入保证叶子节点的权值各不相同。

#### 输入

第一行输入一个整数t，表示有t组测试数据。
对于每组测试数据，首先输入一个整数N (1 <= N <= 10000)，代表二叉树有N个节点，接下来的一行输入这棵二叉树中序遍历的结果，最后一行输入这棵二叉树后序遍历的结果。

#### 输出

对于每组测试数据，输出一个整数，代表二叉树中叶子节点权值最小值。

#### 样例输入

```
3
7
3 2 1 4 5 7 6
3 1 2 5 6 7 4
8 7 8 11 3 5 16 12 18
8 3 11 7 16 18 12 5 1
255
255
```

#### 样例输出

```
1
3
255
```

#### 题目解析

因为后续输出的最后一个一定是根节点，就可以通过后序的数据**在中序的数据中找到根节点**，然后中序的数据中，**根节点左边一定是左子树，右边一定是右子树**。中序是先输出左子树，后序也是先输出左子树，找到后序的左子树部分的最后一个节点，就是左子树的根节点。详细的看代码。

#### AC代码

```c++
#include <iostream>
using namespace std;

class Node {
public:
	int data;
	Node* left, * right;
};

class BiTree {
private:
	Node* root;
	int num;	// num of nodes
	int* post;
	int* in;
	int min;
	Node* CreateTree(int* post, int* in, int n);
	void preorder(Node* p);
	void getMin(Node* p);
public:
	BiTree();
	~BiTree();
	void CreateTree();
	void preorder();
	int getMin();
};

BiTree::BiTree() {
	cin >> num;
	post = new int[num];
	in = new int[num];
	for (int i = 0; i < num; i++)
		cin >> in[i];
	for (int i = 0; i < num; i++)
		cin >> post[i];
	root = CreateTree(in, post, num - 1);
}

BiTree::~BiTree() {
	//理论上是要删节点的
	delete[] post;
	delete[] in;
}

Node* BiTree::CreateTree(int* in, int* post, int n) { //n是新建节点在post的位置
	if (n == -1)
		return nullptr;
	int i = 0;
	for (; in[i] != post[n]; i++);		//用i记录在in串的位置
	Node* node = new Node;
	node->data = post[n];
	node->left = CreateTree(in, post, i - 1);
	//in指针起始位置移当前结点的下一个，剩下的串为右子树节点
	//post指针【起始位置】移到后移i位，i为左子树的结点数，剩下的串为右子树节点
	//post[n-i-1]是下一个递归的post[n]
	node->right = CreateTree(in + i + 1, post + i, n - i - 1);
	return node;
}

void BiTree::getMin(Node* p) {
	if (p->left == nullptr && p->right == nullptr) {
		if (p->data < min)
			min = p->data;
		return;
	}
	if (p->left != nullptr)
		getMin(p->left);
	if (p->right != nullptr)
		getMin(p->right);
}

int BiTree::getMin() {
    //其实也是在建树的时候就可以计算数据，时间会少一半左右
    //但是建树的代码比较复杂，所以就不让它更复杂了吧
    //这个就是简单的前序遍历找权值最小的叶子结点
	min = root->data;
	getMin(root);
	return min;
}

int main() {
	int t;
	cin >> t;
	while (t--) {
		BiTree mytree;
		cout << mytree.getMin() << endl;
	}
}
```

### 二叉树的中后序遍历及操作

#### 题目描述

按中序遍历和后序遍历给出一棵二叉树，现在有如下操作：

1. UPDATE A B，将中序遍历中A位置（从1开始编号的下标）对应的在二叉树中的节点的权值改为B
2. QUERY，询问树上所有节点的权值，以及从根节点到该节点的路径权值之和
3. STOP，停止操作，STOP操作一定出现在最后
中序遍历和后序遍历的输入保证叶子节点的权值各不相同。但是，之后如果存在UPDATE操作，则**UPDATE操作可能会使得两个或两个以上的叶子节点的权值相同**。

#### 输入

输入t组测试数据；
接下来输入k组测试数据，对于每组测试数据：
第一行输入这棵二叉树的结点数
第二行输入这棵二叉树中序遍历的结果

第三行输入这棵二叉树后序遍历的结果
接下来每一行输入一种操作，直到输入STOP操作时结束本组测试数据的输入。其中，QUERY操作次数 <= 2，总操作数 <= 104

#### 输出

对于每组测试数据：
对于QUERY操作，按中序遍历输出节点的权值，以及从根节点到该节点路径权值之和。这里，我们认为根节点到其本身的路径权值之和为根节点的权值

#### 题目解析

因为update操作可能会导致权值相同，所以用权值查找节点可能找的不准，要用序号找，注意到这一点就应该没啥问题了。然后每查询一次就遍历一次其实效率会比较低，可以用一个指针数组，存序号对应的节点，然后update的时候从数组访问节点。那这么一看好像连建树都不用了...不过我做的时候没想到，用的update一次就遍历一次的方法QwQ

#### 样例输入

```
3
7
3 2 1 4 5 7 6
3 1 2 5 6 7 4
UPDATE 5 99
UPDATE 6 123
UPDATE 1 37
QUERY
UPDATE 1 36
QUERY
STOP
8
7 8 11 3 5 16 12 18
8 3 11 7 16 18 12 5
QUERY
STOP
1
255
255
STOP
```

#### 样例输出

```
37 43
2 6
1 7
4 4
99 226
123 127
6 133
36 42
2 6
1 7
4 4
99 226
123 127
6 133

7 12
8 31
11 23
3 26
5 5
16 33
12 17
18 35
```


#### AC代码

```c++
#include <iostream>
using namespace std;

class Node {
public:
	int data;
	Node* left, * right;
};

class BiTree {
private:
	Node* root;
	int num;	// num of nodes
	int* post;
	int* in;
	int a;
	int b;
	Node* CreateTree(int* post, int* in, int n);
	void inOrder(Node* n, int& cnt);
	void query(Node* n, int cnt);
public:
	BiTree();
	~BiTree();
	void update();
	void query();
};

BiTree::BiTree() {
	cin >> num;
	post = new int[num];
	in = new int[num];
	for (int i = 0; i < num; i++)
		cin >> in[i];
	for (int i = 0; i < num; i++)
		cin >> post[i];
	root = CreateTree(in, post, num - 1);
}

BiTree::~BiTree() {
	delete[] post;
	delete[] in;
}

Node* BiTree::CreateTree(int* in, int* post, int n) {
	if (n == -1)
		return nullptr;
	int i = 0;
	for (; in[i] != post[n]; i++);
	Node* node = new Node;
	node->data = post[n];
	node->left = CreateTree(in, post, i - 1);
	node->right = CreateTree(in + i + 1, post + i, n - i - 1);
	return node;
}

void BiTree::update() {
	cin >> a >> b;
	int cnt = 0;
	inOrder(root, cnt);
}

void BiTree::inOrder(Node* n, int& cnt) {
	if (n == nullptr)
		return;

	inOrder(n->left, cnt);
	cnt++;
	if (cnt == a)
		n->data = b;
	else
		inOrder(n->right, cnt);
	return;
}

void BiTree::query() {
	query(root, 0);
}

void BiTree::query(Node* n, int cnt) {
	if (n == nullptr)
		return;
	cnt += n->data;
	query(n->left, cnt);
	cout << n->data << ' ' << cnt << endl;
	query(n->right, cnt);
	return;
}

int main() {
	int t;
	cin >> t;
	while (t--) {
		BiTree mytree;
		string cmd;
		while (cin >> cmd) {
			if (cmd[0] == 'U')
				mytree.update();
			if (cmd[0] == 'Q')
				mytree.query();
			if (cmd[0] == 'S')
				break;
		}
		cout << endl;
	}
}
```



### Falling Leaves （DFS求每一列权值的和）

#### 题目描述

按先序遍历给出一棵二叉树，每个结点都有一个水平位置：左子结点在它左边一个单位，右子结点在右边1个单位。从左向右输出每个水平位置的所有节点的权值之和。
例如：以下二叉树有三个水平位置，从左至右的输出是7,11,3。

<img src="../BinaryTreePic/8.png" alt="f851422df82b29b2006ceb74573b7dc" style="zoom:50%;" />

#### 输入

测试数据有多组，每组测试数据输入按先序遍历输入一棵二叉树，其中-1代表空节点（一棵树的节点数量不超过 103）
当输入的二叉树是一棵空树时，结束输入。

#### 输出

对于每组测试数据，首先输出一行"Case x:"，其中x代表这是第x个测试数据的输出，然后从左到右输出每个水平位置所有节点的权值之和

#### 样例输入

```
5 7 -1 6 -1 -1 3 -1 -1
8 2 9 -1 -1 6 5 -1 -1 12 -1 -1 3 7 -1 -1 -1
-1
```

#### 样例输出

```
Case 1:
7 11 3

Case 2:
9 7 21 15
```

#### 题目解析

这个题目说的不是人话。用人话说是，输出每一竖列节点的权值和。其实这道题不用建树，大概判断一下最多可能有多少列，然后开个数组存每一竖列的权值和。因为先序的数据是根、左、右，所以只用写个巧妙的递归就能计算出结果。

#### AC代码

```c++
#include <iostream>
#include <string.h>
using namespace std;

const int maxn = 200;
int sum[maxn];

void build(int p) {
	int v;
	cin >> v;
	if (v == -1)
		return;	//空树返回
	sum[p] += v;
	build(p - 1);
	build(p + 1);
}

bool init() {
	int v;
	cin >> v;
	if (v == -1)
		return false;
	memset(sum, 0, sizeof(sum));
	int pos = maxn / 2;		//根节点位置
	sum[pos] = v;
	build(pos - 1);		//左子树
	build(pos + 1);		//右子树
}

int main() {
	int _case = 0;
	while (init()) {
		int p = 0;
		while (sum[p] == 0)
			p++;
		cout << "Case " << ++_case << ":\n" << sum[p++];
		while (sum[p] != 0)
			cout << ' ' << sum[p++];
		cout << "\n\n";
	}
	return 0;
}
```

