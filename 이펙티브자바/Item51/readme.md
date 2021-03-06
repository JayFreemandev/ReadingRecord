# Item51, 메서드 시그니처를 신중히 설계하라

## 이번 아이템은 개별로 두기 애매한 API 설계요령들을 모아놓았다.

- 메서드 이름을 신중히 짓자
- 편의 메서드를 너무 많이 만들지 말자
- 매개변수 목록은 짧게 유지하자
- 매개변수 타입으로는 클래스보다 인터페이스가 낫다
- boolean보다는 원소 2개짜리 열거 타입이 낫다

---
<br>

## **메서드 이름을 신중히 짓자**

1. **항상 표준 명명 규칙을 따르자.** → Item 68. 일반적으로 통용되는 명명 규칙을 따르라
    - 이해할 수 있고, 같은 패키지의 이름들과 일관되게 짓는 것이 최우선
2. 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용하라
3. 긴 이름은 피하자
4. 애매하면 Java 라이브러리 API 가이드를 참조하자

---
<br>

## **편의 메서드를 너무 많이 만들지 말자**

메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 **어렵다.** (인터페이스도 마찬가지)  
→ 클래스나 인터페이스는 자신의 **각 기능을 완벽히 수행하는 메서드**로 제공해야한다.  
→ 아주 자주 쓰이는 경우에만 약칭 메서드로 두고, 아니거나 애매하면 만들지 말자.  

---
<br>

## **매개변수 목록은 짧게 유지하자**

**4개 이하**가 적당하다.
- 4개가 넘어가면 매개변수를 전부 기억하기 어렵다.  
- 같은 타입의 매개변수 여러 개가 연달아 나오는 경우는 특히 해롭다.  
그럼 이걸 매개변수를 어떻게 줄일 것인가?
  
### **첫 번째, 여러 메서드로 쪼갠다.**

![https://user-images.githubusercontent.com/37873745/109662889-e0bde780-7bae-11eb-8daf-703200ee10a4.png](https://user-images.githubusercontent.com/37873745/109662889-e0bde780-7bae-11eb-8daf-703200ee10a4.png)

- 쪼개진 메서드 각각은 원래의 매개변수 목록의 부분집합을 받는다.
- **잘못하면 메서드가 너무 많아질 수 있지만, 직교성(orthogonality)을 높여 오히려 메서드 수를 줄여주는 효과도 있다.**
    - **직교성?:** 직교 - 직각을 이루며 교차하는 것
        
        → 직교하는 요소들은 서로 독립적이다.   
	→ 즉, "직교성이 높다는 것"은 "공통점이 없는 기능들이 잘 분리되어 있다",      
	→ "기능을 원자적으로 잘 쪼갰다"로 해석할 수 있다.  
        
- 메서드가 많아지지만, 메서드를 줄여준다?
    - 직교성이 낮다
        - 코드 중복, 결합성 높아짐
            
            ![https://user-images.githubusercontent.com/37873745/109662871-de5b8d80-7bae-11eb-9d22-339cfa653531.png](https://user-images.githubusercontent.com/37873745/109662871-de5b8d80-7bae-11eb-9d22-339cfa653531.png)
            
    - 직교성이 높다
        - 중복이 줄어듬, 결합성 낮아짐 ⇒ 코드 수정 수월
            
            ![https://user-images.githubusercontent.com/37873745/109664698-bff69180-7bb0-11eb-8f29-4803df4d44b7.png](https://user-images.githubusercontent.com/37873745/109664698-bff69180-7bb0-11eb-8f29-4803df4d44b7.png)
            
- 그렇다고 무한정 작게 나누는 것은 좋지 않다. API 사용자의 눈높이에 맞게, 구현 프로젝트에 어울리게 조절해야한다.
<br>

### **두 번째, 매개변수 여러 개를 묶어주는 도우미 클래스를 만들어라**

일반적으로 이런 도우미 클래스는 정적 멤버 클래스(Item 24)로 둔다.  

- 잇따른 매개변수 몇 개를 독립된 하나의 개념으로 볼 수 있는 경우, 추천하는 기법  
- Ex) 카드 게임에서의 카드를 전달하는 경우, 카드 무늬(suit)/숫자(rank)를 매개변수로 받을 때  
    - 카드 클래스를 다른 클래스에선 거의 사용 안한다고 가정  

```java
`public class CardGame {
	public boolean compareCards(int myCardRank, Suit myCardSuit, int otherCardRank, Suit otherCardSuit){
		// Spades 4 point, Clubs 3 point, Diamonds 2 point, Hearts 1 point
		// 숫자와 무늬 합이 누가 큰가?
		// ... Logic
	}
}`

`public class CardGame {
    public boolean compareCards(Card myCard, Card otherCard){
        // Spades 4 point, Clubs 3 point, Diamonds 2 point, Hearts 1 point
        // 숫자와 무늬 합이 누가 큰가?
    }

    public static class Card{
        private final int rank;
        private final Suit suit;
        public Card(int rank, Suit suit) {
            this.rank = rank;
            this.suit = suit;
        }
    }
}`
```
<br>

### **세 번째, 객체 생성에 사용한 빌더 패턴(Item 2)을 메서드 호출에 응용하라**

- 매개변수가 많을 때, 특히 그 중 일부는 생략해도 괜찮은 경우, 추천하는 기법
- 모든 매개변수를 하나로 추상화한 객체를 정의하고, setter 메서드를 통해 필요한 값을 설정하게 하는 것

```java
`public void inputGamer(String name, double height, double weight, boolean hair, int age){
	/*
		Logic
		매개변수가 너무 많다.
	*/
}

// 호출시 //
// 뭐가 먼저 나와야하는지 헷갈림
cardGame.inputGamer("Kim", 199.99, 99.99, true, 27);`

`public void inputGamer(Person person){
		// Logic
}

// 호출시 //
// 원하는 값만 순서에 상관없이 설정 가능
cardGame.inputGamer(Person.builder()
      .height(199.99)
      .weight(99.02)
      .name("Foo bar").build());

cardGame.inputGamer(Person.builder()
	    .name("Foo bar")
	    .height(199.99)
	    .weight(99.02)
	    .hair(false)
	    .build());

// 유효성 검사도 가능
cardGame.inputGamer(Person.builder()
	    .height(199.99)
	    .weight(-99.02)
	    .build()); // ERROR

// Person class //
@Builder(builderClassName = "Builder")
public class Person {
    private String name;
    private double height;
    private double weight;
    private boolean hair;
    private int age;

    public static class Builder {
        Person build() {
            if (height < 0 || weight < 0 || age < 0){
                throw new IllegalArgumentException("Invalid data: data is greater than 0");
            }
            return new Person(name, height, weight, hair, age);
        }
    }
}`
```

---
<br>

## **매개변수의 타입으로는 클래스보다는 인스턴스가 낫다**

- 매개변수로 적합한 인터페이스가 있다면 **인터페이스를 직접 사용**하자.
- 예를 들어 HashMap 대신 Map을 사용하는 것
    - public void someLogicExecute(Map<K, V> map)
    - TreeMap, ConcurrentHashMap 등 어떠한 Map 구현체던지 간에 해당 메서드를 활용할 수 있게 된다.

---
<br>

## **boolean보다는 원소 2개짜리 열거 타입이 낫다**

- 열거 타입(enum)을 사용하면 코드를 읽고, 쓰기가 더 쉬워진다.

예제: 화씨온도와 섭씨온도

- public enum TemperatureScale { FAHRENHEIT, CELSIUS }
- 이 열거타입을 받아 적합한 온도계 인스턴스를 만들어준다할 때
    - 열거타입이 아닌 경우
        - Thermometer.newInstance(true)
        - true면 섭씨 → 생성코드만을 보고는 무엇인지 알 수 없다.
    - 열거타입을 사용한 경우
        - Thermometer.newInstance(TemperatureScale.CELSIUS)
        - 훨씬 읽기가 좋아졌다.
        
<br>

#### 그림 및 설명 출저 (김보배)
