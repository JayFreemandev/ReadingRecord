# Item 79, 과도한 동기화는 피하자

# 정리

- 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.  
- 예를들어, 동기화된 영역 안에서는 재정의할 수 있는 메서드를 호출하면 안되고 클라이언트가 넘겨준 함수 객체를 호출해서도 안된다.  
- 이러한 애들은 외계인이라고 생각하면 된다. 즉, 메서드가 무엇을 할지 전혀 모르기 때문이다.  
- 따라서 위에서 말한것 처럼 교착상태나, 예외, 데이터를 훼손할수 있다.  

## **동기화(Synchronize)**

"여러개의 쓰레드가 한 개의 리소스를 사용하려고 할 때 ,   
사용 하려는 쓰레드를 제외한 나머지들이 접근하지 못하게 막는것"   

→ Thread-safe

동기화 방법

- 메소드 자체 Synchronized로 선언하는 방법  
- 블록으로 객체를 받아 lock을 거는 방법 - syncrhonized(this)  

### **멀티스레드 프로세스는 동시성 또는 병렬성으로 실행**

동시성 : 하나의 코어에서 여러개의 프로세스가 번갈아 가면서 실행  
병렬성 : 멀티 코어에서 개별 스레드를 동시에 실행  

아이템 18, 상속보다는 컴포지션을. 김세윤님의 예제 코드를 들고왔다.

**Observer pattern**으로 구현한 Set 코드인데 Set에 원소가 추가되면 알림

을 받을 수 있는 방법이다. 

**Observer pattern:** 객체의 상태 변화를 관찰하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다   
메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴이다.   
주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. 발행/구독 모델로 알려져 있기도 하다.  

```java
import java.util.*;

public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public void clear() {
        s.clear();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public boolean isEmpty() {
        return s.isEmpty();
    }

    public int size() {
        return s.size();
    }

    public Iterator<E> iterator() {
        return s.iterator();
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean remove(Object o) {
        return s.remove(o);
    }

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}
```

```java
public interface SetObserver<E> {
    void added(ObservableSet<E> set, E element);
}
```

```java
import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

		private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override 
		public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override 
		public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
```

- 여기서 중요한 메서드는 addObserver와 removeObserver, notifyElementAdded 이다.
- addObserver와 removeObserver 메소드는 콜백 SetObserver 함수형 인터페이스를 받는다.
- 그럼 0 ~ 99까지 출력하는 코드를 작성해보자.

```java
public class Test1 {
    public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver((s, e) -> System.out.println(e));

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
}
```

정상적으로 99까지 출력하는데 여기서 값이 23이라면 구독을 해지하는 옵저버를 추가해보자 

if(element = 23) → set.remover 이렇게 말이다.

```java
public static void main(String[] args) {
        ObservableSet<Integer> set =
                new ObservableSet<>(new HashSet<>());

        set.addObserver(new SetObserver<Integer>() {
            @Override
            public void added(ObservableSet<Integer> set, Integer element) {
                System.out.println(element);
                if  (element == 23) {
                    set.removeObserver(this);
                }
            }
        });

        for (int i = 0; i < 100; i++)
            set.add(i);
    }
```

결과는 `ConcurrentModificationException 발생`

```java
...
23
Exception in thread "main" java.util.ConcurrentModificationException
```

- 23까지 출력하지만, 그 이후에는 위와 같이 예외를 발생시키게 된다.
- 그 이유는 added 메서드가 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다. 즉, added 메서드는 ObservableSet의 removeObserver 메서드를 호출하고, 이 메서드는 다시 observers.remove 메서드를 호출한다.
- 이때 원소를 제거하려고 하는데, notifyElementAdded 메서드에서 리스트를 순회하는 도중이기 떄문에 허용되지 않은 동작이며, 동기화 블록안에 있어 동시 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지는 막지 못한다.
- 이번에도 이상한 시도를 해보자. 구독해지를 하는 removeObserver를 직접 호출하지 않고 다른 스레드에게 맡겨보는 것이다.(쓸데없는 백그라운드 스레드를 사용하는 것이다.)

다른 스레드에 맡기니 교착상태가 발생했다. 메인 스레드가 이미 락을 쥐고있기 때문이다. 위의 예제는 실상황에서 멀지만 실제 시스템 GUI 툴킷에도 동기화된 영역안에 저런 외계인 메서드 호출해버리면 교착상태 빠지는 경우가 존재한다.

## 동기화와 성능

자바의 동기화 비용은 빠르게 낮아져 왔지만, 과도한 동기화를 피하는일은 오히려 과거 어느 때보다 중요하다.

멀티코어가 일반화된 오늘날 과도한 동기화가 초래하는 **진짜 비용은 락을 얻는데 드는 CPU 시간이 아니다.**

서로 스레드끼리 경쟁하는 Race Condition에 낭비가 발생한다.

- 병렬로 실행할 기회를 잃는다.
- 모든 코어가 메모리를 일관되게 보기위한 지연시간이 진짜 비용
- 가상머신의 코드최적화를 제한하는 점도 숨은 비용

## 해**결 방법은 무엇인가?**

- 외계인 메서드 호출을 동기화 블록 바깥으로 옮겨버리면 된다.
- 바로 아래 코드처럼 말이다.
- 이러한 코드를 열린 호출이라고 한다. 얼마나 오래 실행될지 알수 없는데, 동기화 영역 안에서 실행이 된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기만 해야 하지만, 아래와 같은 코드는 실패 방지 효과외에도 동시성 효율을 크게 개선해주는 것이다.

```java
private void notifyElementAdded(E element) {
        List<SetObserver<E>> snapshot = null;
        synchronized (observers) {
            snapshot = new ArrayList<>(observers);
        }
        for (SetObserver<E> observer : snapshot)
            observer.added(this, element);
    }
```

- 또한 이것보다 더 좋은 방안은 CopyOnWriteArrayList를 사용하는 것이다. 이 List가 나온 이유는 위와 같은 문제를 해결하기 위해서이다.
- 즉, 아래와 같이 변경하면 되는것이다.

```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
for (SetObserver<E> observer : observers)
		observer.added(this, element);
```

### **그럼 어떻게 해결해야 할까?**

- 가변 클래스일때 두 가지 선택지 중 하나를 선택하자.
    1. 동기화를 하지 않고 외부에서 알아서 동기화 하게 하자
    2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자
- 2번째를 선택해야 하는 경우는 외부에서 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 사용하자.
- 그러나 2번을 선택해야 한다면 아래와 같은 기법으로 해결해봐라.
    - 락 분할
    - 락 스트라이핑
    - 비차단 동시성 제어

자바에서는..

- 자바도 SpringBuffer가 초창기에 내부적으로 동기화를 수행했다. 그래서 StringBuffer가 나오게 되었다.
- 쓰레드 안전한 Random도 동기화하지 않은 버전인 ThreadLocalRandom으로 대체되었다.

## **정리**

- 기본 규칙은 동기화 영역에서 가능한 한 일을 적게하는 것
- 오래 걸리는 작업이라면 동기화 영역 밖으로 옮기는 방법을 찾아보자.
- 여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화 해야 한다.
- 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자
- 동기화 영역 안에서 작업은 최소한으로 줄이자
