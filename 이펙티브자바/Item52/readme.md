## **아이템52. 다중정의는 신중히 사용하라**

**다중정의(오버로딩)란?**

- 서로 다른 시그니처를 갖는 여러 메소드를 같은 이름으로 정의하는 것
    - 시그니처가 같다? -> 메서드명, 매개변수의 개수, 타입, 순서까지 모두 같다
- 즉, 메서드명은 같지만 매개변수의 개수, 타입, 순서 중 하나라도 다른 메서드를 선언하는 것

---
<br>

**오버로딩과 오버라이딩은 어떤 메서드가 실행될지 결정되는 시점이 다르다**

- 오버로딩에선 어떤 메서드가 실행될지는 컴파일 타임에 정해진다.
    
    ```java
    public class CollectionClassifier {
        public static String classify(Set<?> s) {
            return "집합";
        }
    
        public static String classify(List<?> lst) {
            return "리스트";
        }
    
        public static String classify(Collection<?> c) {
            return "그 외";
        }
    }
    ```
    
    ```java
    Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String, String>().values()
    };
    
    for (Collection<?> c : collections) {
        System.out.println(CollectionClassifier.classify(c));
    }
    //그 외
    //그 외
    //그 외
    ```
    
- 오버라이딩에선 어떤 메서드가 실행될지는 런타임에 정해진다.

```java
`public class 동물 {
    String name() {
        return "동물";
    }
}`

`public class 포유류 extends 동물 {
    @Override
    String name() {
        return "포유류";
    }
}`

`public class 인간 extends 포유류 {
    @Override
    String name() {
        return "인간";
    }
}`

`List<동물> animals = Arrays.asList(new 동물(), new 포유류(), new 인간());

for (동물 animal : animals) {
    System.out.println(animal.name());
}
//동물
//포유류
//인간`
```
<br>

**웬만하면 매개변수 개수가 같은 오버로딩은 하지 말자**

- 매개변수 개수가 같아야 한다면 오버로딩 대신 메서드 이름을 다르게 지어주는게 좋다

---

**매개변수 개수가 같은 오버로딩을 피할 수 없다면?**

- 매개변수 중 하나 이상이 근본적으로 다르면(서로 어느쪽으로든 형변환 할 수 없으면) 그나마 괜찮다
    - ex) ArrayList의 인자가 1개인 생성자는 ArrayList(int initialCapacity)와 ArrayList(Collection<? extends E> c)가 있지만 int와 Collection은 근본적으로 다르므로 괜찮다

---

**하지만 매개변수가 근본적으로 달라도 항상 안전하진 않다**

- List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
list.remove(3); //3을 지우라는 것일까? index 3번째 원소를 지우라는 것일까?
- 혼란스러운 이유는 List<E>인터페이스가 remove(int index) 와 remove(Object o) 를 오버로딩 했기 때문이다.
- 제네릭과 오토박싱이 등장하면서 int를 Integer로 자동 변환 해주니까 근본적으로 달라지지 않게 되었다.

---

**오버로딩할때 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다**

- new Thread(System.out::println).start(); //컴파일 성공
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println); //컴파일 에러
- 넘겨진 인수는 모두 System.out::println 로 똑같고, 둘 다 Runnable을 받는 형제 메서드를 오버로딩 하고 있다
- 하지만 아래만 컴파일 에러가 발생한다.
- 정확한 이유는 모르겠으나 서로 다른 함수형 인터페이스라도 인수 위치가 같으면 혼란이 발생한다.(사실 함수형 인터페이스들은 근본적으로 같다.)

---

### **결론**

매개변수 수가 같을 땐 오버로딩을 피하는 게 좋다.
