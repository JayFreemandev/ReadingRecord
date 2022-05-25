# Item16, public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

Date: May 25, 2022

# 왜

Public 필드로 만들 경우에 패키지 바깥에서 접근할 수 있는 클래스라면 

접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유

연성을 얻을 수 있다. 하지만 **누구나 접근할수있기에**  public 클래스가 필

드를 공개하면 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 

방식을 마음대로 바꿀 수 없게 된다.

# 예외

자바 라이브러리중 public 클래스의 필드를 직접 노출하지 말라는 규칙을 

어기는 사례가 존재한다. 대표적인 예가, point와 dimension 클래스다. 

(웹개발을 하면서 사용해본적 없음) 내부를 노출한 결과로 심각한 성능문

제로 오늘날까지 고통받는 라이브러리들이다.

이 클래스들을 흉내 내지 말고 타산지석으로 삼아서 명심하자. 

Public으로 만들경우 클래스의 필드가 불변이면 상관없지 않을까? 

불변일때는 그나마 직접 노출시 단점이 줄어들지만 좋은 생각이 아니다.

API를 변경하지 않고는 내부값을 바꿀 수 없고 필드를 읽을때 부수적인 

작업을 수행할 수 없다라는점이 있다. 단, 불변식은 보장된다.

### 다음 예시는 각 인스턴스가 유요한 시간을 보장한다.

```java
private final class Time {   
 private static final int HOURS_PER_DAY = 24; 
   private static final int MINUTES_PER_HOUR = 60; 
    public final int hour; 
   public final int minute;  
   public Time(int hour, int minute) {  
      if (hour < 0 || hour >= HOURS_PER_DAY)     
       throw new IllegalArgumentException("시간: " + hour);   
     if (minute < 0 || minute >= MINUTES_PER_HOUR)    
        throw new IllegalArgumentException("분: " + minute); 
       this.hour = hour;       
			 this.minute = minute; 
   }    ... // 나머지 생략}
```

# 결론

**public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다**. 불변 필드라

면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-

private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필

드를 노출하는 편이 나을 때도 있다.
