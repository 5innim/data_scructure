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

## Collections(Map)
<img width="656" alt="스크린샷 2024-09-11 오전 11 52 20" src="https://github.com/user-attachments/assets/4954efde-2e45-45ad-a167-61a0c9802ff1">

<br/>
<br/>

## Map
key, value 쌍의 데이터를 저장할 수 있는 map 자료구조를 위한 메서드를 정의하는 인터페이스, key가 중복되는 데이터는 저장할 수 없다. Key 하나 당 하나의 value를 저장할 수 있다.

<br/>

### HashMap
HashMap은 key, value 쌍의 데이터를 저장하며, key에 대한 hash 값을 기반으로 index를 계산하여 Node[] 배열에 저장한다. 그렇기에 특정 데이터에 접근 시 O(1) 시간 복잡도를 가진다.(아래에서 설명하겠지만 항상 그렇지는 않다.)
- 배열의 길이가 n 일때 (n-1) & hash 를 index로 사용한다.(n 값은 항상 2의 제곱수인것이 보장된다.) 따라서 hash의 크기가 크다고 하더라도 Index out of range와 같은 예외는 발생하지 않는다.
- HashMap은 해시충돌의 경우, 해당 index에 연결리스트 또는 Red-Black Tree(이진 탐색 트리의 한 종류)를 버킷으로 사용하여 같은 index에 여러 데이터를 저장하고 관리한다. 이때, 해당 index의 버킷에 m개의 데이터가 있다면 O(m) 또는 O(log(m))이 추가 발생하게 된다. 추가로, HashMap은 버킷의 항목 수가 8개 이상이 되면 Red-Black Tree로 전환하여 관리하고, 6개 이하가 되면 다시 연결 리스트로 전환한다.
```java
    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;
```
```java
    // Java HashMap의 내부에서 key에 해당하는 Node<K,V>를 찾는 메서드로 getNode 전반적인 HashMap의 동작을 볼 수 있다.
    final Node<K,V> getNode(Object key) {

        Node<K,V>[] tab; // 해시맵에서 Node<K,V>를 저장하는 배열
        Node<K,V> first, e; // firtst: key에 해당하는 index 위치의 Node, e: 현재 탐색 중인 노드
        int n, hash; // n: 배열 크기, hash: 주어진 key의 hash 값.
        K k; // 탐색 중인 노드의 키.

        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & (hash = hash(key))]) != null) { // 배열이 비었는지와 key에 해당하는 index 위치의 값이 비었는지 확인.
            if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) // key의 hash 값이 같은지 && (key의 동일성||동등성) 확인.
                return first;
            if ((e = first.next) != null) { // 해당 버킷에 다른 값이 있다면
                if (first instanceof HashMap.TreeNode) // 버킷이 이진 탐색 트리인 경우, O(log(n)) 
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do { // 버킷이 연결 리스트인 경우, 아래와 같이 순차 탐색하게 된다.
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);// 32bit h의 상위 비트와 하위비트를 섞는 연산. 원래의 상위 16bit + (상위 16bit xor 하위 16bit)
                                                                        // 이렇게 하면 해시맵에서 키의 해시 코드가 보다 균등하게 분포되며, 해시 충돌을 줄이는 효과가 있다.
    }
```
- 버킷(bucket): 해시 충돌을 처리하기 위한 저장 공간으로, 해시 값을 기반으로 결정된 배열의 각 인덱스에 위치하는 저장 공간을 의미한다. 


