# Item35, ordinal 메서드 대신 인스턴스 필드를 사용

![Untitled](https://user-images.githubusercontent.com/72185011/175974995-e48ad1ec-0bcc-4afc-8bc9-da8c9680d19b.png)

Enum API 문서에 보면 ordinal에 대해 ‘**대부분 프로그래머는 이 메서드를 쓸 일이 없다**’ 라고 한다.   
ordinal은 EnumSet과 EnumMap이라는 열거 타입 기반 자료구조에 쓸 목적으로 설계되었는데 이 용도 외라면 사용하지마라. 

아직 실무에서 본적이 없고 강사는 Ordinal을 사용하는 사람이 있는지도 의문이라는 말을 했다. 해당 상수가 Enum 타입에서 몇 번째 상수인지 리턴해주는 메소드가 Ordinal의 기능이다.

### Ordinal 메서드에 대해 알아보자

```java
public enum Ensemble {
        SOLO, DUET, TRIO, QUARTET, QUINTET, 
        SEXTET, SEPTET, OCTET, NONET, DECTET;

        public int numberOfMusicians() { return ordinal() + 1; }
}
```

다음과 같은 합주단(솔로, 듀엣, 트리오 *** )  Enum이 존재하는데 OCTET은 8명이서 연주하고 DOUBLE_QUARTET또한 추가될시 8명이서 협주하는 같은 값을 가진다. 

Ordinal 사용지 중복 상수를 만들수없으므로 출력시 다른 값을 뱉으며 이 외에도 아래와 같은 이유로 사용을 권장하지 않는다.

**쓰지 말아야 하는 이유**

1. 상수 선언 순서를 바꾸면 ordinal()은 오동작한다.
2. 선언 순서에 기반해 동작하는 ordinal() 입장에선 당연하다.
3. 이미 사용중인 정수와 똑같은 값을 가지는 상수는 만들 수 없다.
4. 8중주는 8명이서 연주한다(octet). 그러나 복4중주(double quartet) 또한 8명이서 연주하는데 이를 표현할 방법이 없다.
5. 값을 중간에 비워둘 수도 없다.
6. dummy 상수를 넣어 순서를 맞추는 방법도 있지만 굉장히 비효율적이다.
7. API User를 위해 구현된 메서드가 아니다.
8. Enum API 문서에 명시된 바로는 EnumSet과 EnumMap과 같이 열거타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다고 한다

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

Enum의 순서를 알고 싶을때는 Ordinal 사용이 아니라 인스턴스 필드에 저장하여 사용하여 사용하는게 Best Practice다.

### 결론

Enum이 제공하는 Ordinal은 EnumSet, EnumMap에 사용할 목적으로 설계되었으므로 외에는 사용하지마라. (나는 실무에서 아직 본적이 없다.)
