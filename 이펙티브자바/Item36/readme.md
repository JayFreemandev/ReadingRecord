# Item36. 비트 필드 대신 EnumSet을 사용

## 오래전 비트 필드를 사용한 방법

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;
    public static final int STYLE_ITALIC = 1 << 1;
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;

    public void applyStyles(int styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
    }
}
```

## 실행결과

```java
Applying styles 8 to text

Process finished with exit code 0
```

집합 연산을 통해서 하나의 값으로 출력하려고하는 오래된 방법이다.

### EnumSet을 이용한 현대적 방법

```java
ublic class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }

    // 사용 예
    public static void main(String[] args) {
        Text text = new Text();
        text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
    }
}
```

## 실행 결과

```java
Applying styles [BOLD, ITALIC] to text

Process finished with exit code 0
```

집합 형태로 사용할 수 있다고해도 비트 필드를 사용할 이유가 없다. (업무중 아직 비트연산자로 출력 해본적 없음)

Reference

폭간
