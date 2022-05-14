# Item33, 타입 안정 이종컨테이너를 고려하라

제네릭은 Set<E>, Map<K,V>와 같이 단일원소를 필요로 하는곳에서도 많이 쓰인다.

위와 같이 Set은 하나의 엘리먼트, 맵은 하나의 키와 하나의 벨류, 하나의 컨테이너에서 매개변수화

할 수 있는 타입의 수가 제한된다.

```java
Set<Integer> intSet = new HashSet<>();
Map<Integer, String> map = new HashMap<>();
```

더 유연한 수단이 필요할 때도 종종 있다.

```java
Set<Object> magic = new HashSet<>();
```

이렇게 사용하면 모든 타입을 다 받을수있지만 컨테이너에서 얻은 객체가 어떤 타입인지 알 수 없다.

컴파일시에는 슝 하고 통과할꺼고 런타임때 오류가 터질 수 있다. 이럴때 타입 안정 이종컨테이너라

는 패턴을 고려하자.

### 타입 안전 이종 컨테이너 패턴

컨테이너 대신 키를 매개변수화한 다음,  컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제

공하면 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해준다.

이러한 설계 방식을 **타입 안전 이종 컨테이너 패턴**이라 한다.

예제로 타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스를 생각해본다. 

각 타입의 Class 객체를 매개변수화한 키 역할로 사용하면 되는데, 이 방식이 동작하는 이유는 class

의 클래스가 제네릭이기 때문이다. class 리터럴의 타입은 Class가 아닌 **Class<T>**이다. 

따라서 String.class의 타입은 Class<String>이고 Integer.class는 Class<Integer>인 식이다. 

한편, 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴

을 타입 토큰(type token)이라 한다. 

Favorites 클래스의 API는 아래와 같이 단순하다. 키가 매개변수화되었다는 점만 빼면 일반 맵처럼 보

일 정도다. 클라이언트는 즐겨찾기를 저장하거나 얻어올 때 Class 객체를 알려주면 된다.

(로키산맥글 설명)

```java
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> T getFavorite(Class<T> type);
}
... // 코드생략
public static void main(String[] args) {
    Favorites favorites = new Favorites();
    favorites.putFavorite(String.class,"morning");
    favorites.putFavorite(Integer.class, 0xcafebabe);
    favorites.putFavorite(Class.class, Favorites.class);

    String favoriteString = favorites.getFavorite(String.class);
    Integer favoriteInteger = favorites.getFavorite(Integer.class);
    Class<?> favoriteClass = favorites.getFavorite(Class.class);

    System.out.printf("%s %x %s", favoriteString, favoriteInteger, favoriteClass.getName());
}
```

Favorite의 인터스턴스는 안전하다고 볼 수 있다. String을 요청했는데 Integer를 반환할수없다.

일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수가 있다. 

```java
Map<Class<?>, Object> favorites = new HashMap<>();

favorites.put(String.class, "안녕!");

favorites.put(Integer.class, 123);

String item = (String) favorites.get(String.class);
```

key에는 타입, value에는 값이 들어가기 때문에 key를 통해서 value의 자료형을 유추할 수 있다.

```java
Map<Class<?>, Object> favorites = new HashMap<>();

favorites.put(Integer.class, "안녕!"); 

Integer item = (Integer) favorites.get(Integer.class);

*// ClassCastException 발생*
```

하지만 인스턴스 저장 시 key와 인스턴스의 자료형을 다르게 저장할 경우 

ClassCastException이 발생하기 때문에 타입 안정성이 떨어진다.

# 결론

컬렉션 API로 대표되는 일반적인 제네릭형태는 한 컨테이너가 다룰수 있는 매개변수의 수가 고정적

이다.하지만 컨테이너자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨

테이너를 만들 수 있다.타입 이종 컨테이너는 `Class`를 키로 사용하며, 이런 식으로 쓰는 `Class` 객

체를 타입 토큰이라 한다.

또한 직접 구현한 키 타입도 쓸 수 있다. 예컨데 DB의 행(컨테이너)을 표현한 `DatabaseRow` 타입에는 

제네릭 타입인 `Column<T>`를 사용할 수 있다.
