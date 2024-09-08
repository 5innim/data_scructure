## Stack

스택이란 FILO(First In Last Out)방식의 데이터 삽입/출력 방식을 가지는 자료구조이다. 

### 기본 연산
- Push: 스택의 맨 위에 데이터를 추가하는 연산.
- Pop: 스택의 맨 위에 있는 데이터를 제거하고 반환하는 연산.
- Peek(Top): 스택의 맨 위에 있는 데이터를 제거하지 않고 확인하는 연산.
### 구현

먼저 java.util에서 제공하는 Collection 프레임워크의 Stack 클래스를 보자. 
```java
/*
 * Copyright (c) 1994, 2021, Oracle and/or its affiliates. All rights reserved.
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

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = 1224463164541339165L;
}
```
Stack 클래스는 Vector 클래스를 상속한다. 따라서 내부에서 구현하고 있는 함수도 부모 클래스에서
구현하고 있는 함수를 사용하는데 주요한 push, pop, peek은 모두 synchronized 키워드를 사용하게 된다. 

따라서 연산 시 멀티 스레드에 대한 동기화 작업이 수행되므로 코딩 테스트처럼 주로 하나의 스레드만 사용하는 경우, 비효율적이다. 




## Queue
