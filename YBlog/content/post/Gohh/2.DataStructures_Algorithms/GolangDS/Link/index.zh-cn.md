---
title: "链表(Go语言实现)"
date: 2024-03-10T16:21:29+08:00
draft: false
taps: ["List", "Golang"]
categories: ["Golang", "CS", "DataStructures"]
author: ["Yeelight"]
showtoc: true
weight:
readingTime: true
---


## 链表介绍

链表是一种常见的数据结构，用于存储一系列元素。链表由一个个节点组成，每个节点包含数据和指向下一个节点的指针。链表的最后一个节点指向空值（nil），表示链表的结束。

链表有多种类型，包括单向链表、双向链表和循环链表。其中：

- 单向链表：每个节点包含指向下一个节点的指针。
- 双向链表：每个节点包含指向下一个节点和上一个节点的指针，因此可以双向遍历链表。
- 循环链表：最后一个节点指向第一个节点，形成一个循环。

链表相对于数组的优势在于插入和删除操作的效率较高，因为它们不需要移动大量元素。然而，链表的缺点是访问任意位置的元素的效率较低，因为它们需要从头开始遍历链表。

## Go 语言实现

```go
package linkedlist

import "fmt"

/// Linked List in Golang

// Linked List Data Structure

type node struct {
	data interface{}
	next *node
}
func newLinkNode(data interface{}) *node{
	return &node{
		data : data,
		next : nil,
	}
}

type LinkedList struct {
	head  *node
	size  uint
}
// NewLinkedList create a linked list
func NewLinkedList() *LinkedList {
	return &LinkedList{
		head : nil,
		size: 0,
	}
}


/// Public function of linked list

// Len get the linked list length
func (list *LinkedList) Len() uint {
	return list.size
}

// Insert insert on the head of the linked list
func (list *LinkedList) Insert(data int) *LinkedList {
	newNode := newLinkNode(data)
	newNode.next = list.head
	list.head = newNode
	list.size+=1
	return list
}

// RemoveHead delete head node at the linked list
func (list *LinkedList) RemoveHead(){
	if list.head == nil{
		return
	}
	list.head = list.head.next
	list.size-=1
}

// RemoveItem delete the n node
func (list *LinkedList) RemoveItem(n *node) {
	if n == nil {
		return
	}
	if n == list.head {
		list.RemoveHead()
		return
	}
	cur := list.head
	for cur.next != nil {
		if cur.next == n {
			cur.next = cur.next.next
			list.size-=1
			return
		}
		cur = cur.next
	}
}

// Find find the node
func (list *LinkedList) Find(data int) *node {
	if list.head == nil{
		return nil
	}
	cur := list.head
	for cur != nil{
		if cur.data == data{
			return cur
		}
		cur = cur.next
	}
	return nil
}

// Head get the head node
func (list *LinkedList) Head() *node {
	return list.head
}

// Print print the linked list
func (list *LinkedList) Print() string {
	var format string
	cur := list.head
	for cur != nil {
		format += fmt.Sprintf("%+v", cur.data)
		cur = cur.next
		if cur != nil {
			format += "->"
		}
	}
	// fmt.Println(format)
	return format
}

```

## 相关算法
