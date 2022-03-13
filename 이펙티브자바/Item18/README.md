# Item18, 상속 보다는 컴포지션을

상속은 코드를 재사용하는 강력한 수단과 동시에 오류를 내기 쉬운 소프트웨어를 만들 가능성을 존재하게한다. 상위클래스가 변경됨에 따라 하위 클래스의 동작에 문제가 생길수있다. 여기서 상속이 캡슐화를 깨트리는 이유이다.(다른 패키지의 구체 클래스를 상속하는 것은 위험하다)

자식클래스는 부모클래스에 의존하기 때문에 부모클래스의 동작을 변경함에 따라 하위클래스가 오동작을 할 수 있다. 이렇게 상위클래스 설계자가 확장을 충분히 고려하고 문서화를 안하면 변경하지 않은 하위 클래스도 변경해야 한다.

Item1, 정적 팩토리 메소드의 단점으로 언급된 public과 protected가 아니기에 상속할 수 없다가 있었는데 오히려 컴포지션을 유도하기에 조금 더 안전한 클래스를 만들어주는 장점이 될수도 있다고 생각한다.

## 하위 클래스가 영향을 받는다.

**상속은 캡슐화를 깨뜨린다.**

상위클래스는 릴리스마다 내부 구현이 달라질 수 있으며, 그 여파로 하위 클래스가 오동작할 수 있다.

### a. HashSet의 add와 addAll

```java
public class InstrumentedHashSet<E> extends HashSet<E>{
  private int addCount = 0;

  @Overrid public boolean add(E e){
    addCount++;
    return super.add(e);
  }

  @Overrid public boolean addAll(Collection<? extends E> c){
    addCount += c.size();
    return super.add(c);
  }
}
```

잘 작동하지 않는다 : addAll 메서드가 add를 사용해 구현되어 addCount 값이 중복해서 더해진다.

내부 구현 방식에 해당하는 부분을 고려해서 수정을 해도, 이 내부구현이 다음 릴리즈에서 유지될지 알 수 없다.

### b. 상위클래스의 메서드 동작을 재구현

addAll 메서드를 다른식으로 재정의한다 : 주어진 컬렉션을 순회하며 원소 하나당 add 메서드를 한번만 호출 시간도 더들고, 오류를 내거나 성능저하가 있을 수 있다. 하위클래스에서 접근이 불가한 private이라면 이 방식 구현 자체가 불가능하다.

### c. 만약 다음 릴리즈에서 상위 클래스에 새로운 메서드 추가?

상위클래스에 또다른 원소 추가 메서드가 생성된다 가정하자. 하위 클래스에서 재정의하지 못한 새로운 메서드를 사용해 허용되지 않은 원소를 추가할 수 있게된다.

### d. 클래스 확장하더라도 메서드 재정의 대신 새로운 메서드 추가

운이 없게도, 다음 릴리즈에서 상위클래스에 추가된 새 메서드가 내가 하위클래스에 추가한 메서드와 시그니처가 같고 반환타입이 다르면 컴파일조차 안된다.

## 상속의 문제를 해결하는 방법 : 컴포지션

**컴포지션 설계** : 기존 클래스가 새로운 클래스의 구성요소로 쓰인다.기존 클래스 확장 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하자

**전달** : 새 클래스의 인스턴스 메서드는 기존 클래스에 대응하는 메서드들을 호출해 그 결과를 반환한다.

**전달 메서드** : 새 클래스의 메서드

```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

**래퍼클래스 :** 다른 인스턴스를 감싸고 있다 `InstrumentedSet` : **데코레이터 패턴**

**위임(deleagtion)** : 컴포지션과, 전달의 조합 : 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우

## 래퍼클래스의 단점

콜백(callback) 프레임워크와는 어울리지 않는다.

SELF 문제 : 콜백에서는 자기 자신의 참조를 다른 객체에게 넘겨서 다음 호출때 사용한다. 내부 객체는 자신을 감싸는 래퍼의 존재를 모르니 대신 자신(this) 참조를 넘기고, 콜백때 래퍼 객체가 아닌 내부 객체를 호출한다.

전달 메서드 작성이 귀찮으면 : 재사용할 수 있는 전달클래스 만들기

# 결론

상속은 확장을 위해 편리한 기능이지만 캡슐화를 해친다. 상위 클래스와 하위 클래스의 관계가 순수한 is -a 관계 일때만 써야한다. is - a 관계 이더라도 확장을 고려해서 상위클래스가 설계되지 않았다면 문제를 일으킬수있다. 

컴포지션을 사용해야할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 것이다

API가 내부 구현에 묶이고 클래스의 성능도 영원히 제한된다.

**상속을 결정하는 질문**

- 확장하는 클래스의 API에 아무런 결함이 없는가?
- 결함이 있다면 이 결함이 내 클래스의 API까지 전파되도 괜찮은가?: 컴포지션으로는 이런 결함을 숨길 수 있지만, 상속은 아니다.