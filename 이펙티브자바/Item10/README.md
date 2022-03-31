# Item10, equals 재정의

Date: March 30, 2022

## Equals 재정의 하는 과정

1. == 으로 참조 주소를 확인한다.
비교 대상이 자기 자신의 참조인지

2. `instanceof` 로 입력된 변수가 올바른 타입인지 확인한다. 
    - parent instanceof Parent : 부모가 본인집을 찾았으니 true
    - child instanceof Parent : 자식이 상속받은 부모집을 찾았으니 true 
    (상속을 받았으니 자기집이라해도 무방하다?)
    - child instanceof Child : 자식이 본인집을 찾았으니 true
3. 입력한 대상을 형변환한다. 위에서 타입검사를 했으므로 넘어가게된다.

4. 입력한 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치한지 비교한다.
모두 일치하면 넘어가고 아니면 false다.
float과 double을 제외한 기본 타입은 == 연산자로, 
float과 double은 부동 소수점때문에 float.compare, double.compare로 비교한다.
참조 타입 필드의 경우는 ==이 아니라 equals 메서드로 비교해준다. 

```java
if( o == this) {
            return true;
        }

        if( o == null) {
            return false;
        }

        if(!(o instanceof PhoneNumber)) {
            return false;
        }

        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && 
							 pn.prefix == prefix && 
							 pn.areaCode == areaCode;
    }
```

Date를 상속하여 만들어진 Timestamp의 Equals 꼬라지를 보면 

```java
    Timestamp timestamp = new Timestamp(0L);
    Date date = new Date(timestamp.getTime());

    System.out.println(timestamp.equals(date)); // false
    System.out.println(date.equals(timestamp)); // true
```

**Timestamp의 equals 메서드에서는** instanceof 연산자로 인해 `false`가 된다. 

물론 타입 검사없이 형변환을 한다고 하더라도 nanos 값을 검사로 인해 false가 반환될 것이다.

**Date의 equals 메서드에서는** 시간이 같은지만 검사하므로 `true`가 된다.

즉, 먼저 살펴본 것처럼 결론적으로 정말 필요한 상황이 아니라면 재정의하지 않는 것이 좋다.

## 결론

위와 같은 방법으로 재정의할수있으나 하지마라.

- 값을 표현하는 것이 아니라 **동작하는 개체를 표현하는 클래스인 경우**
    - 예를 들어 Thread 클래스가 있을 때, Object의 equals 메서드로도 충분하다.
    - 인스턴스가 가지는 값보다 동작하는 개체임을 나타내는 것이 더 중요하다.
- **논리적 동치성(logical equality)을 검사할 필요가 없는 경우**
    - 예를 들어서 두 개의 Random 객체가 같은 난수열을 만드는지 확인하는 것은 의미가 없다.
- **private이나 패키지 전용 클래스**라서 클래스의 equals 메서드가 절대 호출되지 않아야하는 경우
    - 이런 경우에는 equals 메서드를 반드시 오버라이딩해서 호출되지 않도록 막아야 한다.
