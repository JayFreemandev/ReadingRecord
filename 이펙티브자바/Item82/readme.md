# Item 82, 스레드 안정성 수준을 문서화하라

# 정리

- 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다.
- 정확한 언어로 명확히 설명하거나 스레드 안전성 애너테이션을 사용할 수 있다.
- synchronized 한정자는 문서화와 관련이 없다.

### 

- 한 메서드를 여러 스레드가 동시에 호출할 때 그 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약과 같다.
- API 문서에서 아무런 언급도 없으면 그 클래스 사용자는 나름의 가정을 해야만 한다.
- 만약 그 가정이 틀리면 클라이언트 프로그램은 동기화를 충분히 하지 못하거나(아이템 78), 지나치게 한(아이템 79) 상태일 것이며, 두 경우 모두 심각한 오류로 이어질 수 있다.
- 메서드 선언에 synchronized 한정자를 선언하는것만으로 그 메서드가 스레드 안전하다고 믿기는 어렵다.
- 멀티스레드 환경에서도 API를 안전하게 사용하게 하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.
- 다음 목록은 스레드 안전성이 높은 순으로 나열한 것으로, 완벽하진 않지만 일반적인 경우는 포괄한다
<br>

### **불변(immutable)**

- 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다.
- ex) String, Long, BigInteger
<br>

### **무조건적 스레드 안전(unconditionally thread-safe)**

- 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 사용해도 안전하다.
- AtomicLong, ConcurrentHashMap
- Atomic
<br>

### **조건부 스레드 안전(conditionally thread-safe)**

- 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다.
- Collections.synchronized 래퍼 메서드가 반환한 컬렉션들이 여기 속한다.
<br>

### **스레드 안전하지 않음(not thread-safe)**

- 이 클래스의 인스턴스는 수정될 수 있다.
- 동시에 사용하려면 각각의 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다.
- ex) ArrayList, HashMap 같은 기본 컬렉션
<br>

### **스레드 적대적(thread-hostile)**

- 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다.
- 이 수준의 클래스는 일반적으로 정적인 데이터를 아무 동기화 없이 수정한다.
- 고의로 만들 일은 없겠지만, 우연히 만들어질 수 있다.
- 스레드 적대적으로 밝혀진 클래스나 메서드는 문제를 고쳐 재배포하거나 사용 자제(deprecated) API로 지정한다.
- ex) 아이템 78-4의 generateSerialNumber() 메서드

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber ++;
}
```

- volatile 키워드는 Java 변수를 Main Memory에 저장하겠다는 의미
- 위 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다.
- nextSerialNumber 라는 단 하나의 필드로 결정되는데, 원자적으로 접근할 수 있고 어떤 값이든 허용한다.
- 따라서 굳이 동기화하지 않더라도 불변식을 보호할 수 있어 보이지만, 동기화 없이는 올바로 동작하지 않는다.
- 문제는 "증가 연산자(++)" 이다.
- 위 코드는 실제로 nextSerialNumber 필드에 두 번 접근하는데, 먼저 값을 읽고, 그 후 새로운 값(1증가)을 저장하는 것이다.
- 만약 두 번째 스레드가 위의 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다.
- 따라서 위 메서드에서 synchronized 한정자를 사용하면 이와 같은 문제가 해결된다.
- 위 분류는 (스레드 적대적만 빼면) "자바 병렬 프로그래밍"의 부록A에 나오는 스레드 안전성 애너테이션(@Immutable, @ThreadSafe, @NotThreadSafe)과 대략 일치한다.
<br>

## **정리**

- 모든 클래스가 자신의 스레드 안전성 정보를 명확히 문서화해야 한다.
- 정확한 언어로 명확히 설명하거나 스레드 안전성 애너테이션을 사용할 수 있다.
- synchronized 한정자는 문서화와 관련이 없다.
