---
layout: post
title: "ArrayList浅析"
description: ""
category: java容器
tags: [java,容器,ArrayList]
---

#ArrayList浅析
* ArrayList的作用、特点
* ArrayList的数据结构
* ArrayList的常用方法及实现
* ArrayList里需要注意的地方

###ArrayList的作用、特点
ArrayList实现了List接口，常用操作有add,remove,get,size。其内部的数据采用数组进行存储。因此随机读写的速度很快，但删除、添加等操作相对会消耗比较多的时间，因为会有相关的一系列节点移动。

###ArrayList的数据结构
ArrayList内部以数组的形式存储数据。默认的是一个空数组，当添加数据后，会扩充为一个长度为10的数组。

	默认的空数组
	/**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }
	
	private static final Object[] EMPTY_ELEMENTDATA = {};

###ArrayList的常用方法及其实现
1.add()

加入元素之前，先确保有足够的空间

    public boolean add(E e) {<br/>
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

指定位置加入元素时，之后所有的元素都需要后移

    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1, size - index);
        elementData[index] = element;
        size++;
    }

#### addAll

	public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

2.remove
	
删除之前，先检查是否越界,同时也有大量的元素移动操作

	public E remove(int index) {
        rangeCheck(index);//简单地检查下标是否越界

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

3.contain

    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

4.clone

	public Object clone() {
        try {
            @SuppressWarnings("unchecked")
                ArrayList<E> v = (ArrayList<E>) super.clone();
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // this shouldn't happen, since we are Cloneable
            throw new InternalError();
        }
    }

5.iterator

通过使用cursor和lastRet来进行标记

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;
		...
	}

###ArrayList里需要注意的地方

* sublist并没有创建一个新的ArrayList,只是加了一个下标的起点

代码：

	public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }

	 private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;
		...
	}

所以在原来的List或者Sublist里的操作，都会影响到另一个list

* 因为ArrayList是基于数组的，所以当添加的元素超过原来的数组大小时，它需要先创建一个新的数组，并把原来的元素复制过去。这在一定程序上会影响程序的性能。类似的操作有删除，根据上面的实现代码可以知道，每删除一个元素就会对它后面的元素进行移动。可以把那些需要删除的参数用一个list保存下来，然后用removeAll来一次全部删除。

代码

	public boolean removeAll(Collection<?> c) {
        return batchRemove(c, false);
    }
	
	private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)//这是一个比较巧妙的实现，值得学习
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }

* ArrayList里只定义了size和elementData两个元素,其他的都来自于父类AbstractList
	
>protected transient int modCount = 0;
>
>private int size;

### 以下是测试指定容器大小和让容器自动扩张比较的代码
	public static void main(String[] args) {
		long startTime = System.currentTimeMillis();
		for(int i=0;i<100000;i++){
			ArrayList list = new ArrayList();
			for(int num=0;num<1000;num++){
				list.add("object");
			}
		}
		System.out.println("用时:"+ (System.currentTimeMillis()-startTime)+"毫秒");
		startTime = System.currentTimeMillis();
		for(int i=0;i<100000;i++){
			ArrayList list = new ArrayList(2000);
			for(int num=0;num<1000;num++){
				list.add("object");
			}
		}
		System.out.println("用时:"+ (System.currentTimeMillis()-startTime)+"毫秒");
	}

运行结果:
>用时:607毫秒
>用时:415毫秒


