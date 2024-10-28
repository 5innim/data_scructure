```java
/*
 * Copyright (c) 2003, 2013, Oracle and/or its affiliates. All rights reserved.
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */
```


# Collections Framework 

Java의 Collections Framework는 데이터를 효율적으로 관리하고 조작할 수 있는 여러 자료구조와 알고리즘을 제공하는 프레임워크다.

Collections Framework에서 제공하는 자료구조는 크게 4가지 유형인 List, Queue, Set, Map으로 볼 수 있겠다.

먼저 List와 Queue에 대해 알아보자.

</br>

## Collections(List & Queue)
<img width="787" alt="스크린샷 2024-09-08 오후 11 38 51" src="https://github.com/user-attachments/assets/12be4616-745b-4d34-bd77-b0417bfdc4ce">

List와 Queue의 상속 구조는 위 사진처럼 복잡하다. 

순서대로 하나씩 알아보자. 

<br/>

### Iterable
Iterable은 반복 가능한 객체를 나타내는 인터페이스이다.
이를 구현하는 객체는 for-each 루프에서 사용할 수 있다. 
```java
public interface Iterable<T> {
    // Iterator 객체를 반환.
    Iterator<T> iterator();

    // Consumer 구현체를 인자로 받아 순차적으로 적용 및 수행.
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    // Spliterator 객체를 반환.
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}

```
- Iterator는 자바 컬렉션에서 요소들을 순차적으로 접근하고 처리하기위한 인터페이스(hasNext(), next(), remove(), forEachRemaining(Consumer<? super E> action))
- Spliterator는 Iterator의 확장판으로 데이터를 순차적으로 반복하는 것뿐만 아니라 데이터를 여러 개로 분할하여 병렬처리 지원을 위한 인터페이스, 스트림 API와 함께 주로 사용됨.

<br/>

### Collection
Collection 계층 구조에서 최상위 인터페이스이다. 모든 컬렉션의 기본적인 동작을 정의한다.
```java
public interface Collection<E> extends Iterable<E> {
    // Query Operations

    int size();

    boolean isEmpty();

    boolean contains(Object o);

    Iterator<E> iterator();

    Object[] toArray();

    <T> T[] toArray(T[] a);

    default <T> T[] toArray(IntFunction<T[]> generator) {
        return toArray(generator.apply(0));
    }

    // Modification Operations

    boolean add(E e);

    boolean remove(Object o);

    // Bulk Operations

    boolean containsAll(Collection<?> c);

    boolean addAll(Collection<? extends E> c);

    boolean removeAll(Collection<?> c);

    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }

    boolean retainAll(Collection<?> c);

    void clear();

    // Comparison and hashing

    boolean equals(Object o);

    int hashCode();

    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }

    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }

    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```

<br/>

### AbstractCollection
Collection 인터페이스의 일부 메서드를 구현한다. 

```java
public abstract class AbstractCollection<E> implements Collection<E> {

    protected AbstractCollection() {
    }

    // Query Operations

    public abstract Iterator<E> iterator();

    public abstract int size();

    public boolean isEmpty() {
        return size() == 0;
    }

    // 특정 요소를 포함하고 있는지 확인하는 메서드로, 순차탐색하여 일치하는 값을 찾는다.
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext())
                if (it.next()==null)
                    return true;
        } else {
            while (it.hasNext())
                if (o.equals(it.next()))
                    return true;
        }
        return false;
    }

    // 배열로 변환해주는 메서드로, size 크기의 배열을 생성 후 순차적으로 값을 복사해준다.
    public Object[] toArray() {
        // Estimate size of array; be prepared to see more or fewer elements
        Object[] r = new Object[size()];
        Iterator<E> it = iterator();
        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) // fewer elements than expected
                return Arrays.copyOf(r, i);
            r[i] = it.next();
        }
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        // Estimate size of array; be prepared to see more or fewer elements
        int size = size();
        T[] r = a.length >= size ? a :
                  (T[])java.lang.reflect.Array
                  .newInstance(a.getClass().getComponentType(), size);
        Iterator<E> it = iterator();

        for (int i = 0; i < r.length; i++) {
            if (! it.hasNext()) { // fewer elements than expected
                if (a == r) {
                    r[i] = null; // null-terminate
                } else if (a.length < i) {
                    return Arrays.copyOf(r, i);
                } else {
                    System.arraycopy(r, 0, a, 0, i);
                    if (a.length > i) {
                        a[i] = null;
                    }
                }
                return a;
            }
            r[i] = (T)it.next();
        }
        // more elements than expected
        return it.hasNext() ? finishToArray(r, it) : r;
    }

    @SuppressWarnings("unchecked")
    private static <T> T[] finishToArray(T[] r, Iterator<?> it) {
        int len = r.length;
        int i = len;
        while (it.hasNext()) {
            if (i == len) {
                len = ArraysSupport.newLength(len,
                        1,             /* minimum growth */
                        (len >> 1) + 1 /* preferred growth */);
                r = Arrays.copyOf(r, len);
            }
            r[i++] = (T)it.next();
        }
        // trim if overallocated
        return (i == len) ? r : Arrays.copyOf(r, i);
    }

    // Modification Operations

    public boolean add(E e) {
        throw new UnsupportedOperationException();
    }

    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }


    // Bulk Operations

    public boolean containsAll(Collection<?> c) {
        for (Object e : c)
            if (!contains(e))
                return false;
        return true;
    }

    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<?> it = iterator();
        while (it.hasNext()) {
            if (c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    // 교집합
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            if (!c.contains(it.next())) {
                it.remove();
                modified = true;
            }
        }
        return modified;
    }

    // clear() 함수는 순차적으로 접근하여 요소들을 지우므로 O(n)
    public void clear() {
        Iterator<E> it = iterator();
        while (it.hasNext()) {
            it.next();
            it.remove();
        }
    }


    //  String conversion

    public String toString() {
        Iterator<E> it = iterator();
        if (! it.hasNext())
            return "[]";

        StringBuilder sb = new StringBuilder();
        sb.append('[');
        for (;;) {
            E e = it.next();
            sb.append(e == this ? "(this Collection)" : e);
            if (! it.hasNext())
                return sb.append(']').toString();
            sb.append(',').append(' ');
        }
    }

}
```
<br/>

## List
List는 순서가 있는 데이터의 집합을 다룰 때 사용된다. index를 제공하므로 index를 인자로하는 메서드 및 ListIterator를 제공한다. <br/>
```java
public interface List<E> extends Collection<E> {
    // Query Operations

                                        .
                                        .
                                        .


    // Modification Operations

                                        .
                                        .
                                        .



    // Bulk Modification Operations

                                        .
                                        .
                                        .


    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }

    // sort 함수를 보면 Arrays.sort()를 사용 후, iterator를 다시 설정해주고있다.
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }

    // Comparison and hashing

                                        .
                                        .
                                        .


    // Positional Access Operations

    E get(int index);

    E set(int index, E element);

    void add(int index, E element);

    E remove(int index);


    // Search Operations

    int indexOf(Object o);

    int lastIndexOf(Object o);


    // List Iterators

    ListIterator<E> listIterator();

    ListIterator<E> listIterator(int index);

    // View

    List<E> subList(int fromIndex, int toIndex);

    @Override
    default Spliterator<E> spliterator() {
        if (this instanceof RandomAccess) {
            return new AbstractList.RandomAccessSpliterator<>(this);
        } else {
            return Spliterators.spliterator(this, Spliterator.ORDERED);
        }
    }

    @SuppressWarnings("unchecked")
    static <E> List<E> of() {
        return (List<E>) ImmutableCollections.EMPTY_LIST;
    }

    static <E> List<E> of(E e1) {
        return new ImmutableCollections.List12<>(e1);
    }

    static <E> List<E> of(E e1, E e2) {
        return new ImmutableCollections.List12<>(e1, e2);
    }
                                        .
                                        .
                                        .

    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
        return ImmutableCollections.listFromTrustedArray(e1, e2, e3, e4, e5,
                                                         e6, e7, e8, e9, e10);
    }

    @SafeVarargs
    @SuppressWarnings("varargs")
    static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                @SuppressWarnings("unchecked")
                var list = (List<E>) ImmutableCollections.EMPTY_LIST;
                return list;
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return ImmutableCollections.listFromArray(elements);
        }
    }

    static <E> List<E> copyOf(Collection<? extends E> coll) {
        return ImmutableCollections.listCopy(coll);
    }
}
```
- ListIterator는 Iterator를 상속하고 리스트를 양방향으로 순회할 수 있게 하며, 순회 중에 리스트를 수정할 수 있다. 또한, 리스트에서 iterator의 현재 위치를 얻을 수 있다.

<br/>

### AbstractList
AbstractList는 List 인터페이스를 부분적으로 구현하여 리스트의 기본 동작을 제공. List 인터페이스를 구현하는 서브클래스들의 반복적인 코드 작성을 피하게 한다. 
=> 클래스 내부를 확인하였을때 매개변수로 받은 index를 직접적으로 사용하는 get 멤버 메서드들은 구현하지 않았다. set/add/remove 또한 예외를 발생하도록만 구현되어있다. 대신 나머지 메서드들이 get/set/add/remove를 사용하여 동작하는 것으로 구현되어있다. 

<br/>

### ArrayList
ArrayList는 동적 배열을 구현한 클래스이다. ArrayList는 내부적으로 elementData라는 배열 타입의 멤버필드를 기반으로 하며, iterator()와 listIterator() 메서드 또한 호출 시 elementData를 기반으로 동작하는 객체를 생성해서 반환한다. 
동적 배열을 구현하는 방식은 현재 capacity가 부족한 경우, 새로운 capacity의 배열을 생성하고 기존 요소들을 복사하여 저장한다.

```java
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */); // 해당 함수에서 대략적으로 oldCapacity + (oldCapacity >> 1)(즉, 기존 크기 + 기존 크기의 절반) 방식으로 계산되어 확장된다.
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
```

### Vector
Vector는 ArrayList와 마찬가지로 동적 배열을 제공하는 클래스이다. 기본적으로 비슷한 기능을 제공하지만 Vector는 연산 메서드에 synchronized 키워드를 사용함으로써 멀티스레드 환경에서 동기화를 제공한다. 또한 elementData 배열의 capacity가 ArrayList에서는 데이터가 다 차면 1.5배씩 증가한 반면 Vector에서는 2배씩 증가한다는 점에서 차이가 난다.

<br/>

### Stack
Stack은 Vector 클래스를 상속하면서 stack 자료구조의 기본연산인 push/pop/peek을 구현한다. 구현한 연산들은 부모 클래스인 Vector 클래스의 함수를 사용하게 되므로, 멀티스레드 환경에서 동기화를 제공한다.

```java
public class Stack<E> extends Vector<E> {

    public Stack() {
    }

    public E push(E item) {
        addElement(item);

        return item;
    }

    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }

    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }

    public boolean empty() {
        return size() == 0;
    }

    public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }

    @java.io.Serial
    private static final long serialVersionUID = 1224463164541339165L;
}
```

<br/>

### AbstractSequentialList
AbstractSequentialList는 AbstractList의 하위 클래스이며, get/set/add/remove 메서드를 listIterator를 사용하여 구현함으로써 순차적인 접근으로 각 연산을 제공한다.(특이한 점은 LinkedList는 해당 메서드들을 오버라이딩하여 사용하고 있다.)
```java
public abstract class AbstractSequentialList<E> extends AbstractList<E> {
    protected AbstractSequentialList() {
    }

    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public E set(int index, E element) {
        try {
            ListIterator<E> e = listIterator(index);
            E oldVal = e.next();
            e.set(element);
            return oldVal;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public void add(int index, E element) {
        try {
            listIterator(index).add(element);
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }

    public E remove(int index) {
        try {
            ListIterator<E> e = listIterator(index);
            E outCast = e.next();
            e.remove();
            return outCast;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }


    // Bulk Operations

    public boolean addAll(int index, Collection<? extends E> c) {
        try {
            boolean modified = false;
            ListIterator<E> e1 = listIterator(index);
            for (E e : c) {
                e1.add(e);
                modified = true;
            }
            return modified;
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }


    // Iterators

    public Iterator<E> iterator() {
        return listIterator();
    }

    public abstract ListIterator<E> listIterator(int index);
}
```

<br/>

### LinkedList
LinkedList는 AbstractSequentialList를 상속하고 Deque를 구현한다. 따라서 양방향에서 큐연산이 가능하며, 데이터의 순차접근이 가능하다.
데이터는 Node<E> 클래스를 사용하여 저장하며, 멤버필드로 first node와 last node를 가진다. 즉, 연결리스트로써 처음과 마지막에 삽입과 삭제 시에는 O(1)의 시간 복잡도를 가지지만 특정 노드를 찾으로면 노드 하나하나를 순차 접근하여야하므로 O(n) 시간 복잡도를 가지게 된다. 
```java
/**
* Pointer to first node.
*/
transient Node<E> first;

/**
* Pointer to last node.
*/
transient Node<E> last;

                                        .
                                        .
                                        .

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
```

<br/>
<br/>

## Queue
Queue 인터페이스는 add(E e), offer(E e), remove(), poll(), element(), peek() 함수를 정의한다. 
여기서 add(E e), remove(), element() 함수는 queue가 비거나 다 찼을 경우, 예외를 발생시킨다. 이에 대응하는 offer(E e), poll(), peek() 함수는 false 또는 null을 반환한다. 
```java
public interface Queue<E> extends Collection<E> {
    // Returns: true (as specified by Collection.add)
    // IllegalStateException if no space is currently available.
    boolean add(E e);

    // Returns: true if the element was added to this queue, else false
    boolean offer(E e);

    // it throws an exception if this queue is empty.
    // Returns: the head of this queue
    E remove();

    // Returns: the head of this queue, or null if this queue is empty
    E poll();

    // throws an exception if this queue is empty.
    // Returns: the head of this queue
    E element();

    // Returns: the head of this queue, or null if this queue is empty
    E peek();
}
```
<br/>

### Deque
Deque는 양방향 삽입 삭제가 가능한 자료구조 deque의 기능을 정의한다. 
```java
public interface Deque<E> extends Queue<E> {

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

    // *** Queue methods ***

    boolean add(E e);
    boolean offer(E e);
    E remove();
    E poll();
    E element();
    E peek();
    boolean addAll(Collection<? extends E> c);

    // *** Stack methods ***
    void push(E e);
    E pop();

    // *** Collection methods ***

    boolean remove(Object o);
    boolean contains(Object o);
    int size();
    Iterator<E> iterator();
    Iterator<E> descendingIterator();

}
```
<br/>

### ArrayDeque
ArrayDeque는 동적배열을 사용하여 Deque를 구현하는 클래스이다. ArrayDeque가 스택으로 사용할 때 Stack 클래스보다 더 빠르고, 큐로 사용할 때 LinkedList보다 더 빠를 수 있다.

**1. Stack 클래스와의 비교**
- Stack 클래스는 **동기화(synchronization)**가 포함된 자료 구조다. 반면, ArrayDeque는 동기화를 지원하지 않으므로 스택처럼 사용할 때 동기화에 따른 성능 저하가 발생하지 않아 더 빠르게 동작할 수 있다.

<pre>
Stack: 동기화로 인해 성능이 느려질 수 있음. (index 접근이 가능. O(1))
ArrayDeque: 동기화가 없기 때문에 빠른 연산 가능. (index 접근이 불가능.)
</pre>

<br/>

**2. LinkedList와의 비교**
- LinkedList는 이중 연결 리스트(doubly linked list)로 구현되어 있다. 이 때문에 요소를 큐의 앞이나 뒤에 삽입하거나 제거하는 연산은 O(1) 시간 복잡도를 갖지만, 메모리 관점에서 비효율적일 수 있다. 각각의 요소는 데이터와 함께 포인터를 포함해야 하므로 추가적인 메모리 사용이 필요하며, 연결 리스트에서 요소에 접근할 때는 노드를 순차적으로 따라가야 O(n) 시간 복잡도를 가진다.
- ArrayDeque는 내부적으로 배열을 사용하고, 배열은 메모리 상에 연속적으로 저장되므로 캐시 적중률(cache hit rate)이 높다. 요소를 순환큐의 head/tail에 추가하거나 제거할 때는 O(1) 시간 복잡도를 가지며, 배열의 크기가 초과될 때만 배열을 재할당하는 과정이 발생한다.

<pre>
LinkedList: 이중 연결 리스트 구조로 인해 추가적인 메모리 소비와 상대적으로 낮은 캐시 효율성. (get/set 메서드 구현. O(n))
ArrayDeque: 배열 기반으로 더 나은 캐시 성능과 연산 속도. (index 접근 불가능.)
</pre>

<br/>

ArrayDeque는 내부적으로 head, tail 필드를 사용해 순환큐를 구현한다. 순환큐이기 때문에 배열을 사용하여 구현한다고 하더라도 index를 사용한 접근이 불가능하다. 따라서, get/set과 같은 메서드는 구현하지 않고 있으며, 오로지 head와 tail 위치에 요소를 추가하거나 삭제하는 offerFirst/offerLast, pollFirst/pollLast 또는 peekFirst/peekLast 등의 함수만 제공한다.
```java
    transient Object[] elements;

    /**
     * The index of the element at the head of the deque (which is the
     * element that would be removed by remove() or pop()); or an
     * arbitrary number 0 <= head < elements.length equal to tail if
     * the deque is empty.
     */
    transient int head;

    /**
     * The index at which the next element would be added to the tail
     * of the deque (via addLast(E), add(E), or push(E));
     * elements[tail] is always null.
     */
    transient int tail;

```

<br/>

### AbstractQueue
AbstractQueue는 Queue 인터페이스의 일부 메서드를 구현한다. add(E e), remove(), element() 메서드 내에서 각각 offer(E e), poll(), peek() 메서드를 사용하며 반환값에 따라 예외를 발생시키는 것을 볼 수 있다. 
```java
public abstract class AbstractQueue<E>
    extends AbstractCollection<E>
    implements Queue<E> {

    protected AbstractQueue() {
    }

    public boolean add(E e) {
        if (offer(e))
            return true;
        else
            throw new IllegalStateException("Queue full");
    }

    public E remove() {
        E x = poll();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    public E element() {
        E x = peek();
        if (x != null)
            return x;
        else
            throw new NoSuchElementException();
    }

    public void clear() {
        while (poll() != null)
            ;
    }

    public boolean addAll(Collection<? extends E> c) {
        if (c == null)
            throw new NullPointerException();
        if (c == this)
            throw new IllegalArgumentException();
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

}
```
<br/>

### PriorityQueue
PriorityQueue는 내부적으로 동적배열 역할을 하는 queue 멤버필드와 siftUp(int k, E x), siftDown(int k, E x) 메서드를 구현하여 heap 자료구조를 지원하는 클래스이다. 
또한, 요소들간 비교를 위해 comparator 멤버필드를 가지며 comparator가 초기화되지 않은 경우, 비교 대상이 되는 Element는 Comparlabe을 구현하여야한다.
```java
/**
* Priority queue represented as a balanced binary heap: the two
* children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
* priority queue is ordered by comparator, or by the elements'
* natural ordering, if comparator is null: For each node n in the
* heap and each descendant d of n, n <= d.  The element with the
* lowest value is in queue[0], assuming the queue is nonempty.
*/
transient Object[] queue; // non-private to simplify nested class access

/**
* The comparator, or null if priority queue uses elements'
* natural ordering.
*/
@SuppressWarnings("serial") // Conditionally serializable
private final Comparator<? super E> comparator;
```

