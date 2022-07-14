# Item63, 문자열 연결은 StringBuilder

# ****문자열 연결은 느리니 주의하라****

## 

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이다.

하지만, 문자열 연결 연산자를 많이할 경우 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n2에 비례한다.

**왜 시간이 더 오래 걸릴까? 일단 String에 대해서 알고 넘어가야 한다.**

## **String 이란?**

**개념 :** [https://ko.wikipedia.org/wiki/%EB%AC%B8%EC%9E%90%EC%97%B4](https://ko.wikipedia.org/wiki/%EB%AC%B8%EC%9E%90%EC%97%B4)

- String은 새로운 값을 할당할 때마다 새로 생성
- 불변(immutable)의 속성을 가지고 있다.
- 이유 :
    - String의 값들은 외부에서 접근할 수 없도록 private로 보호
    - final형이기 때문에 초기값으로 주어진 String의 값은 불변(immutable)으로 바뀔 수 없다.
    - String pool 공간에 만들어지기 때문이다. [https://www.baeldung.com/java-string-pool](https://www.baeldung.com/java-string-pool)
- 변하지 않는 문자열을 자주 읽어들일 경우 String 사용
- 문자열 추가, 삭제 등의 연산이 빈번하게 발생하면 힙 메모리(Heap)에 많은 임시 가비지(Garbage)가 생성되어 치명적

> 요약하자면, 문자열(String)은 불변이라서 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하가 일어나는 것이다.
> 

**그러면 이러한 성능 저하를 보안하기 위해서 만들어진 것이 무엇일까?**

### **바로 StringBuilder와 StringBuffer이다.**

StringBuilder와 StringBuffer는 **가변성(mutable)**을 가지고 있다.

가변성을 가지고 있기 때문에 동일 객체내에서 문자열을 변경 가능.

## **StringBuilder 란?**

개념 : [https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html)

- 동기화(Synchronized)를 지원하지 않는다. (Non Thread Safe) -> 그대신 StrinbBuffer보다 속도가 빠르다
- 단일쓰레드에서의 성능은 StringBuffer 보다 뛰어나다.
- 기존의 데이터에 더하는 방식이라 속도가 빠르다.

## **StringBuffer 란?**

개념 : [https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/StringBuffer.html](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/StringBuffer.html)

- 멀티쓰레드 환경(Thread Safe)에서 안전하다 (Thread Safe) --> Synchronized를 이용한 동기화 가능
- String 자료형 보다 메모리 사용량이 많고 속도가 느리다.
- Buffer라는 독립적인 공간을 가진다. 기본적으로 16개 단어 저장가능

**문자열과 Builder 속도차이**

좋은 예시를 찾던 도중 김세윤님의 프로그래머스 문자열 문제가 있었다.

![Untitled](Item63,%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A1%E1%84%8B%E1%85%A7%E1%86%AF%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%AF%E1%84%8B%E1%85%B3%E1%86%AB%20StringBuilder%2013dd7cf9529b4bb6b1d96e388b5e4556/Untitled.png)

문자열 연결로 return을 구해보면 

```java
import java.util.Arrays;
import java.util.Collections;

class Solution {
    public String solution(String s) {
        String[] sArray = s.split("");
        Arrays.sort(sArray, Collections.reverseOrder());
        String answer = "";
        for (String sValue : sArray){
            answer += sValue;
        }
        return answer;
    }
}
```

Arrays.sort로 한번 정렬시키고 문자열을 사용하여 이어 붙였다.

![Untitled](Item63,%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A1%E1%84%8B%E1%85%A7%E1%86%AF%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%AF%E1%84%8B%E1%85%B3%E1%86%AB%20StringBuilder%2013dd7cf9529b4bb6b1d96e388b5e4556/Untitled%201.png)

StringBuilder를 사용했을때

```java
import java.util.Arrays;
import java.util.Collections;

class Solution {
    public String solution(String s) {
        String[] sArray = s.split("");
        Arrays.sort(sArray, Collections.reverseOrder());
        StringBuilder stringBuilder =new StringBuilder();
        for (String sValue : sArray){
            stringBuilder.append(sValue);
        }
        return stringBuilder.toString();
    }
}
```

얼마나 차이가 있을까?

![Untitled](Item63,%20%E1%84%86%E1%85%AE%E1%86%AB%E1%84%8C%E1%85%A1%E1%84%8B%E1%85%A7%E1%86%AF%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%A7%E1%86%AF%E1%84%8B%E1%85%B3%E1%86%AB%20StringBuilder%2013dd7cf9529b4bb6b1d96e388b5e4556/Untitled%202.png)

약 두배 가까이 속도가 향상되었다. 

배열인 조건에서는 String.join이라는 메소드도 사용이 가능한데 성능차이는 Builder랑 유사하기때문에 문자열 연결시에는 기본 문자열을 사용하지말자.

문자열 연결 순서가 중요한 멀티스레드 환경이라면 Buffer를 사용하자

# **결론**

성능에 신경 써야 한다면 많은 문자열을 연결할 때는 문자열 연결 연산자(+)를 피하자. 대신 StringBuilder의 append 메서드를 사용하자.
