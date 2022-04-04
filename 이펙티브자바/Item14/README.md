# Item14, Comparable을 구현할지 고려하라

### Comparable `<T>` 인터페이스란?

> Comparable 인터페이스는 객체를 정렬하는데 
사용되는 메서드인 compareTo()를 정의하고 있다.
> 

### Comparable 인터페이스 특징

- 자바에서 같은 타입의 인스턴스를 비교해야만 하는 클래스들은 모두 `Comparable` 인터페이스를 구현하고 있다.
- `Boolean` 타입을 제외한 래퍼 클래스와 알파벳, 연대같이 순서가 명확한 클래스들은 모두 정렬이 가능하다.
- 기본 정렬 순서는 **작은 값에서 큰 값으로 정렬**되는 **오름차순**이다.

### **Comparable**

을 구현했다는 것은 그 클래스의 인스턴스들에는 순서가 있음을 뜻한다.  이러한 특징 덕분에 비교 

연산이 포함된 여러 자바 라이브러리들은 이 인터페이스를 구현한 객체들에게 Arrays.sort이나 

Collections.addAll 등의 다양한 기능들을 제공한다. 거의 모든 값 클래스나 이넘 클래스는 

Comparable을 구현했다. 알파벳, 숫자, 연대 같이 명확한 클래스 정의시 구현하자.

이번 장에서 다른 메서드들과 달리 compareTo는 Object의 메서드가 아니다. 성격은 두 가지만 빼면 

equals와 같다. 그렇다면 무엇이 다른가? compareTo는 **단순 동치성 비교에 더해 순서까지 비교**할 수 

있으며, 제네릭하다.

Comparable을 구현하여 수많은 제네릭과 컬렉션의 힘을 누릴 수 있다. 알파벳, 숫자, 연대  같이 순서

가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

compareTo 메서드의 일반 규약은 equals의 규약과 비슷하다.

이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0

을, 크면 양의 정수를 반환한다. 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진

다.

다음 설명에서 sgn(표현식) 은 부호 함수를 뜻한다. -1,0,1을 사용한다.

- Comparable을 구현한 클래스는 모든 x, y에 대해
    
    sgn(x.compareTo(y)) == -sgn(y.compareTo(x) 예외도 마찬가지다.
    
- Comparable을 구현한 클래스는 추이성을 보장해야 한다.
    
    즉, x.compareTo(y) > 0 && y.compareTo(z) > 0 이면, x.compareTo(z) > 0이다.
    
- Comparable을 구현한 클래스는 모든 z에 대해
    
    ( x.compareTo(y) == 0 ) == ( x.equals(y)) 여야 한다. 
    
    이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다.
    
     "주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."
    

equals 메서드와 달리, compareTo는 타입이 다른 객체가 주어지면 

간단히 ClassCastException을 던져도 되며, 대부분 그렇게 한다.

compareTo 규약을 지키지 못하면 비교를 활용하는 클래스와 어울리지 못한다. 정렬된 컬렉션인 

TreeSet, TreeMap과 검색과 정렬 알고리즘을 활용하는 유틸리티 클래스인 Collections와 Arrays가 있

다.

위의 세 규약은 주의 사항도 equals와 똑같다. 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포

넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다. 우회법도 같다. Comparable을 구현한 클래

스를 확장해 값 컴포넌트를 추가하고 싶다면, 확장하는 대신 독립된 클래스를 만들고, 이 클래스에 원

래 클래스의 인스턴스를 가리키는 필드를 두고, 내부 인스턴스를 반환하는 '뷰' 메서드를 제공하면 된

다.

compareTo의 *마지막 규약은 필수는 아니지만 꼭 지키길 권한다*. 이를 잘 키지면 compareTo로 줄지

은 순서와 equals의 결과가 일관되게 된다. compareTo의 순서와 equals의 결과가 일관되지 않은 클

래스도 여전히 동작은 한다. 단, 이 클래스의 객체를 정렬된 컬렉션에 넣으면 해당 컬렉션이 구현한 

인터페이스(Collection, Set, 혹은 Map)에 정의된 동작과 엇박자를 낼 것이다. 이 인터페이스들은 

equals 메서드의 규약을 따른다고 되어 있지만, **놀랍게도 정렬된 컬렉션들은 동치성을 비교할 때** 

**equals 대신 compareTo를 사용하기 때문**이다. 주의해야 한다.

BigDecimal 클래스를 예로 들어보자. 빈 HashSet 인스턴스를 생성한 다음 new BigDecimal("1.0"), 

new BigDecimal("1.00")을 차례로 추가한다. equals 메서드로 비교하면 서로 다르기 때문에 HashSet

은 원소 2개를 갖게 된다. 하지만, TreeSet을 사용하면 하나만 갖게 된다. compareTo 메서드로 비교하

면 인스턴스가 똑같기 때문이다.

compareTo 작성 요령은 equals 와 비슷하지만 몇 가지 차이점만 주의하면 된다. Comparable은 인수

로 타입을 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해진다. 

입력 인수의 타입을 확인하거나 형변환할 필요가 없다는 뜻이다.

객체 참조 필드를 비교하려면 compareTo 메서드를 재귀적으로 호출한다. Comparable을 구현하지 

않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 대신 사용한다. 직접 만들

거나 자바 제공 중에 골라 쓰면 된다. 다음 코드는 아이템 10에서 구현한 CaseInsensitiveString 용 

compareTo 메서드로, 자바가 제공하는 비교자를 사용하고 있다.

```
public final class CaseInsensitiveString implements Comparable<CaseInsensitiveString> {
   public int compareTo(CaseInsensitiveString cis) {
      return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
   }
   ... // 나머지 생략
}
```

이 책의 2판에서는 compareTo 메서드에서 정수 기본 타입 필드를 비교할 때는 < 와 >를, 실수는 

Double.compare, Fload.compare를 사용하라고 권했다. 하지만 **자바 7부터는 상황이 변했다.** 

compare가 새롭게 추가 되었다. **<와 > 를 사용하는 이전 방식은 거추장스럽고 오류를 유발**하니, 이

제는 추천하지 않는다.

핵심 필드가 여러개라면 가장 핵심적인 필드부터 비교해나가자. 비교 결과가 0이 아니라면, 즉 순서

가 결정되면 거기서 끝이다. 바로 반환하면 된다. 다음은 PhoneNumber 클래스용 compareTo 메서드

를 이 방식으로 구현한 모습이다.

```java
public int compareTo(PhoneNumber pn) {
   int result = Short.compare(areaCode, pn.areaCode);
   if (result == 0) {
      result = Short.compare(prefix, pn.prefix);
      if (result == 0) {
         return = Short.compare(lineNum, pn.lineNum);
   }
   return result;
}
```

자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 팀을 꾸려 메서드 연쇄 방식

으로 비교자를 생성할 수 있게 되었다. 이를 이용하여 compareTo 메서드를 더 멋지게 활용할 수 있

다. 약간의 성능 저하가 뒤따른다고 한다.

```
private static final Comparator<PhoneNumber> COMPARATOR =
    comparingInt((PhoneNumber pn) -> pn.areaCode)
        .thenComparingInt(pn -> pn.prefix)
        .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

thenComparingInt는 원하는 만큼 연달아 호출할 수 있다. long, double 용으로도 메서드가 존재한다. 

short는 int용 버전을 사용하고, fload에서는 double 형 버전을 사용한다.

객체 참조용 비교자 생성 메서드도 준비되어 있다. comparing이라는 정적 메서드 2개가 다중 정의 

되어있다. 

첫 번째는 키 추출자를 받아서 그 키의 자연적 순서를 사용한다. 

두 번째는 키 추출자 하나와 추출된  키를 비교할 비교자까지 총 2개의 인수를 받는다. 

thenComparing도 경우에 따라 3가지가 다중정의되어 있다.

### 결론

> 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. 

compareTo 에서 필드의 값을 비교할 때 < , > 연산자는 쓰지 말아야한다. 

기본 타입 클래스가 제공하는 정적 compare 메서드나, Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.
>
