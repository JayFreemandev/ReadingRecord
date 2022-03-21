# Item4, 인스턴스화를 막으려거든 private 생성자를 사용하라

## 단순한 정적 메서드와 정적 필드만을 담은 클래스

예를 들어) 자바 컬렉션중에 Math라던가 Arrays 유틸 클래스들처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 모아 놓을수가있다.

```java
public class Item4 { 
    public static void main(String[] args) {
      Math.abs(3);
      Arrays.sort(new int[]{1, 3});
    }
}
```

생성자를 명시하지 않으면 컴파일러가 자동으로 public 생성자를 만들게된다.

이게 사용자 의도로 만든건지 컴파일러가 없어서 만든건지 알 수 없음.

실제로 공개된 API중에서도 인스턴스화 할 수 있도록 의도치않은 케이스가 발견됨.

## 결론

private로 쓰면 상속도 안되고 인스턴스화도 막을수있다.

### +

클래스 바깥에서 private 생성자에 접근할수없으니 아래와 같이 굳이 error를 던지지 않아도 된다. 

```java
private UtilityClass(){
	throws new AssertionError();
}
```

단 에러 설정을 해준다면 의도는 명확하게 전달할 수 있다.