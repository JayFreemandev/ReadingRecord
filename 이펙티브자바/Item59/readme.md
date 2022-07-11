# Item59, 라이브러리 알고쓰자

### 바퀴를 다시 발명하지 말자. 특별한 기능이 아니라면 라이브러리에 이미 구현되어있을 가능성이 크다. 그러므로 라이브러리 사용을 우선적으로 고려해보자. 좋은 품질의 코드를 손쉽게 만들어낼 수 있다.

## 표준 라이브러리를 모르고 사용할때

```java
// 문제가 많은 코드!
static Random rnd = new Random();

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```

위 코드는 세 가지의 문제점을 내포하고 있다.

1. n이 크지 않은 2의 제곱수라면 조금 뒤 같은 수열이 반복된다.
2. n이 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주 반환된다.
3. 지정한 범위를 벗어난 수가 종종 튀어나온다.

### 지정한 범위를 벗어나는 이유

위의 코드에서 Math.abs()로 정수를 반환하기 때문에 발생한다. 예를 들어, nextInt()가 Integer.MIN_VALUE를 반환하면 Math.abs도Interger.MIN_VALUE를 반환한다. 따라서 Math.abs(rnd.nextInt()) % n은 음수를 반환하게 된다.

> Math.abs(Integer.MIN_VALUE)가 음수를 반환하는 이유?Integer의 범위는 2^31(-2,147,483,648) ~ 2^31-1(2,147,483,647)이다. -2,147,483,648의 절댓값은 Integer 최대 범위를 벗어나고, 이 수는 Integer로 표현할 수 없다. 따라서 최솟값을 가장 잘 표현할 수 있는 동일한 값을 내어준다.
> 

### 해결 방법

Random.nextInt(n) 메서드를 사용하면 이 결함은 간단히 해결된다. 우리는 이러한 문제점을 직접 해결하려 하지 않아도 된다. 앞서 말했듯, 표준 라이브러리는 지속적으로 개선되고 있기 때문이다. 우리는 이러한 라이브러리들을 잘 익히고 사용하면 될 뿐이다.

Java 7부터는 Random을 더 이상 사용하지 않는 게 좋다. **ThreadLocalRandom**으로 대체하면 대부분 잘 동작하고 더 빠르다.

**ThreadLocalRandom에 대해**

`java.util.Random`은 멀티 쓰레드 환경에서 하나의 인스턴스에서 전역적으로 의사 난수(pseudo random)를 반환한다. 따라서 같은 시간에 동시 요청이 들어올 경우 경합 상태에서 성능에 문제

**Java가 난수를 생성하는 원리**

컴퓨터는 수학적으로 완전한 난수를 생성할 수 없다. 컴퓨터는 동일 입력에 동일 출력이 보장되어야 하는 **결정적 유한 오토마타**(Deterministic Finite Automata)이기 때문이다.

Java와 같은 언어는 **시드(Seed)와 그에 해당하는 난수의 대응으로 난수를 결정**한다. 해쉬 함수의 원리나 고등학교 수학의 상용 로그표와 유사하다. 이는 같은 시드가 주어질 경우 생성되는 난수는 같다는 것을 의미하고, 설계적으로 의도한 난수 생성을 위해서는 서로 다른 시드가 주어져야 한다는 것을 의미한다.

Java는 Seed를 지정하지 않으면 컴퓨터의 현재 시간을 이용하여 난수에 대응한다.

```java
public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }
```

```java
public Random(long seed) {
        if (getClass() == Random.class)
            this.seed = new AtomicLong(initialScramble(seed));
        else {
            // subclass might have overriden setSeed
            this.seed = new AtomicLong();
            setSeed(seed);
        }
    }
```

```java
private static long seedUniquifier() {
        // L'Ecuyer, "Tables of Linear Congruential Generators of
        // Different Sizes and Good Lattice Structure", 1999
        for (;;) {
            long current = seedUniquifier.get();
            long next = current * 1181783497276652981L;
            if (seedUniquifier.compareAndSet(current, next))
                return next;
        }
    }
```

**왜 Random이 멀티 스레드에서 위험한가?**

`java.util.Random`은 멀티 쓰레드에서 하나의 Random 인스턴스를 공유하여 전역적으로 동작한다.

> 이때 동일한 nanoTime에 멀티 쓰레드 요청이 들어오면 어떻게 될까? 같은 난수를 반환하는가?
> 

다행히 그렇게 동작하지는 않는다. `Random`의 의사 난수 생성은 **선형 합동 생성기**(Linear Congruential Generator)알고리즘으로 동작하는데, 하나의 쓰레드가 동시 경합에서 이기면 다른 쓰레드는 자신이 이길 때까지 계속 같은 동작을 반복하며 경합한다.

```java
private final AtomicLong seed;

protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        oldseed = seed.get();
        nextseed = (oldseed * multiplier + addend) & mask;
    } while (!seed.compareAndSet(oldseed, nextseed));

    return (int)(nextseed >>> (48 - bits));
}
```

이때 이 과정을 여러 쓰레드가 동시에 경합 - 패배 - 재도전을 반복할 경우 **성능상의 심각한 문제가 발생**할 수 있다. 랜덤으로 할인 쿠폰을 반환하는 이벤트에서 수만 명의 요청이 동시에 몰린다면? 문제가 생긴다.

**ThreadLocalRandom을 써야하는 이유**

`java.util.concurrent.ThreadLocalRandom`은 똑같이 Random API의 구현체이며, `java.util.Random`를 상속받는다. `ThreadLocalRandom`은 위의 동시성 문제를 해결하기 위해 **각 쓰레드마다 생성된 인스턴스에서 각각 난수를 반환**한다. 따라서 `Random`과 같은 경합 문제가 발생하지 않아 안전하며, 성능상 이점이 있다. `Random`대신 `ThreadLocalRandom`을 쓰자.

```
public static ThreadLocalRandom current() {
        if (U.getInt(Thread.currentThread(), PROBE) == 0)
            localInit();
        return instance;
    }
```

## 표준 라이브러리의 이점

### 1. 전문가의 지식과 다른 프로그래머들의 경험을 활용할 수 있다.

라이브러리의 코드를 작성한 전문가의 지식과 이를 사용해본 다른 프로그래머들의 많은 경험을 토대로 손쉽게 원하는 기능을 만들 수 있다.

### 2. 핵심적인 일과 관련 없는 문제를 해결하는데 드는 시간을 줄일 수 있다.

애플리케이션 기능 개발에 집중할 수 있다.

### 3. 따로 노력하지 않아도 성능이 개선된다.

표준 라이브러리는 더 나은 방법을 꾸준히 모색한다. 자바 플랫폼 라이브러리의 많은 부분들이 지속적으로 개선되며 성능 개선이 이루어진 것처럼 말이다.

### 4. 기능이 점점 많아진다.

라이브러리가 개선되어지며 부족한 기능들도 발견되는데, 이 부분이 다음 릴리즈 때 추가되곤 한다.

### 5. 작성한 코드가 자연스럽게 좋은 코드가 된다.

실상 많은 개발자들이 라이브러리를 사용하기보다 직접 구현해서 사용하고 있다. 그 이유는 아마도 라이브러리에 그런 기능이 존재하는지 모르기 때문일 것이다.

Java 메이저 릴리스마다 주목할 만한 수많은 기능이 라이브러리에 추가된다. Java는 이런 메이저 릴리스마다 새로운 기능을 소개하는 웹 페이지를 제공하니 관심을 가지고 살펴볼만하다.

Java의 라이브러리는 너무 방대해서 모든 API 문서를 공부하기 어려울 수 있다. 하지만 자바 개발자라면 java.lang, java.util, java.io와 그 하위 패키지들에는 익숙해져야 한다.

컬렉션 프레임워크, 스트림 라이브러리, java.util.concurrent의 동시성 기능도 알아두면 좋다.

스트림 라이브러리와 동시성 기능에 대해는 잘 모르기때문에 시간을 내서 숙지할 필요가 있어보인다.

라이브러리를 사용하면 사람들에게 낯익은 코드가 된다. 자연스럽게 다른 개발자들이 읽기 좋고, 유지 보수하기 좋고, 재활용하기 쉬운 코드가 된다.

구현해야 할 기능이 라이브러리로 존재한다면 그것을 사용하자. 있는지 잘 모르겠다면 찾아보자. 우리는 그 라이브러리가 어떤 영역의 기능을 제공하는지 살펴보고, 익히고, 사용하면 된다.

바퀴를 다시 발명하지 말자. 특별한 기능이 아니라면 라이브러리에 이미 구현되어있을 가능성이 크다. 그러므로 라이브러리 사용을 우선적으로 고려해보자. 좋은 품질의 코드를 손쉽게 만들어낼 수 있다.

### 우선순위

- 우선 라이브러리를 사용하려 시도하자.
- 원하는 기능이 없다면 고품질의 서드파티 라이브러리를 고려하자.
- 그래도 없다면 직접 구현하자.
