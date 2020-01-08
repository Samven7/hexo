---
title: 【剑指Offer】复杂链表的复制
urlname: 2019102001
top: false
cover: false
toc: true
mathjax: false
tags:
  - 链表
categories:
  - 【剑指Offer】
abbrlink: 60762
date: 2019-10-20 11:06:22
author:
img:
coverImg:
password:
summary:
---

### 【题目】

 输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空） 

<!-- more -->

### 【思路】

思路主要是在原来链表的基础上进行复制，分三步：

（1）遍历一次，在原链表每个节点后面新建一个和该节点值相同的节点，并插入进去。

（2）再遍历一次，处理原链表的random指针，把它复制到新建节点上面。

（3）最后遍历一次，把复制的链表抽离出来。

注意：第三步抽离的时候一定要把和原链表连接的所有指针断开。

### 【代码】

~~~cpp
/*
struct RandomListNode {
    int label;
    struct RandomListNode *next, *random;
    RandomListNode(int x) :
            label(x), next(nullptr), random(nullptr) {
    }
};
*/
class Solution {
public:
	RandomListNode* Clone(RandomListNode* pHead)
	{
		if (pHead == nullptr) {
			return nullptr;
		}
		RandomListNode *pNode = pHead;
		// 第一步，复制next指针所指的结点
		while (pNode != nullptr) {
			RandomListNode *pClone = new RandomListNode(pNode->label);
			pClone->next = pNode->next;
			pNode->next = pClone;

			pNode = pClone->next;
		}
		// 第二步，处理random指针
		pNode = pHead;
		while (pNode != nullptr) {
			if (pNode->random == nullptr) {
				pNode->next->random = nullptr;
			}
			else {
				pNode->next->random = pNode->random->next;
			}
			pNode = pNode->next->next;
		}
		// 第三步，抽离出复制好的链表
		RandomListNode *oldNode = pHead;
		RandomListNode *newHead = pHead->next;
		pNode = pHead->next;
		while (pNode != nullptr) {
			oldNode->next = pNode->next;
			if (pNode->next != nullptr) {
				pNode->next = pNode->next->next;
			}
			pNode = pNode->next;
			oldNode = oldNode->next;
		}
		// 返回新链表的头指针
		return newHead;
	}
};
~~~