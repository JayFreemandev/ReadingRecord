# Item53, 가변인수는 신중히

## **가변인수란?**

- 명시한 타입의 인수를 0개 이상 받을 수 있도록 하는 것이다.

```java
static int sum(int... numbers) {
	int sum = 0;
	for (int number : numers) {
		sum += number;
	}
	
	return sum;
}

sum(1, 2);
sum(1);
sum();
```
<br>

- 가변인수 메소드를 호출하면 먼저 인수의 갯수와 같은 배열을 만들고 인수들을 배열에 저장해 가변인수 메서드에 전달한다.
- 최소한 인수가 한개라도 필요하게 하려면 아래와 같이 validation 로직을 작성해주면 된다.

```java
static int sum(int... numbers) {
	if (numbers.lenght == 0) {
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
	}
	int sum = 0;
	for (int number : numers) {
		sum += number;
	}
	
	return sum;
}

sum(1, 2);
sum(1);
sum(); // 에러 발생
```
<br>

## **단점**

- 위와 같이 validation을 하면 컴파일단계가 아니라 런타임시에 에러가 발생한다는 점입니다.
- 그러나 이러한 문제는 아래와 같이 해결할수 있다.

```java
static int sum(int firstNumber, int... numbers) {
	int sum = firstNumber;
	for (int number : numers) {
		sum += number;
	}
	
	return sum;
}

sum(1, 2);
sum(1);
sum(); // 컴파일 시 에러
```

- 성능에 민감한 사항에서는 가변인수가 걸림돌이 될 수 있습니다.

- 왜냐하면 호출될 때마다 배열을 새로 하나 할당하고 초기화하기 때문입니다.

- 최소한으로 하기 위해서는 아래와 같이 여러개의 메서드를 오버로딩하는 전략을 사용하면 된다.

```java
static int sum(int number1)
static int sum(int number1, int number2)
static int sum(int number1, int nubmer2, int number3)
static int sum(int number1, int nubmer2, int number3, int... numbers)
```

- 그러나 개인적으로 굉장히 코드가 더러워지느걸 볼수 있다.

- 추가적으로 EnumSet이 잘 구현되어 있는 편입니다.

```java
public static <E extends Enum<E>> EnumSet<E> of(E e) {
    EnumSet<E> result = noneOf(e.getDeclaringClass());
    result.add(e);
    return result;
}
public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    return result;
}

public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    return result;
}

public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    result.add(e4);
    return result;
}

public static <E extends Enum<E>> EnumSet<E> of(E e1, E e2, E e3, E e4,
                                                    E e5) {
    EnumSet<E> result = noneOf(e1.getDeclaringClass());
    result.add(e1);
    result.add(e2);
    result.add(e3);
    result.add(e4);
    result.add(e5);
    return result;
}

@SafeVarargs
public static <E extends Enum<E>> EnumSet<E> of(E first, E... rest) {
    EnumSet<E> result = noneOf(first.getDeclaringClass());
    result.add(first);
    for (E e : rest)
        result.add(e);
    return result;
}
```
<br>

## **결론**

- 일정하지 않은 매개변수가 들어온다면 가변인수가 반드시 필요하지만, 역시나 단점은 존재한다.
- 최소한의 성능문제를 겪기 위해서는 메소드를 오버로딩해서 사용하는걸 추천한다.

가변인수를 실무에서 아직 한번도 본적이없는데 단순한 성능문제 때문인지 가독성 문제때문인지 모르겠다.
