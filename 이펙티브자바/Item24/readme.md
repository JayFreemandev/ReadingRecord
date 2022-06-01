# Item24, 멤버 클래스는 되도록 static으로 만들어라 *

## **중첩 클래스(nested class)**

중첩 클래스(nested class)란 다른 클래스 안에 정의된 클래스를 말한다. 중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

### **중첩 클래스 종류**

중첩 클래스는 네 가지이다. 정적 멤버 클래스를 제외한 나머지 세 가지는 내부 클래스(inner class)에 해당한다.

1. 정적 멤버 클래스
2. (비정적) 멤버 클래스
3. 익명 클래스
4. 지역 클래스

각각의 중첩 클래스를 언제 그리고 왜 사용해야 하는지 이야기한다.

## **정적 멤버 클래스**

정적 멤버 클래스는 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고는 일반 클래스와 똑같다. 정적 멤버 클래스는 다른 정적 멤버와 똑같은 접근 규칙을 적용받는다. private으로 선언하면 바깥 클래스에서만 접근할 수 있는 식이다.

흔히 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미 클래스로 쓰인다. 계산기가 지원하는 연산 종류를 정의하는 열거 타입을 예로 생각해보자. Operation 열거 타입은 Calculator 클래스의 public 정적 멤버 클래스가 되어야 한다.

- 그러면 Calculator의 클라이언트에서 Calculator.Operation.PLUS나 Calculator.Operation.MINUS 같은 형태로 원하는 연산을 참조할 수 있다.

```java
@FunctionalInterface
interface Operator{
	int operate(int x, int y);
}

public class Calculator {
   static enum Operation{
			PLUS((x,y) -> (x+y)),
			MINUS((x,y) -> (x-y));

      private final Operator operator;

      private Operation(Operator operator) {
         this.operator = operator;
      }

      int operate(int x, int y){
         return this.operator.operate(x,y);
      }
   }

   public static void main(String[] args) {
      int operate = Operation.PLUS.operate(1, 2);
      System.out.println("operate = " + operate);

      int operate2 = Operation.MINUS.operate(1, 2);
      System.out.println("operate2 = " + operate2);
   }
}

```

### **바깥 클래스의 한 부분이라면 private으로**

private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분을 나타낼 때 쓴다.

키와 값을 매핑시키는 Map 인스턴스를 생각해보자. 많은 Map 구현체는 각각의 키-값 쌍을 표현하는 엔트리(Entry) 객체들을 가지고 있다. 모든 엔트리가 맵과 연관되어 있지만 엔트리의 메서드들(getKey, getValue, setValue)은 맵을 직접 사용하지는 않는다.

- 따라서 엔트리를 비정적 멤버 클래스로 표현하는 것은 낭비고, private 정적 멤버 클래스가 가장 알맞다.

## **비정적 멤버 클래스**

비정적 멤버 클래스의 인스턴스와 바깥 인스턴스 사이의 관계는 멤버 클래스가 인스턴스화될 때 확립되며, 더 이상 변경할 수 없다.

- 이 관계는 바깥 클래스의 인스턴스 메서드에서 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지는 게 보통이다.
- 드물게 직접 **바깥 인스턴스의 클래스.new. Member Class(args)**를 호출해 수동으로 만들기도 한다.
- 예상할 수 있듯, 이 관계 정보는 비정적 멤버 클래스의 인스턴스 안에 만들어져 메모리 공간을 차지하며, 생성 시간도 더 걸린다.

```arduino
public class FoodTruck {
   private final String name;

   public FoodTruck(String name) {
      this.name = name;
   }

   // 비정적 멤버 클래스와 바깥 클래스 연관 관계
   public void orderPizza(String pizzaName) {
      Pizza pizza = new Pizza(pizzaName);
      pizza.printOrderStatus();
   }

   public String getName() {
      return name;
   }

	 // 비정적 멤버 클래스
   private class Pizza {
      private final String pizzaName;

      public Pizza(String pizzaName) {
         this.pizzaName = pizzaName;
      }

      public void printOrderStatus() {
         // 정규화된 this - 바깥 클래스의 인스턴스 메서드 사용
         System.out.println("["+ FoodTruck.this.getName()+ " truck] order success: "+ pizzaName +" pizza");
      }
   }

   public static void main(String[] args) {
      FoodTruck foodTruck = new FoodTruck("loosie");
      foodTruck.orderPizza("combination");
   }
}

```

## **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여 정적 멤버 클래스로 만들자**

정적과 비정적의 구문상 차이는 단지 static이 붙어 있고 없고 뿐이지만, 의미상 차이는 의외로 꽤 크다.

### **static을 생략하면 바깥 인스턴스로의 숨은 외부 참조를 갖게 된다.**

비정적 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다. 그래서 비정적 멤버 클래스의 인스턴스 메서드에서 정규화된 this를 사용해 바깥 인스턴스의 메서드를 호출하거나 바깥 인스턴스의 참조를 가져올 수 있다

- 정규화된 this란 클래스명.this 형태로 바깥 클래스의 이름을 명시하는 용법을 말한다.

이 시간과 공간이 더 소비되는 것이다. 더 심각한 문제는 가비지 컬렉션이 바깥 클래스의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다는 점이다.

### **바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로**

따라서 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다. 비정적 멤버 클래스는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.

멤버 클래스가 공개된 클래스의 public이나 protected 멤버라면 정적이냐 아니냐는 두 배로 중요해진다. 멤버 클래스 역시 공개 API가 되니, 혹시라도 향후 릴리스에서 static을 붙이면 하위 호환성이 깨진다.

말이 너무 어려워서 이해못했다.

## **익명 클래스**

익명 클래스는 당연히 이름이 없다. 또한 바깥 클래스의 멤버도 아니다. 멤버와 달리, 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다. 코드는 어디서든 만들 수 있다.

- 오직 비정적인 문맥에서만 사용될 떄만 바깥 클래스의 인스턴스를 참조할 수 있다.
- 정적 문맥에서는 바깥 클래스 인스턴스를 참조할 수 없을 뿐더러 상수 변수 이외의 정적 멤버는 가질 수 없다. 즉, 상수 표현을 위해 초기화된 final 기본 타입과 문자열 필드만 가질 수 있다.

익명 클래스는 응용하는 데 제약이 많은 편이다.

- 선언한 지점에서만 인스턴스를 만들 수 있고, instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 여러 인터페이스를 구현할 수 없고, 인터페이스를 구현하는 동시에 다른 클래스를 상속할 수도 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.
- 익명 클래스는 표현식 중간에 등장하므로 짧자 않으면 가독성이 떨어진다.

가독성에 관련한 문제는 람다가 등장하고 나서는 어느정도 해결이 되었다. 람다를 지원하기 전에는 즉석에서 작은 함수 객체나 처리 객체(process object)를 만드는 데 익명 클래스를 주로 사용했다. 그리고 또 다른 주 쓰임은 정적 팩터리 메서드를 구현할 때다.

```java
enum Type{
	 PLUS,MINUS;
}

interface Operator{
   int plus();
   int minus();
}

public class Calculator {
   private int num1;
   private int num2;

   public Calculator(int num1, int num2) {
      this.num1 = num1;
      this.num2 = num2;
   }

   public int operate(Type type){
      Operator operator = new Operator() {
         @Override public int plus() {
            return num1 + num2;
         }
         @Override public int minus() {
            return num1 - num2;
         }
      };

      switch (type){
         casePLUS: return operator.plus();
         caseMINUS: return operator.minus();
         default: throw new AssertionError(type);
      }
   }

   public static void main(String[] args) {
      Calculator calculator = new Calculator(5, 2);
      int plus = calculator.operate(Type.PLUS);
      System.out.println("plus = " + plus);
      int minus = calculator.operate(Type.MINUS);
      System.out.println("minus = " + minus);
   }
}

```

내가 이해한 익명 클래스란 이름이 없고 쓰이는 시점에 선언과 동시에 만들어진 클래스다. 

가독성이 안좋았으나 람다가 나오고나서 가독성 단점이 보완된 클래스다.

## **지역 클래스**

지역 클래스는 네 가지 중첩 클래스 중 가장 드물게 사용된다. 지역 클래스는 지역변수를 선언할 수 있는 곳이면 실질적으로 어디서든 선언할 수 있고, 유효 범위도 지역변수와 같다. 다른 세 중첩 클래스와의 공통점도 하나씩 가지고 있다.

- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다.
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다.
- 정적 멤버는 가질 수 없으며, 가독성을 위해 짧게 작성해야 한다

```arduino
public class Practice {
   private String name;

   public Practice(String name) {
      this.name = name;
   }

   public void foo(String name){
      Local local = new Local(name);
      local.localPrint();
   }

	 // 지역 클래스
   class Local {
      private String name;

      public Local(String name) {
         this.name = name;
      }

      public void localPrint() { // 역 클래스 안에서  바깥 인스턴스 참조
         System.out.println("outerClass: " + Practice.this.name + ", inner local: " + name);
      }
   }

   public static void main(String[] args) {
      Practice practice  = new Practice("outClass");
      practice.foo("inner");
   }
}

```

지역 클래스또한 사용될 일이 거의 없어보인다. 

## 결론

중첩 클래스에는 네 가지가 있으며, 각각의 쓰임이 다르다.

- 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 **멤버 클래스**로 만든다.
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 **비정적**으로, 그렇지 않으면 **정적**으로 만들자.
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 **익명 클래스**로 만들고,
- 그렇지 않다면 **지역 클래스**로 만들자.

24번 아이템은 이해하는데 다시 읽어보고 관련 자료를 더 찾아봐야할거같다. 이해하기 어렵다.
