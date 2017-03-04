---
layout: post
title: "CopyOnWriteArray浅析"
description: ""
category:  java容器 
tags: [java, 容器, CopyOnWriteArray]
---
* CopyOnWriteArrayList的特点与用法
* CopyOnWriteArrayList的数据结构
* CopyOnWriteArrayList的常用方法及实现

## CopyOnWriteArrayList的特点与用法    
CopyOnWriteArrayList的数据结构和用法与ArrayList基本都相同,区别主要在于CopyOnWriteArrayList在添加和删除元素等修改集合状态的操作时，都会重新复制一个新的数组，以保证多线程下的安全性。因为每次修改数据结点都要复制整个对象数组，会比较耗时间，所以CopyOnWriteArrayList适合读多写少情况下的多线程场景，如缓存之类的。

## CopyOnWriteArrayList的数据结构     
CopyOnWriteArrayList的数据结构与ArrayList的基本一样，只是它不再有扩容之类的操作，因为每次修改都是一个新的数组。。。

## CopyOnWriteArrayList的常用方法及实现    
    
	public class CopyOnWriteArrayList<E>
		implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
		...
	}

从CopyOnWriteArrayList的类结构就可以看出，它的方法签名与ArrayList基本一致，这里就不再重复了。下面讲讲它方法的特殊实现。

1. 构造方法    
    CopyOnWriteArrayList的默认构造方法会创建一个长度为0的数组来储存数据，而不是像ArrayList那样创建一个长度为10的数组
		/**Sets the array. */    
		final void setArray(Object[] a) {
		  array = a;
		}
		
		/**
		* Creates an empty list.
		*/
		public CopyOnWriteArrayList() {
			setArray(new Object[0]);
		}

2. 添加方法

        /**
         * Appends the specified element to the end of this list.
         *
         * @param e element to be appended to this list
         * @return {@code true} (as specified by {@link Collection#add})
         */
        public boolean add(E e) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len + 1);
                newElements[len] = e;
                setArray(newElements);
                return true;
            } finally {
                lock.unlock();
            }
        }

        /**
         * Inserts the specified element at the specified position in this
         * list. Shifts the element currently at that position (if any) and
         * any subsequent elements to the right (adds one to their indices).
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public void add(int index, E element) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;
                if (index > len || index < 0)
                    throw new IndexOutOfBoundsException("Index: "+index+
                                                        ", Size: "+len);
                Object[] newElements;
                int numMoved = len - index;
                if (numMoved == 0)
                    newElements = Arrays.copyOf(elements, len + 1);
                else {
                    newElements = new Object[len + 1];
                    System.arraycopy(elements, 0, newElements, 0, index);
                    System.arraycopy(elements, index, newElements, index + 1,
                                     numMoved);
                }
                newElements[index] = element;
                setArray(newElements);
            } finally {
                lock.unlock();
            }
        }

3. 修改方法

   方法在进入和退出时有加锁、释放锁

        /**
         * Replaces the element at the specified position in this list with the
         * specified element.
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public E set(int index, E element) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                E oldValue = get(elements, index);

                if (oldValue != element) {
                    int len = elements.length;
                    Object[] newElements = Arrays.copyOf(elements, len);
                    newElements[index] = element;
                    setArray(newElements);
                } else {
                    // Not quite a no-op; ensures volatile write semantics
                    setArray(elements);
                }
                return oldValue;
            } finally {
                lock.unlock();
            }
        }

4. 删除方法

        /**
         * Removes the element at the specified position in this list.
         * Shifts any subsequent elements to the left (subtracts one from their
         * indices).  Returns the element that was removed from the list.
         *
         * @throws IndexOutOfBoundsException {@inheritDoc}
         */
        public E remove(int index) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;
                E oldValue = get(elements, index);
                int numMoved = len - index - 1;
                if (numMoved == 0)
                    setArray(Arrays.copyOf(elements, len - 1));
                else {
                    Object[] newElements = new Object[len - 1];
                    System.arraycopy(elements, 0, newElements, 0, index);
                    System.arraycopy(elements, index + 1, newElements, index,
                                     numMoved);
                    setArray(newElements);
                }
                return oldValue;
            } finally {
                lock.unlock();
            }
        }

        /**
         * Removes the first occurrence of the specified element from this list,
         * if it is present.  If this list does not contain the element, it is
         * unchanged.  More formally, removes the element with the lowest index
         * {@code i} such that
         * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
         * (if such an element exists).  Returns {@code true} if this list
         * contained the specified element (or equivalently, if this list
         * changed as a result of the call).
         *
         * @param o element to be removed from this list, if present
         * @return {@code true} if this list contained the specified element
         */
        public boolean remove(Object o) {
            Object[] snapshot = getArray();
            int index = indexOf(o, snapshot, 0, snapshot.length);
            return (index < 0) ? false : remove(o, snapshot, index);
        }

        /**
         * A version of remove(Object) using the strong hint that given
         * recent snapshot contains o at the given index.
         */
        private boolean remove(Object o, Object[] snapshot, int index) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] current = getArray();
                int len = current.length;
                if (snapshot != current) findIndex: {
                    int prefix = Math.min(index, len);
                    for (int i = 0; i < prefix; i++) {
                        if (current[i] != snapshot[i] && eq(o, current[i])) {
                            index = i;
                            break findIndex;
                        }
                    }
                    if (index >= len)
                        return false;
                    if (current[index] == o)
                        break findIndex;
                    index = indexOf(o, current, index, len);
                    if (index < 0)
                        return false;
                }
                Object[] newElements = new Object[len - 1];
                System.arraycopy(current, 0, newElements, 0, index);
                System.arraycopy(current, index + 1,
                                 newElements, index,
                                 len - index - 1);
                setArray(newElements);
                return true;
            } finally {
                lock.unlock();
            }
        }

        /**
         * Removes from this list all of the elements whose index is between
         * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
         * Shifts any succeeding elements to the left (reduces their index).
         * This call shortens the list by {@code (toIndex - fromIndex)} elements.
         * (If {@code toIndex==fromIndex}, this operation has no effect.)
         *
         * @param fromIndex index of first element to be removed
         * @param toIndex index after last element to be removed
         * @throws IndexOutOfBoundsException if fromIndex or toIndex out of range
         *         ({@code fromIndex < 0 || toIndex > size() || toIndex < fromIndex})
         */
        void removeRange(int fromIndex, int toIndex) {
            final ReentrantLock lock = this.lock;
            lock.lock();
            try {
                Object[] elements = getArray();
                int len = elements.length;

                if (fromIndex < 0 || toIndex > len || toIndex < fromIndex)
                    throw new IndexOutOfBoundsException();
                int newlen = len - (toIndex - fromIndex);
                int numMoved = len - toIndex;
                if (numMoved == 0)
                    setArray(Arrays.copyOf(elements, newlen));
                else {
                    Object[] newElements = new Object[newlen];
                    System.arraycopy(elements, 0, newElements, 0, fromIndex);
                    System.arraycopy(elements, toIndex, newElements,
                                     fromIndex, numMoved);
                    setArray(newElements);
                }
            } finally {
                lock.unlock();
            }
        }

5. 遍历

    遍历时，直接使用内部的数组，这样可以避开多线程时的问题,类似的还有indexOf

        public void forEach(Consumer<? super E> action) {
            if (action == null) throw new NullPointerException();
            Object[] elements = getArray();
            int len = elements.length;
            for (int i = 0; i < len; ++i) {
                @SuppressWarnings("unchecked") E e = (E) elements[i];
                action.accept(e);
            }
        }

        public int indexOf(E e, int index) {
            Object[] elements = getArray();
            return indexOf(e, elements, index, elements.length);
        }



----------------


