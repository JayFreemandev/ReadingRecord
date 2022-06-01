# Item23, 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 태그 달린 클래스

두 가지 이상의 의미를 표현할 수 있으며, 그중 현재 표현하는 의미를 태그 값으로 알려주는 클래스를 본적이 있을 것 이다. 다음은 원과 사각형을 표현할 수 있는 클래스이다.

태그 달린 클래스의 **단점**은 아래와 같다.

- 여러 구현이 하나의 클래스에 혼합돼있어서 가독성이 나쁘다.
- 다른 의미를 위한 코드도 항상 함께하니 메모리도 많이 사용한다.
- 필드들을 final로 선언하려면 해당 의미에 쓰이지 않는 필드들까지 생성자에서 초기화해야한다.
- 새로운 의미를 추가하려면 모든 switch문을 찾아 새 의미를 처리하는 코드를 추가해야한다.
- 즉, 장황하고 오류를 내기 쉽고 비효율적이다.

```java
// 코드 23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다! (142-143쪽)class Figure {
    enum Shape {RECTANGLE, CIRCLE};

// 태그 필드 - 현재 모양을 나타낸다.final Shape shape;

// 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.double length;
    double width;

// 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.double radius;

// 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

// 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

### 태그 달린 클래스 -> 계층 클래스 변경

계층구조의 루트가 될 추상 클래스를 정의하고, 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 정의 선언한다.

```java
abstract class Figure {
    abstract double area();
}
```

루트 클래스를 확장하는 구체 클래스를 의미별로 하나씩 정의한다.

```java
class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

타입 사이의 자연스러운 계층 관계를 반영할 수 있어서 유연성은 물론 컴파일타임 타입 검사 능력을 높여준다는 장점도 있다. 위의 코드에서 정사각형을 추가하려면 Rectangle 클래스를 확장하는 Square 클래스를 만들어주면 된다.

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

이번 아이템은 업무를 하면서 태그 달린 클래스를 본적도없고 실무와는 거리가 있는 내용이였다.
