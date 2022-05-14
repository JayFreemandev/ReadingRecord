# Item30, 이왕이면 제네릭메서드로 만들라

매개변수화 타입을 받는 정적 유틸리티 메서드(static method)는 보통 제네릭.

(매개변수화 타입이란 제네릭 타입이 인스턴스화 되었을때의 타입을 말함)

클래스뿐만 아니라 메소드도 제네릭타입으로 만들수가있다.

자주 사용하는 Collection의 메서드 sort 같은 경우도 제네릭이다.

```java
public static <E> List<E> union(List<E> list1, List<E> list2) {
     List<E> allList = new ArrayList<>();
     allList.addAll(list1);
     allList.addAll(list2);
     return allList;
 }
```

이러면 모든 매개변수 타입이 같아진다. 

위의 Union 메소드는 제네릭 타입을 하나의 Element로만 받기로 지정을 해놓은 상태이다.

만약에 파라미터로 들어오는 list1의 제네릭 타입은 Integer, list2의 제네릭타입은 String이면

아래와 같은 컴파일 에러를 볼수있다.  

![Untitled](https://user-images.githubusercontent.com/72185011/168414645-43ac04c8-3ba4-4c1a-9c15-eb2286e1ac73.png)

![Untitled 1](https://user-images.githubusercontent.com/72185011/168414636-54052e40-9f2d-41ab-8a7e-77bc30f275ed.png)
## 결론

클라이언트에서 매개변수와 반환값을 명시적으로 형변환하는 메서드보다

제네릭으로 타입안정성을 보장하는 경우가 더 안전하고 사용하기 쉽다.
