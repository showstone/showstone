---
layout: post
title: "LinkedList浅析"
description: ""
category: java容器
tags: [java,容器,LinkedList]
---

# LinkedList浅析
* LinkedList的作用及特点
* LinkedList的数据结构
* LinkedList的常用方法

## LinkedList的作用及特点
* LinkedList可作为链表或者双向队列来使用
* LinkedList的数据结构是基于链表来实现的，所以在它的开头、末尾添加结点的速度会非常快

> LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。     
> LinkedList 实现 List 接口，能对它进行队列操作。    
> LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。  
> LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。   
> LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。    
> LinkedList 是非同步的。
> 以上内容来自于 [skywang12345的博客][refer1]    

## LinkedList的数据结构
LinkedList的定义

	public class LinkedList<E>
	    extends AbstractSequentialList<E>
	    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
	{
	    transient int size = 0;

	    /**
	     * Pointer to first node.
	     * Invariant: (first == null && last == null) ||
	     *            (first.prev == null && first.item != null)
	     */
	    transient Node<E> first;

	    /**
	     * Pointer to last node.
	     * Invariant: (first == null && last == null) ||
	     *            (last.next == null && last.item != null)
	     */
	    transient Node<E> last;

	 ...

	}

Node的定义

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }


## LinkedList的常用方法

list接口定义的方法

	int size();
	boolean isEmpty();
	boolean contains(Object o);
	Iterator<E> iterator();
	Object[] toArray();
	<T> T[] toArray(T[] a);
	boolean add(E e);
	boolean remove(Object o);
	boolean containsAll(Collection<?> c);
	boolean addAll(Collection<? extends E> c);
	boolean addAll(int index, Collection<? extends E> c);
	boolean removeAll(Collection<?> c);
	boolean retainAll(Collection<?> c);
	default void replaceAll(UnaryOperator<E> operator) ;
	default void sort(Comparator<? super E> c);
	void clear();
	boolean equals(Object o);
	int hashCode();
	E get(int index);
	E set(int index, E element);
	void add(int index, E element);
	E remove(int index);
	int indexOf(Object o);
	int lastIndexOf(Object o);
	ListIterator<E> listIterator();
	ListIterator<E> listIterator(int index);
	List<E> subList(int fromIndex, int toIndex);

LinkedList自己的方法

	void addFirst(E e);
	void addLast(E e);
	boolean offerFirst(E e);
	boolean offerLast(E e);
	E removeFirst();
	E removeLast();
	E pollFirst();
	E pollLast();
	E getFirst();
	E getLast();
	E peekFirst();
	E peekLast();
	boolean removeFirstOccurrence(Object o);
	boolean removeLastOccurrence(Object o);
	boolean add(E e);
	boolean offer(E e);
	E remove();
	E poll();
	E element();
	E peek();
	void push(E e);
	E pop();
	boolean remove(Object o);
	boolean contains(Object o);
	public int size();
	Iterator<E> iterator();
	Iterator<E> descendingIterator();


总的来说，LinkedList里基本上都是这两个接口对应的方法
各个接口的实现，基本上都是比较正常的，这是不再重复描述

  [refer1]: http://www.cnblogs.com/skywang12345/p/3308807.html "skywang12345的博客"   



