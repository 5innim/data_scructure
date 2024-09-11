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

## Collections(Set)
<img width="765" alt="스크린샷 2024-09-10 오후 10 55 16" src="https://github.com/user-attachments/assets/2761bace-5bdd-41ff-bbc4-4d0a09e2437d">

<br/>
<br/>

### Set
Set 인터페이스는 Collection 인터페이스를 상속하며, set 자료구조의 기본 동작을 정의한다. List 인터페이스와 다르게 순서를 보장하지 않으니 index 매개변수를 받는 메서드는 정의되어있지않다. 

<br/>

### AbstractSet
AbstractSet은 Set 인터페이스를 구현하기 위한 기본적인 구현을 제공한다. 
```java
public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E> {

    protected AbstractSet() {
    }

    // Comparison and hashing

    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Set))
            return false;
        Collection<?> c = (Collection<?>) o;
        if (c.size() != size())
            return false;
        try {
            return containsAll(c);
        } catch (ClassCastException | NullPointerException unused) {
            return false;
        }
    }

    public int hashCode() {
        int h = 0;
        Iterator<E> i = iterator();
        while (i.hasNext()) {
            E obj = i.next();
            if (obj != null)
                h += obj.hashCode();
        }
        return h;
    }

    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        boolean modified = false;

        if (size() > c.size()) {
            for (Object e : c)
                modified |= remove(e);
        } else {
            for (Iterator<?> i = iterator(); i.hasNext(); ) {
                if (c.contains(i.next())) {
                    i.remove();
                    modified = true;
                }
            }
        }
        return modified;
    }

}
```

<br/>

### HashSet
HashSet은 HashMap을 기반으로 set 자료구조를 구현하는 클래스이다. 정확히는 요소를 HashMap의 key 위치에 저장한다. 따라서 put(key, value) 함수를 사용할때 쓸 dummy value 역할로 PRESENT라는 상수를 멤버필드로 가진다. 당연히 저장된 요소들의 순서는 보장되지 않는다.
```java
public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable {

                                        .
                                        .
                                        .

    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

                                        .
                                        .
                                        .

    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }

                                        .
                                        .
                                        .

}
```

