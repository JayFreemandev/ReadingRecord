# Item38. 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용

설명하는분은 이번 아이템이 **실무에서 사용할 일이 있을까? 의문**이 들기에 내용은 가볍게 정리

Enum은 거의 모든 상황에서 typesafe enum pattern보다 우수하다. 다만 typesafe pattern 경우 확장이 가능하지만 enum은 그렇지 않다.

```java
//typesafe enum pattern

public class DSymbolType{
  private final String type;
  private DSymbolType(String type){
    this.type = type;
  }
  public String toString(){
    return type;
  }
  public static final DSymbolType Terminal = new DSymbolType("Terminal");
  public static final DSymbolType Process = new DSymbolType("Process");
  public static final DSymbolType Decision = new DSymbolType("Decision");
}
```

확장성이 높아지면 고려할 요소가 늘어나 설계와 구조가 복잡해진다. 

Enum의 경우는 확장이 불가능하도록 설계가 되어있다. 책에서는 연산 코드라는 예제를 통해서 인터페이스로 우회하는 경우를 보여주고있다. 인터페이스를 구현받는 패턴인데 공통적으로 사용할 수 있게 된다.

```java
public interface Operation {
    double apply(double x, double y);
}
```

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```

```java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
```

실제 사용에 대한 예제는 본인이 라이브러리를 개발하는게 아닌 이상 아 이렇구나 까지만 알아도 충분할거같다. 정리하자면 Enum을 확장할 수는 없지만 인터페이스를 통해 확장의 효과를 줄수있다. 

```java
private static void test(Collection<? extends Operation> opSet,
                         double x, double y) {
  for (Operation op : opSet)
    System.out.printf("%f %s %f = %f%n",
                      x, op, y, op.apply(x, y));
}
```

오히려 잘 쓰이지 않기때문에 모르고 넘어갈 가능성이 농후하다. 이런 기회가 아니라면 볼 일이 없기때문에 한 번 정독해보는것도 나쁘지않은거같다.
