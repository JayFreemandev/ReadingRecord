# Item56, 자바 doc

## API문서화에 대해 다루는 아이템이다

1. 직렬화할 수 있는 클래스 경우 직렬화 형태 문서화시켜라
2. 기본 생성자에 주석 달지마라 헷갈린다.
3. HOW가 아닌 WHT을 써라
<br>

열거 타입도 문서화 해야된다.

```java
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe */
    WOODWIND,
    /** Brass instruments, such as french horn and trumpet */
    BRASS,
    /** Percussion instruments, such as timpani and cymbals */
    PERCUSSION,
    /** Stringed instruments, such as violin and cello */
    STRING;
}
```
<br>

어노테이션 타입을 문서화

- 필드, 멤버들에도 모두 주석을 달아야 한다.
- 필드 설명은 명사구로 한다.
- 어노테이션 타입의 요약 설명은 프로그램 요소에 이 어노테이션을 단다는 것이 어떤 의미인지를 설명하는 동사구로 한다.

```java
/**
 * Indicates that the annotated method is a test method that
 * must throw the designated exception to pass.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
     * The exception that the annotated test method must throw
     * in order to pass. (The test is permitted to throw any
     * subtype of the type described by this class object.)
     */
    Class<? extends Throwable> valueO;
}
```
<br>

현재 업무중에는 메소드와 인터페이스, 클래스와 필드까지는 문서화를 남기고있다. 구현체 경우는 남겨두는편이다 인터페이스에 잘 문서화 해두니 바로 찾아가기가 쉽고 덜 지저분해보였다.
