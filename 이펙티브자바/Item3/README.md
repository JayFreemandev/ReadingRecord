# Item3, private 생성자나 열거 타입으로 싱글턴임을 보증하라

**싱글턴의 사용 이유는 한번의 객체 생성으로 메모리 낭비를 방지하기 위해서이다.**

(스프링에서 Bean의 기본 scope가 singleton임. )

## 싱글턴을 만드는 두가지 방법

은 private로 생성자만든후 static으로 인스턴스 접근하는법이다. private 생성자로 싱글턴?

이렇게 만들게 되면 new를 통해서 밖에서 생성못하게 되고 유일한 인스턴스로 접근할수있는 

static method를 만들어서 접근할수있다. 

## 두가지 방법 모두 문제점이 존재 

**각 클래스를 직렬화한 후 역직렬화할 때 새로운 인스턴스를 만들어서 반환한다.**

싱글턴 클래스는 직렬화할때 Serializable 이거 붙인다고 끝나는게 아니다. 

모든 인스턴스 필드에 transient 선언하고 `readResolve` 를 제공해야지 역직렬화 할때마다 

새로운 인스턴스가 생성되는것을 방지할수있다. 

## 결론

### 이런 문제점 때문에 싱글턴을 만들기 위해서는 열거 타입으로 만들어준다.

대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글턴을 만들기 가장 좋은 방법이다.

```java
public enum Trundle{
	INSTANCE;

	public String getName() {
		return "Trundle";
	}

	public void leaveTheBuilding() { ... }
}
```

String name = Trundle.INSTANCE.getNameI(); 

트런들 타입의 인스턴스는 INSTANCE 하나뿐이라 더 이상 만들 수 없다.

직렬화 상황이나 리플렉션 set으로 들췄을때도 새로운 인스턴스 생성을 막아준다. 

유일한 예외 상황은 해당 ***싱글턴이 다른 상위 클래스를 상속받아야된다면 Enum을 사용할수없다.***
