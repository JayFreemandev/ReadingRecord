# Item11, equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.

Date: March 31, 2022

![Untitled](https://user-images.githubusercontent.com/72185011/161044137-a591f7ed-f64e-4979-8ac0-a6ba495f2c62.png)

### 왜

hashCode를 재정의하지 않으면 hashCode 일반 규약을 어기게 되어 

HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제가 발생

### equality와 identity의 차이

equals()는 두 객체의 내용이 같은지 equality를 비교하고 

hashCode()는 두 객체가 같은 객체인지 identity를 비교한다.

따라서 Heap에 있는 한 객체를 서로 다른 reference가 참조 하는 경우 같은 값을 반환한다.

하지만 같은 필드를 가진 두 개의 객체를 판단하기 위해 equals를 재정의하여 사용하는데 

이때 hahCode를 재정의 하지 않으면 각 객체는 서로 다른 hashcode를 가지게된다. 

### hashCode 재정의 조건

1. equals 비교에 사용되는 정보가 변경되지 않는다면, hashCode는 항상 같은 값을 반환해야 한다.
2. equals가 두 객체가 같다고 판단하면, hashCode는 같은 값을 반환해야 한다.
3. equals가 두 객체가 다르다고 판단해도, hashCode가 꼭 다를 필요는 없다. 
다만, 다른 객체에서는 다른 값을 반환해야 성능이 좋아진다.

### 좋은 hashCode

서로 다른 인스턴스에는 다른 해시코드를 반환한다. 

```java
@Override
public int hashCode() {
   int result = Short.hashCode(areaCode);
   result = 31 * result + Short.hashCode(prefix);
   result = 31 * result + Short.hashCode(lineNum);
   return result;
}
```

왜 하필 31이라는 숫자를 곱하는것인가? 자바8부터 17에 이르기까지 31로 곱하여 해시를 계산한다.

책에는 31로 정한 이유는 31이 홀수이면서 소수이기 때문이라고 한다. 

왜 짝수는 안되고 소수만 될까? 왜냐하면 숫자를 비트로 바꾸면, 

짝수를 곱한다는것은 Shift 연산과 동일하게된다. 

![Untitled 1](https://user-images.githubusercontent.com/72185011/161044185-13b28417-d7a9-4ed4-b268-5759943fac4b.png)  
<sub><sup>[https://www.google.com/url?sa=i&url=https%3A%2F%2Fkkhipp.tistory.com%2F66&psig=AOvVaw3H79buRtW15Ggw0orUm9LU&ust=1648811191007000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCNjV2L2a8PYCFQAAAAAdAAAAABAI](https://www.google.com/url?sa=i&url=https%3A%2F%2Fkkhipp.tistory.com%2F66&psig=AOvVaw3H79buRtW15Ggw0orUm9LU&ust=1648811191007000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCNjV2L2a8PYCFQAAAAAdAAAAABAI)</sup></sub>

짝수는 무조건 오른쪽이 0 이고 홀수는 1이다. 0이란 말은 이미 들어올수 있는 숫자 범위 절반을

날리고 시작하는 것과 동일하다. 많은 수를 비교해야 겹치지 않는 해시코드가 나오는데 따라서 충돌

가능성이 높아진다.

![Untitled 2](https://user-images.githubusercontent.com/72185011/161044305-bef368ec-2679-4e85-a463-4abc86e8b3b2.png)

<sub><sup>[https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier](https://stackoverflow.com/questions/299304/why-does-javas-hashcode-in-string-use-31-as-a-multiplier)</sub></sup>

그럼 왜 소수만 되는가? 스택오버플로우를 참고하면 일종의 미신이다. 소수를 사용하면 알고리즘

이 우수한 속도를 보일것이다 라는것이다. 소수중에 왜 하필 31인가에 대한거는 이전 어셈블러에서

최적화되기 때문이다. 31이란 8bit에 넣을수있는 max값이다. 요즘은 컴파일러가 좋아서 31뿐만 아니

라 다른 소수를 곱해도 빠르다. 실제로 롬북에서는 다른 숫자로 해시코드를 생성하고있다. 

![Untitled 3](https://user-images.githubusercontent.com/72185011/161044493-7d9997b6-c208-4ff9-b7fb-3742dee10563.png)

[lombok.org](http://lombok.org)

### 재정의시 주의할점

1. **불변 객체에 대해 hashCode 생성비용이 크다면 캐싱을 고려해라.**
이 때, 지연 초기화로 구현할 경우 스레드 안정성 또한 고려해야한다.
2. **성능을 높이기 위해 핵심필드를 제외하고 hashCode를 계산하지 않는다.**
속도는 빨라 지더라도 hash 품질이 나빠져 해시 테이블 성능을 오히려 떨어트린다.
필드 생략한다면 해당 영역에 수많은 인스턴스가 단 몇개의 해시코드로 집중되어 해시 테이블의 
O(1) 속도가 링크드리스트마냥 선혀으로 느려진다. 
3. **HashCode가 반환하는 값의 생성규칙을 API 사용자에게 공표하지않는다.**
클라이언트가 hashCode값에 의지해 코드를 짜지 않기 위함이다.

### 결론

서로 다른 인스턴스라면 다른 해시코드가 나오게 구현해야된다. 

롬북에서 제공하는 equalsandhashcode 어노테이션이나 Autovalue를 사용하면 

알아서 잘 자동으로 만들어준다.
