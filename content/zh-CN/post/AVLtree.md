+++
title = "手撸AVL树"
date = "2021-06-03T17:47:39+08:00"
categories = [
    "数据结构","算法"
]
tags = [
    "AVL树",
    "平衡二叉树",
    "数据结构"
]
toc = true
+++

Wiki:在计算机科学中，AVL树是最早被发明的自平衡二叉查找树。在AVL树中，任一节点对应的两棵子树的最大高度差为1，因此它也被称为高度平衡树。查找、插入和删除在平均和最坏情况下的时间复杂度都是O(logn)。

<!--more-->

先丢个代码，周末再补原理介绍。

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
			case EH:t->bf = LH; taller = true; break; //原本等高，现在左侧高
			case RH:t->bf = EH; taller = false; break; //原本右侧高，现在等高
			}
	}
	else {	//搜寻右子树
		if (!insertAVL(t->rChild, key, taller))
			return 0;
		if(taller)
			switch (t->bf) {
			case LH:t->bf = EH; taller = false; break;
			case EH:t->bf = RH; taller = true; break;
			case RH:rightBalance(t); taller = false; break;
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

//https://www.jianshu.com/p/fdb3c8c331f1

/*
输入
第一行输入测试数据组数t；
每组测试数据，第一行输入结点数n, 第二行输入n个结点值。
输出
对每组测试数据，按中序遍历的顺序输出树中，结点值及平衡因子（测试数据没有空树），即结点值：平衡因子，不同结点之间间隔一个空格。

样例输入
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
样例输出
1:0 5:0 64:0
5:0 13:0 64:0
1:0 5:1 13:0 15:0 64:0 78:0
1:0 5:0 10:0 13:0 64:-1 78:0
64:0 78:0 100:0
64:0 70:0 80:0
30:0 64:0 68:0 70:0 80:-1 90:0
30:0 64:1 70:0 75:0 80:0 90:0
*/

```