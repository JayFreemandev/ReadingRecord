# Item32, 제네릭과 가변인수를 쓸때는 신중하라

# 가변인수

Method의 Argument의 개수를 클라이언트가 조절할 수 있게한다.

반드시 한개의 가변인수만 허용, 맨 마지막 Argument로 사용해야한다.

```java
Static void mergeAll(List<String>... stringLists){} 
```

위 코드를 풀어서 쓰면 다음과 같은 의미를 지닌다

List<String> one, List<String> two, List<String> three

# 힙오염

제네릭을 사용할때 컴파일시점에 남아있는게 있고 컴파일 시점에 사라지는게 있다. 

가변인수일경우 컴파일 시점에 사라지기때문에 안전성을 보장할수가없다.

```jsx
static<T> List<T> flattern(List<? extends T> ... lists) {
return null;
}
```

작성시 인텔리제이에서는 Possible heap pollution form parameterized vararg  type 경고를 준다.

```jsx
List[] test = {List.of(1), List.of(2)}; // ok
List<Integer[] test2 = {List.of(1), List.of(2)}; // error
```

단순 Read라면 문제가 없지만 여기서 계산을 한다거나 다른 외부로 호출이될 경우 문제가 생긴다.

# 경고 제거

@SuppressWarnings(컴파일 경고 숨기기) 

@SafeVarargs(메서드의 타입 안정성을 보장) 

문제를 일으킬수 있지만 컴파일을 못하게하는건 가혹하기에 경고로 끝내주는 어노테이션이다.

## 강의중,

이펙티브 자바에서 이 코드가 정답이야보다 이렇게 하면 좀 더 좋아질수있지 않을까? 라는 생각으로 

접근하고 업무에서 적용하면 좋겠지만 아닐 확률이 높기에 스트레스 받지말고 각자 주어진 환경에

서 하루 하루 최선을 다하면 좋을거같다.

# 힙오염을 막기위해서

제네릭 배열에는 아무것도 저장하지말고 덮어쓰지말고 외부로 노출시키지마라.

제네릭과 가변인수는 궁합이 안맞다.
