# Item43, 람다보다는 메소드 참조를

Person person1 = new Person("김OO");
Person person2 = new Person("이OO");
Person person3 = new Person("김OO");
Person person4 = new Person("최OO");

List<Person> people = Arrays.asList(person1, person2, person3, person4);

위 코드와 같이 여러 사람들이 있을때, 각 이름의 사람 수를 나타내는 Map을 구하고 싶다고 하자.

그럼 다음과 같이 구현할 수 있다.

//람다 방식
Map<String, Integer> counts = new HashMap<>();
for (Person person : people) {
    counts.merge(person.getName(), 1, (existingValue, providedValue) -> existingValue + providedValue);
}
//counts = {이OO=1, 최OO=1, 김OO=2}

하지만 existingValue나 providedValue 는 크게 하는 일 없이 공간을 많이 차지한다.

메서드 참조 방식으로 더욱 간결하게 나타낼 수 있다.

//메서드 참조 방식
Map<String, Integer> counts = new HashMap<>();
for (Person person : people) {
    counts.merge(person.getName(), 1, Integer::sum);
}
//counts = {이OO=1, 최OO=1, 김OO=2}

이번에는 김씨만 모으고 싶다고 하자.

먼저 익명 클래스 방식을 보자.

//익명 클래스 방식
List<Person> onlyKim = people.stream()
        .filter(new Predicate<Person>() {
            @Override
            public boolean test(Person person) {
                return person.getName().startsWith("김");
            }
        })
        .collect(toList());

아래와 같이 람다를 이용하면 더 간결하게 나타낼 수 있다.

//람다 방식
List<Person> onlyKim = people.stream()
        .filter(person -> person.getName().startsWith("김"))
        .collect(toList());

여기에서 person -> person.getName().startsWith("김") 에 해당하는 로직을 분리하고, 메서드 참조를 이용하면 더 간결하게 나타낼 수 있다.

//메서드 참조 방식
List<Person> onlyKim = people.stream()
        .filter(Person::isLastNameKim)
        .collect(toList());

이처럼 기능을 잘 드러내는 메서드명을 지어주면 가독성이 더 좋아진다.

그렇다고 해서 메서드 참조 방식이 람다 방식보다 무조건 나은 건 아니다. 다음의 경우엔 람다 방식이 더 가독성이 좋을 수 있다.

- 메서드와 람다가 같은 클래스 안에 있을때
    - public class SoLooooooooongName { public void function() { //람다 방식 execute(() -> action()); //메서드 참조 방식 execute(SoLooooooooongName::action); } private void execute(Supplier<?> supplier) { ... } public static Object action() { ... }
    }
- Function.identity() 보다는 (x -> x)가 더 짧고 명확하다

참고로, 메서드 참조 유형은 5가지가 있다.

[Untitled](https://www.notion.so/4dc2c99f73d74fcda247e168cef8c2eb)

### **결론**

가독성을 위해 람다 대신 메서드 참조 방식을 사용하자. 하지만 람다 방식이 더 가독성이 좋다면 람다 방식을 쓰자.
