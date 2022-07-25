# Item1, 무지성 public 생성자를 멈춰라

# 예제

클래스를 정의할 때 생성자로 정의해 클래스를 다른 곳에서 new로 생성하여 사용한다. 

```java
Champion Trunder = new Chanpion("Trunder", "Jungle"); // public constructor
```

하지만 기본 생성자에는 이름을 지을수없어서 생성자 안에 추가적인 로직이 있을때 직접 생성자 안까지 들어가서 확인을 하여야한다.   
이때 정적 팩토리 메소드를 사용하면 생성자를 통해 객체를 생성하는 것이 아닌 메소드를 통하여 객체를 생성시킨다.  

```java
Champion Trunder = Chapion.DefaultJungleLine("Trunder"); // static factory method 
```
<br>

# 장점은 무엇인가

- **객체 생성메소드의 이름을 가질수있다**.  모든 생성자의 이름은 같지만 매개변수만 다르다면 단순히 매개변수만을 보고 이것이 의미하는 바를 정확하게 알 수가 없다. 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 방법은 좋지 않다. 반면 정적 팩터리 메소드를 이름만 잘 지어도 반환될 객체의 특성을 한 눈에 알수있다.

**한 클래스에 생성자가 여러 개 필요하다면, 생성자를 정적 팩터리 메서드로 바꾸고 이름을 지어주자.**

```java
public class Champion {
    private String name;
    private String line;

    public Champion(String name) {
        this.name = name;
    }

    public Champion(String line) {
        this.line = line;
    }
}
```

public은 하나의 시그니처로만 생성이 가능하다 name과 line은 둘다 String으로 받는 생성자가 존재하기때문에 다른 메소드를 호출할 수 있다.

하지만 **정적 팩토리 메소드로는 하나의 String 시그니처로 여러 가지를 반환**시킬수있다.

```java
public static Champion name(String name) {
        Champion champion = new Champion();
        champion.name = name;
        return champion;
    }

  public static Champion line(String line) {
      Champion champion = new Champion();
      champion.line = line;
      return champion;
  }
```

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체든 변환이 가능하다. 반환 클래스를 자유롭게 선택할 수 있기 때문에 유연함을 가진다. 

이를 호출하는 클라이언트는 실제 구현 클래스가 무엇인지 몰라도 된다. 

- **호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.**

*불변 클래스*는 인스턴스를 미리 만들어서 불필요한 객체 생성을 막을 수 있다.

**불변 클래스 (Immutable)**

: 변경이 불가능한 클래스

- 레퍼런스 타입의 객체 (heap 영역에서 생성)
- Thread-safe (재할당은 가능. 값 복사 X)

**예를 들어 Boolean의 valueOf 는 객체를 아에 생성하지않는다.**

Boolean 상수 클래스만 봐도 상수로 선언되어 새로운 객체를 만들지않고 리턴해준다 .

생성 비용이 큰 객체가 자주 불려지는 상황에 정적 팩토리 메소드를 사용한다면 좋은 케이스다.

**인스턴스 통제 클래스**라고도 불리는데 정적 팩터리 메서드는 같은 객체를 반환하는 방법으로 언제 어느 인스턴스가 살아 있게 할지 통제할 수 있다. valueOf의 경우 b의 참, 거짓 여부에 따라 단 하나의 인스턴스를 보장한다.

```java
public final class Boolean implements java.io.Serializable,Comparable<Boolean> {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(boolean b) {
        return (b ? TRUE : FALSE);
    }
}

Boolean isTrundleOp = Boolean.valueOf(true);
```

# 단점은 무엇인가

1. 상속하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다?
    
    - 위 문장의 의미를 풀어서 해석하면, ‘**일반적으로 정적 팩토리 메서드는 `private` 생성자를 제공하므로 하위 클래스를 만들 수 없다**’는 말이다. 정적 팩토리 메서드의 생성자는 `private`으로 처리하는 이유는, 정적 팩토리 메서드를 포함한 클래스 자체가 객체(인스턴스)를 만들때 `public` 생성자를 사용하지 않고 정적 팩토리 메소드를 사용한다는 의도를 가지고 있기 때문이다. 
    
    **만약 하나의 클래스에 `public` 생성자와 정적 팩토리 메소드를 한 번에 같이 포함**하고 있다면 다른 개발자들이 **그 클래스를 봤을 때 어떤 의도로 이 클래스가 생성되었는 지 파악하지 못할 것**이다. 따라서 정적 팩토리 메서드를 제공하는 클래스는 `public`이나 `protectd` 생성자를 사용하지 않는 것이 일반적이다.
    
    위와 같은 규칙성으로 인해, 정적 팩토리 메서드를 가지고 있는 클래스에서는 `private` 생성자를 가지고 있을 것이다. 한편 상속을 하게되면 부모 클래스의 생성자를 뜻하는 `super()` 를 반드시 호출해야 한다. 하지만 정적 메소드를 가진 클래스의 생성자는 private으로 상속을 할 수 없다. 즉, 하위 클래스를 만들 수 없는 것이다.
    
    따라서 정적 팩토리 메소드는 `상속`보다는 `컴포지션`을 사용하도록 유도한다. 이 점 때문에, ‘**상속보다는 컴포지션을 사용해라’라는 통념에 부응하기 때문에 단점이 아닌 장점**으로 인식된다.
    
2. 정적 팩터리 메소드는 개발자가 찾기 힘들다. 
    1. 꼼꼼한 API 문서화가 필요하다. 
    2. **널리 알려진 규약**을 따라 짓는 식으로 문제를 해결해야한다.
    
    ![[https://devaily.tistory.com/26](https://devaily.tistory.com/26)](Item1,%20%E1%84%86%E1%85%AE%E1%84%8C%20a915e/Untitled.png)
    
    [https://devaily.tistory.com/26](https://devaily.tistory.com/26)
    

# 결론

결론적으로 생성자보다는 정적 팩토리 메서드를 사용할것이 추천된다. 단점또한 두가지가 있었지만 정적 팩토리 메소드만 있으면 이걸 상속받아 하위클래스를 만들 수 없다와 정적 팩토리는 다른 개발자들이 찾기 어렵다가 존재한다. 이에 대해서 해결책이 다 나와 있거나 큰 문제점이 아니라 쓰지 말아야할 단점으로 보기가 어렵다.
