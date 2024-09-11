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
