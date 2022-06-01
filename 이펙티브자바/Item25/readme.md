# Item25, 톱레벨 클래스는 한 파일에 하나만 담자

## 이번 아이템 내용은 라이트하다.

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다. 하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위다. 이렇게 하면 한 클래스를 여러 가지로 정의할 수 있으며, 그중 *어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하냐에 따라 달라지기 때문*이다.

다음 소스 파일은 Main 클래스를 하나 담고 있고, Main 클래스는 다른 톱레벨 클래스 2개(Utensil과 Dessert)를 참조한다.

```java
public class Main {
   public static void main(String[] args) {
      System.out.println(Utensil.NAME+ Dessert.NAME);
   }
}
```

집기(Utensil)와 디저트(Dessert) 클래스가 Utensil.java라는 한 파일에 정의되어 있다고 해보자.

**[ 두 클래스가 한 파일(Utensil.java)에 정의되어있다. - 따라하지 말 것! ]**

```java
class Utensil {
   static final StringNAME= "pot";
}

class Dessert {
   static final StringNAME= "pie";
}
```

물론 Main을 실행하면 pancake를 출력한다.

이제 우연히 똑같은 두 클래스를 담은 Dessert.java라는 파일을 만들었다고 해보자.

**[ 두 클래스가 한 파일(Dessert.java)에 정의되어있다. - 따라하지 말 것! ]**

```java
class Dessert {
   static final StringNAME= "pie";
}

class Utensil {
   static final StringNAME= "pot";
}
```

책에서는 다음과 같이 정의를 했는데 출력한 실습 결과는 달랐다.

- javac Main.java Dessert.java 를 하면 컴파일 오류가 발생
- javac Main.java | javac Main.java Utensil.java 를 실행하면 “pancake” 출력
- javac Main.java Dessert.java 를 실행하면 “potpie” 출력

그래서 결과를 토대로 수정해서 작성했다.

## **javac Main.java Utensil.java Dessert.java - 컴파일 오류**

운 좋게 javac Main.java Utensil.java Dessert.java 명령으로 컴파일한다면 컴파일 오류가 나고 Untensil과 Dessert 클래스를 중복 정의했다고 알려줄 것이다.

1. 컴파일러는 가장 먼저 Main.java를 컴파일한다.
2. 그 안에서 (Dessert 참조보다 먼저 나오는) Utensil 참조를 만나면 Utensil.java 파일을 살펴 Utensil과 Dessert를 모두 찾아낼 것이다.
3. 그런 다음 컴파일러가 두 번째 명령줄 인수로 넘어온 Dessert.java를 처리하려 할 때 같은 클래스의 정의가 이미 있음을 알게 된다.

![https://blog.kakaocdn.net/dn/LhdfH/btrqvUAKwWp/FQP4888MF6cIvXxjY2mEVK/img.png](https://blog.kakaocdn.net/dn/LhdfH/btrqvUAKwWp/FQP4888MF6cIvXxjY2mEVK/img.png)

javac Main.java Utensil.java Dessert.java&nbsp;

## **javac Main.java Utensil.java - “pancake” 출력**

한편 javac Main.java Utensil.java 명령으로 컴파일하면 Dessert.java 파일을 작성하기 전처럼 pancake를 출력한다.

1. 컴파일러는 가장 먼저 Main.java를 컴파일한다.
2. Utensil 참조를 먼저 만나서 Utensil.java 파일을 살펴보고 해당 파일 안에서 찾고자하는 변수([Utensil.name](http://utensil.name/), [Dessert.name](http://dessert.name/))를 모두 찾았기 때문에 컴파일 오류 없이 “pancake”를 출력할 것이다.

**[ javac Main.java Utensil.java 명령 실행 후 생성된 Main.class 파일 ]**

```arduino
public class Main {
    public Main() {
    }

    public static void main(String[] var0) {
        System.out.println("pancake");
    }
}

```

## **javac Main.java Dessert.java - “potpie” 출력**

그러나 javac Main.java Dessert.java 명령으로 컴파일하면 “potpie”를 출력한다. 이처럼 컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라지므로 반드시 바로잡아야 할 문제다.

**[ javac Main.java Dessert.java 명령 실행 후 생성된 Main.class 파일 ]**

```arduino
public class Main {
    public Main() {
    }

    public static void main(String[] var0) {
        System.out.println("potpie");
    }
}

```

## **톱레벨 클래스(Utensil, Dessert)를 분리하자**

해결책은 아주 간단하다. 단순히 톱레벨 클래스(Utensil, Dessert)을 서로 다른 소스 파일로 분리하면 그만이다.

굳이 한 파일에 담고 싶다면 정적 맴버 클래스(아이템 24)를 사용하는 방법을 고민해볼 수 있다. 다른 클래스에 딸린 부차적인 클래스라면 정적 멤버 클래스로 만드는 쪽이 일반적으로 나을 것이다. 읽기 좋고, private으로 선언하면 접근 범위도 최소로 관리할 수 있기 때문이다.

**[ 톱 레벨 클래스들을 정적 멤버 클래스로 바꿔본 모습 ]**

```arduino
public class Test {
   public static void main(String[] args) {
      System.out.println(Utensil.NAME+ Dessert.NAME);
   }

   private static class Utensil{
      static final StringNAME= "pan";
   }

   private static class Dessert{
      static final StringNAME= "cake";
   }
}
```

## **핵심 정리**

교훈은 명확하다. 

**소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자.** 

이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어내는 일은 사라진다. 

소스 파일을 어떤 순서로 컴파일하든 바이너라 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.

좋은 예제로 작성된 글들이 많아서 자료를 참고하기 편했다.

소스 파일 하나에는 반드시 중요한 클래스나 인터페이스 하나만을 담자

당연한 내용이기에 라이트하게 읽고 넘어갈수있다.
