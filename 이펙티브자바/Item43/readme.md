아이템 43. 람다보다는 메서드 참조를 사용하라
자바는 함수 객체를 람다 보다 더 간결하게 생성하는 방법을 제공한다. **바로 메서드 참조(method reference)**가 그것이다.

아래 코드는 Map 타입 객체에 첫 번째 인자인 key(여기서는 숫자 1)를 매핑한다.
key가 Map에 존재한다면 두 번째 인자인 value(여기서는 2)와 기존의 key에 매핑되는 value(여기서는 1)를 합한다.
존재하지 않는다면, 새롭게 key, value 쌍을 추가한다.
Map<Integer, Integer> map = new HashMap<>();
map.put(1, 1);
map.put(3, 5);
map.put(5, 8);

map.merge(1, 2, (count, incr) -> count + incr); // {1=3, 3=5, 5=8}
map.merge(2, 1, (count, incr) -> count + incr); //{1=3, 2=1, 3=5, 5=8}
위 코드에서 처럼 람다는 보일러플레이트(boilerplate)가 존재한다.

파라미터인 count와 incr 는 많은 공간을 차지한다.

위 더하기 함수식은 자바 8 이후부터 int의 박싱 타입인 Integer의 정적 메서드인 sum을 쓰는 것과 동일하다.

이것을 메서드 참조를 이용하여 간결하게 바꾸어보자

map.merge(1, 2, Integer::sum);		
무조건 메서드 참조가 좋은건 아니다. 어떤 람다에서는 파라미터 이름이 곧 _문서_이기도 하다.

따라서, 람다가 메서드 참조보다 더 길어도, 유지보수나 가독성면에서 도움이 될 수도 있다.

메서드 참조는 주로 정적 메서드를 사용하지만, 그외에도 4가지 종류의 메서드 참조가 있다.

메서드 레퍼런스 타입	예제	동일한 표현식을 람다로 바꾸면?
정적(Static)	Integer::parseInt	str -> Integer.parseInt(str)
바운드(Bound)	Instant.now()::isAfter	Instant then = Instant.now(); t -> then.isAfter(t);
언바운드(UnBound)	String::toLowerCase	str -> str.toLowerCase();
클래스 생성자	TreeMap<K,V>::new	() -> new TreeMap<K,V>();
배열 생성자	int[]::new	len -> new int[len];
요약하자면, 메서드 참조는 람다보다 더 간결하다.

메서드 참조를 사용할 때 코드가 더 명확하고 짧아진다면, 메서드 참조를 사용하라

그게 아니라면, 람다식을 사용하라.
