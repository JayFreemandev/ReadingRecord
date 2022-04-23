# Item26, raw타입 사용하지마라 (1)

Date: April 18, 2022

## Raw타입이란

제네릭타입에서 매개변수를 전혀 사용하지 않은때를 말한다.

예를들어서 List<E>의 raw타입은 List다. 

제네릭 도입 이후에 Raw타입 지원 이유는 기존 자바 버전과의 호완성때문이다.

```jsx
//로 타입으로 인스턴스 저장
private final Collection stamps = ...;
stamps.add(new Coin());

//데이터 꺼낼 때 오류 발생
for(Iterator i = stamps.iterator(); i.hasNext();){
    Stamp stamp = (Stamp) i.next();  //ClassCaseException 발생
    stamp.cancel();
}
```

Raw로 저장을 하면 컴파일시에는 에러가 발생하지않는다.

하지만 런타임 때 for문에서 객체를 꺼낼때 오류가 발생한다.

무슨 오류든 컴파일 시점에 발견하는게 베스트다. 

그렇기에 Raw를 사용하지않는다.

## List<Object>

List Raw의 타입은 권장하지 않지만 List<Object>를 사용하는것은 좋다.

왜냐하면 모든 타입을 허용한다는 의사를 컴파일러에게 전달했기 때문이다. 

List와 List<Object>의 차이는 List는 제네릭 타입과 무관하다.

```jsx
import java.util.ArrayList;
import java.util.List;

public class UnsafeAdd {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();

        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0);
    }

    // 로 타입
    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }

}
```

List를 사용해서 unsafeAdd라는 메소드를 호출하여 사용한다.

컴파일 에러가 발생하지 않고 코드를 실행해야 strings.get(0); 에서 오류가 난다.

```jsx
mport java.util.ArrayList; 
import java.util.List;

public class UnsafeAdd {

    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();

        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0);
    }

    // List<Object>
    private static void unsafeAdd(List<Object> list, Object o) {
        list.add(o);
    }

}
```

이번에는 unsafeAdd의 타입 List가 List<Object>로 바뀌었다. 

컴파일 시점에 오류를 확인할수있다. 

List<String>은 raw타입인 List의 하위타입이지만 Object의 하위는 아니다.

그렇게에, List<Object>는 더욱 typeSafe한 방법이라고 말할수있다.

## 원소를 몰라요

```jsx
static int numElementsInCommon(Set s1, Set s2){
    int result = 0;
    for(Object o1 : s1){
        if(s2.contains(o1))
            result++;
    }
    
    return result;
}
```

Set뒤에 Raw 타입을 사용중이다 아무것도없다. 코드가 작동은 한다.

Typesafe하지않을뿐. 제네릭 타입을 사용하고는 싶지만 무슨 매개변수가 

올지 모르는 경우 신경쓰지않다면 와일드카드 <?> 넣어주자.

## 예외 케이스

Raw를 쓰지말라는 규칙에도 예외는 존재한다. 

Class 리터럴에는 Raw타입을 사용해야한다. 

예를 들어 List.class String[].class, int.class는 허용하지만

List<String>.class 이런거는 안된다.

두번째 예외는 instaceOf 연산자를 사용할때 런타임에는 제네릭 타입 정

보가 지워지므로 instanceof 연산자는 비한정적 와일드카드 타입 이외의 

매개변수화 타입에는 적용할 수 없다. Raw든 와일드든 instaceOf에서는 

똑같이 작동한다.

```jsx
if( o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```
