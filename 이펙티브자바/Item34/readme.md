# Item34, int 상수 대신

## 열거타입 지원전

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;

public static final int ORANGE_NAVEL = 0
public static final int ORANGE_TEMPLE = 1;
```

1. 타입 안정성과 표현력이 좋지 않다  
APPLE_FUJI 나 ORANGE_TEMPLE 을 사용해도 둘다 정수 0이라 컴파일 문제 없고 ORANGE자리에 APPLE이 들어가도 모른다.

1. 프로그램이 깨지기 쉬움  
상수의 값이 바뀌면 반드시 다시 컴파일 필요

1. 정수 상수는 문자열로 출력하기 어려움

```java
public static final int APPLE_FUJI = 0;
System.out.println(APPLIE_FUJI); // APPLIE_FUJI 가 아닌 0 출력
```

## 열거 타입

정수 열거 패턴의 단점을 보완하고 여러가지 장점을 주는 대안이 바로 **열거 타입이다**

1. 열거 타입 선언으로 만들어진 인스턴스들은 딱 하나씩만 존재한다. 

열거 타입은 싱글턴을 일반화한 형태라고 볼 수 있다.

2. 열거 타입은 컴파일타임에 타입 안전성을 제공한다. 

즉, 값이 같은 0이더라도 Apple 열거 타입에 Orange 열거 타입이 들어올 수 없다.

3. 열거 타입에는 각자의 이름 공간이 있어서 같은 상수도 공존한다.

4. 열거 타입의 toString 메서드는 출력하기에 적합한 문자열로 반환한다.

5. 열거 타입에는 임의의 메소드나 필드를 추가할 수 있고 인터페이스를 구현할 수도 있다.

6. 열거 타입의 또 하나의 장점은 열거 타입에 선언한 상수 하나를 제거하더라도 **제거한 상수를 참조하지 않는 클라이언트에는 아무 영향이 없다** 또한 그런 상수를 참조를 하더라도 컴파일 에러가 발생할 것이다.

열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 final 이다. 따라서 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없으니, 인스턴스는 하나만 존재, 싱글턴.

반대로 싱글턴은 원소가 하나뿐인 열거 타입이라고 할 수있다.

나머지 내용들은 책에 나열된 장점과 동일함

패턴에 관한 내용을 짚고 넘어가자면 

### 값에 따라 분기하는 열거타입

좋은 방식이라고 할 수 있을까?

```java
enum PayrolLDay {
    MONDAY, TUESDAY, WEDSENDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY:

    private static final int MINS_PER_SHIFT = 8 * 60;
    
    int pay(int minutesWorked, int payRate) {
        int basePay = minuitesWorked * payRate;
        
        int overtimePay;
        switch(this) {
            case SATURDAY: case SUNDAY:
                overtimePay = basePay /2;
                break;
            default:
                overtimePay = minuutesWorked <= MINS_PER_SHIFT ? 0 : (minutesWorked - MINS_PER_SHIFT) * payRate /  2;
        }
        return basePay + overtimePay;
    }
}
```

단순하게 생각해도 새로운 날짜가 PayrollDay enum에 추가된다면 case또한 추가되어야한다. 

즉, 새로운 값마다 case가 늘어난다.

가장 깔끔한 방법은 새로운 상수를 추가할 때 잔업수당 '**전략**'을 선택하도록 하는 것이다.

잔업수당 계산을 private 중첩 열거 타입(아래 코드에서 PayType)으로 옮기고 PayrtrollDay 열거타입의 생성자에서 이 중 적당한 것을 선택하는 것이다. 그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요 없게 된다.

### 전략 패턴

```java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}
```

대부분의 경우 열거 타입의 성능은 정수 상수와 별반 다르지 않다. 열거 타입을 메모리에 올리는 공간과 초기화하는 시간이 들기는 하지만 체감할 정도가 아니다. switch 문보다 복잡하지만 더 안전하고 유연하다.

그래서 ***필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거타입을 사용하자.** (태양계 행성, 한 주의 요일, 체스 말)**열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다**. 열거 타입은 나중에 상수가 추가돼도 바이너리 수준에서 호환된다.

[https://techblog.woowahan.com/2527/](https://techblog.woowahan.com/2527/)
