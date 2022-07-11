# Item58, for 말고 for each 사용해라

### iterator나 int=0과 같은 인덱스를 사용하기에 실수할 여지가 있고 잘못된 변수 사용으로 오류가 발생할 수 있음

새로 알게된 사실은 아래와 같다.
**아래 두가지 상황에서는 for-each 사용이 불가**

## 필터링

컬렉션을 순회하면서 선택된 엘리먼트를 제거해야되는 상황에는 명시적으로 iterator를 사용해야한다. remove 메소드를 호출해야하기 때문이다.

## foreach 문장을 사용하여 리스트 자체를 수정

해야할 경우에는 ConcurrentModificationException이 발생한다. 순회중인 상태에서 자기 자신에 대한 데이터 변경이 불가하기 때문에 iterator를 사용할 수 밖에 없다.

```java
// some list
List<String> list = new ArrayList<>();
list.add("a"); list.add("b"); list.add("c");

for (Iterator<String> iterator = list.iterator(); iterator.hasNext(); ) {
    String element = iterator.next();
    if(element.equals("a")) {
        iterator.remove();
    }
    // do something
}
```

Java 8에는 **removeIf** 메소드를 제공하기때문에 컬렉션 명시가 필요 없어졌다. 

```java
// Lambda
list.removeIf(s -> s.equals("a"));

// 위 코드를 풀어서 표현하면,
list.removeIf(new Predicate<String>() {
    @Override
    public boolean test(String s) {
        return s.equals("a");
    }
});
```

외에는 병렬 순회가 존재하는데. 마지막으로 여러 개의 컬렉션을 변ㅇ렬적으로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해서 엄격하고 명시적으로 제어해야한다.

```java
num Suit {
    CLUB, DIAMOND, HEART, SPADE
}

enum Rank {
    ACE, DEUCE, THREE, FOUR, 
    // ... 생략 
    QUEEN, KING
}

List<Card> deck = new ArrayList<>();
List<Suit> suits = Arrays.asList(Suit.values());
List<Rank> ranks = Arrays.asList(Rank.values());
for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
    for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); ) {
        // next 메서드가 숫자(suit) 하나당 불려야 하는데
        // 카드(Rank) 하나당 불리고 있다.
        deck.add(new Card(i.next(), j.next()));
    }
}
```

반복자의 `next` 메서드가 Suit를 탐색하는 루프 1회마다 불려야 하는데, Rank를 탐색하는 루프가 수행될 때마다 불리고 있어서 결국 모든 숫자(Suit)를 탐색하게 되어 `NoSuchElementException`이 발생

위에서 살펴본 상황에 속하게 될 경우 일반적인 for 문을 사용해야하고 `for-each` 문은 컬렉션과 배열은 물론 `Iterable` 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.
