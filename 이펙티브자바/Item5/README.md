# Item5, 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

[https://www.youtube.com/watch?v=0yUxPUXS1pM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=6](https://www.youtube.com/watch?v=0yUxPUXS1pM&list=PLfI752FpVCS8e5ACdi5dpwLdlVkn0QgJJ&index=6)

## 대부분의 클래스는 여러 리소스에 의존한다.

**사용하는 자원에 따라 동작이 달라지는 클래스**에는 ~정적 유틸리티 클래스~나 ~싱글턴 방식~이 적합하지 않다. 이 조건을 만족하는 간단한 패턴은, **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**이다.

### 정적 유틸리티

```java
public class SpellChecker {
	private static final Lexicon dictionary = new LexiconDictionary();

	private static boolean isValid(String word) { ... }
	private SpeckChecker() { } // 객체 생성 방지
}
```

인스턴스를 만들 필요가 없기 때문에 private한 생성자를 만들어주고 static 메소드들이 존재한다.

여기서 dictionary가 static final인 이유는 static method들이 사용해야하는 정적 유틸리티로 만들었

기 때문이다.

### 싱글턴

```java
public class SpellChecker {
	private final Lexicon dictionary = new LexiconDictionary();

	public static SpellChecker INSTANCE = new SpellChecker(...);
	private SpellChecker(...) { }

	public boolean isValid(String word) { ... }
}
```

위와 같은 경우 사전(dictionary)이 하나로 고정이 되기 때문에 좋은 방법이 아닙니다.

SpellChecker가 여러 종류의 사전을 사용할 수 있도록 하려면 final을 제거하고 dictionary를 변경하는 

메서드를 추가할 수 있지만, 오류를 내기 쉬우며 멀티스레드 환경에서는 쓸 수 없다. 사전만해도 지금

은 LexiconDictionary 사전만 있지만 영어, 일본어, 중국어, 불어등 여러 사전이 추가되야할경우 문제

가 생긴다. 그렇다고 final 제거해버리면 위 설명처럼 멀티스레드때 사용할수가없다.

대신 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주도록 하면 된다.

### 생성자 의존 객체 주입

```java
public class SpellChecker {
	private final Lexicon dictionary;

	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public boolean isValid(String word) { ... }
}
```

1. 불변을 보장하기 때문에 여러 클라이언트가 의존 객체들을 안심하고 공유한다.
2. 자원이 몇 개든 의존 관계가 어떻든 상관없이 작동한다.

정적 유틸리티와 싱글턴의 단점을 보완하기에 유연성과 재사용성, 테스트에는 유리해진다.

의존성이 많아지면 코드가 어지러워지는 단점이 존재한다.
