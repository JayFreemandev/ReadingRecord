# Item68, 일반적으로 통용되는 네이밍 규칙을 따르라
<br>

# 성능보다는 견고한 구조의 프로그램을 작성하자견고한 구조의 프로그램을 작성하다보면 성능은 자연스레 따라온다
<br>

### **자바의 명명 규칙은 크게 철자와 문법, 두 범주로 나뉜다.**

- 철자 규칙은 패키지, 클래스, 인터페이스, 메서드, 필드, 타입 변수의 이름을 다룬다.
- 이러한 규칙들은 특별한 이유가 없는 한 **반드시 따라야 한다.**

### **패키지와 모듈**

- 이름은 각 요소를 점(.)으로 구분하여 계층적으로 짓는다.
- 요소들은 모두 소문자 알파벳 혹은 (드물게) 숫자로 이뤄진다.
- 조직의 인터넷 도메인 이름을 역순으로 사용한다.
    - org.junit.jupiter
        - org: organization
    - com.google.common
        - com: company
- 예외적으로 표준 라이브러리와 선택적 패키지들은 각각 java와 javax로 시작한다.
    - java.util.Objects;
    - javax.persistence.*;
- 많은 기능을 제공하는 경우 계층을 나눠 더 많은 요소로 구성해도 좋다.
    - java.util.concurrent.atomic

### **클래스와 인터페이스**

- 이름은 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다.
    - List, FutherTask
- 약자의 경우 첫 글자만 대문자로 하는 쪽이 훨씬 많다.
    - HttpUrl vs HTTPURL

### **메서드와 필드**

- 첫 글자를 소문자로 쓴다는 점만 빼면 클래스 명명 규칙과 같다.
    - remove, ensureCapacity
- 단, '상수 필드'는 예외다.
- 상수 필드를 구성하는 단어는 모두 대문자로 쓰며, 단어 사이는 밑줄로 구분한다.
- 이름에 밑줄을 사용하는 요소로는 상수 필드가 유일하다.

### **변수**

- 타입 매개변수 이름은 보통 한 문자로 표현한다.
- T: 임의의 타입
- E: 컬렉션 원소의 타입
- K: 맵의 키, V: 맵의 값
- X: 예외
- R: 메서드의 반환

### **네이밍(문법 규칙)**

- 객체를 생성할 수 있는 클래스의 이름은 보통 단수 명사나 명사구를 사용한다.
    - Thread, PriorityQueue
- 객체를 생성할 수 없는 클래스의 이름은 보통 복수형 명사로 짓는다.
    - Collectors, Collections
- 어떤 동작을 수행하는 메서드의 이름은 동사나 동사구로 짓는다
    - append, drawImage
- boolean 값을 반환하는 메서드는 보통 is나 (드물게) has로 시작하고 명사, 명사구, 형용사구 등으로 끝나도록 짓는다
    - isDigit, isEmpty, isEnabled
- 반환 타입이 boolean이 아니거나 해당 인스턴스의 속성을 반환하는 메서드의 이름은 보통 명사, 명사구, get으로 시작하는 동사구로 짓는다.
    - size, hashCode, getTime

### **특별한 메서드 이름**

- 객체의 타입을 바꿔서, 다른 타입의 또 다른 객체를 반환하는 메서드는 보통 toType 형태로 짓는다.
    - toString, toArray
- 객체의 내용을 다른 뷰로 보여주는 메서드는 asType 형태로 짓는다.
    - asList
    
    ```java
     public static <T> List<T> asList(T... a) {
            return new ArrayList<>(a);
        }
        List<Integer> list = Arrays.asList(1, 2, 3, 4);
    ```
    
- 객체의 값을 기본 타입 값으로 변환하는 메서드의 이름은 보통 typeValue 형태로 짓는다.
    - intValue
- 정적 팩터리의 이름은 다음과 같이 짓는다.(아이템 1)
    - from, of, valueOf, getInstance, newInstance ...
    

**읽어볼거리**

메서드 이름은 빌더와 조정자의 차이를 생각하고 지으면 더 좋은 이름이 나타날 수 있다.

![Untitled](https://user-images.githubusercontent.com/72185011/178976963-f4a88bf4-d57a-4981-a5bd-3542a06cf11c.png)

***엘레강트 오브젝트*** 

![Untitled 1](https://user-images.githubusercontent.com/72185011/178976974-88cccbff-d655-4e19-9493-18e87c074ea8.png)

```java
// 잘못된 예시들
int save(String content); // 새로운 객체를 반환하는 메서드(빌더)지만 동사로 지어져있음(조정자)
void speed(String value); // 객체를 수정하는 메서드(조정자)지만 명사로 지어져있음(빌더)
```

****좋은 객체의 7가지 덕목****

[https://www.yegor256.com/2014/11/20/seven-virtues-of-good-object.html](https://www.yegor256.com/2014/11/20/seven-virtues-of-good-object.html)

위 엘레강트 오브젝트의 원문이다. 

- 객체는 현실 세계에 존재하며, 객체는 생명체로 취급해야한다.
    - 그렇기 때문에 컨트롤러, 파서, 필터등은 좋은 객체가 아니다. → 이것 들은 현실세계에 존재하지 않기 때문이다. → 즉, 대부분 GoF 패턴은 안티패턴이다?
- 객체가 살아있는 동안은 변하지 않은 채로 머물러야한다. (불변적이여한다.)
- 객체의 클래스에는 정적 멤버가 없다.
- 객체의 이름에서 객체가 하는 역할을 드러내면 안된다. (ex. FileReader)

위에 책 읽어봐야할거같다. 

## 결론

- 표준 명명 규칙을 체화하여 자연스럽게 베어 나오도록 하자.
- 철자 규칙은 직관적이라 모호한 부분이 적은데 반해, 문법 규칙은 더 복잡하고 느슨하다.
- "오랫동안 따라온 규칙과 충돌한다면 그 규칙을 맹종해서는 안 된다." 상식이 이끄는대로 따르자.
