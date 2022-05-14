# Item31, 한정적 와일드카드로 API의 유연성을 높여라

매개변수화 타입은 불공변(invariant) 입니다. 예를 들어 Type1과 Type2가 있을 

때, `List<Type1>`은 `List<Type2>`의 하위 타입 또는 상위 타입이라는 관계가 성

립될 수 없다.

조금 더 풀어보면 `List<Object>`에는 어떠한 객체도 넣을 수 있지

만 `List<String>`에는 문자열만 넣을 수 있습니다. 즉 `List<String>`

이 `List<Object>`의 기능을 제대로 수행하지 못하므로 하위 타입이라고 말할 수 

없습니다.

Stack클래스에서 한 메소드는 매개변수의 모든 원소를 넣는 예시를 들어보면

```java
public void pushAll(Iterable<E> src) {
    for (E e : src) {
        push(e);
    }
}
```

Number 타입으로 선언된 Stack 객체의 메서드에 Integer 타입의 매개변수를

넣어버리면 컴파일 오류가 발생한다. Integer는 Number의 하위 타입이니 잘 

동작 학러같지만. 오류가 발생한다.

```java
Stack<Number> numberStack = new Stack<>();
        Iterable<Integer> integers = Arrays.asList(
                Integer.valueOf(1), Integer.valueOf(2));

        // incompatible types...
        numberStack.pushAll(integers);
```

왜냐하면 제네릭의 매개변수화 타입은 불공변이기때문에 상위-하위 자료형과 관계가 없다,. 이런 문제를 해결하기 위해 한정적 와일드카드(bounded wildcard)자료형을 사용하게 된다.

```java
Iterable<? extends E> test
```

위 코드가 한정적 와일드카드 자료형인데 해석하면 매개변수는 E의 Iterable이 아니라 E의 하위타입의 Iterable이라는 뜻이다.

어떤 Element를 확장한 와일드카드. 한정적 와일드카드라고한다 (bounded wildcard) 

즉 .Number를 상속하는 Integer, Long, Doubl 타입의 요소를 가질수있게된다.

![Untitled](https://user-images.githubusercontent.com/72185011/168414693-31a76883-e3e9-4e7a-a261-3a43824ab7e3.png)


위에 Stack을 보면 push(E)를 통해서만 요소를 추가할수있다. 타입 안정성은 보

장되지만 elements 배열은 런타임시에 E[]가 아닌 Objects[]가 된다. 왜냐하면 

런타임시에 제네릭 타입이 소거가 되기 때문이다.

하위 타입을 상위 타입에게 상속하는건 당연하지만 제네릭의 매개변수는 다시 

언급하면 불공변이다. 상속이랑 관계가 무의미하기에 한정적 와일드카드형을

이용한다.
