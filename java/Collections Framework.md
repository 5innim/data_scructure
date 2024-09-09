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

## List & Queue
<img width="787" alt="스크린샷 2024-09-08 오후 11 38 51" src="https://github.com/user-attachments/assets/12be4616-745b-4d34-bd77-b0417bfdc4ce">

List와 Queue의 상속 구조는 위 사진처럼 복잡하다. 

순서대로 하나씩 알아보자. 

<br/>

### Iterable
Iterator는 자바 컬렉션에서 요소들을 순차적으로 접근하고 처리하기 위한 인터페이스이다.
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



