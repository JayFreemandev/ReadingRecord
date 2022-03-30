# Item50, 방어적복사

Date: March 27, 2022

## 얕은 복사란?

- 객체를 복사할 때, 해당 객체만 복사하여 새 객체를 생성한다.
- 복사된 객체의 인스턴스 변수는 **원본 객체의 인스턴스 변수와 같은 메모리 주소를  참조**한다.
- 따라서, 해당 메모리 주소의 값이 변경되면 원본 객체 및 복사 객체의 인스턴스 변수 값은 같이 변경된다.

## 깊은 복사란?

- 객체를 복사할 때, 해당 객체와 인스턴스 변수까지 복사하는 방식.
- **전부를 복사해 새 주소에 담기 떄문에 참조를 공유하지않는다.**

```java
public class Jaeyun {
    private int money;
    private String menu;

    public Jaeyun(String menu, int money) {
        this.menu = menu;
        this.money = money;
    }

    public void orderFood(String menu){
        this.menu = menu;
    }

    public void spend(int money){
        this.money -= money;
    }
}
```

```java
Jaeyun jaeyun = new Jaeyun("징거버거", 8000);
Jaeyun jaeyunCopy = jaeyun;

jaeyun.orderFood("싸이버거");
jaeyun.spend(5500);
```

징거버거와 8000원을 가진 재윤 객체를 생성하고 재윤카피에 이 인스턴스를 복사한다.

메뉴를  싸이버거로 바꾸고 지출을 5500원으로 변경한다. 

그러면 현재 money -= money 이기때문에 8000 - 5500원일까? 

둘다 싸이버거에 5500원으로 변경되었다.

```java
System.out.println(jaeyun); // Jaeyun@75b84c92
System.out.println(jaeyunCopy); // Jaeyun@75b84c92
```

재윤카피는 값을 복사한게 아니고 참조값을 복사했기 때문이다.  

재윤 인스턴스를 생성했을때 stack에는 참조값, heap에는 실제값이 올라가게된다.

![Untitled](https://user-images.githubusercontent.com/72185011/160834423-fba9b763-90d4-4331-b235-fe66cf1b34fb.png)

아까와 같이 메뉴와 가격을 수정한다면 같은 객체를 참조하고있는 재윤카피또한 영향을 받는다.

jaeyun.orderFood("싸이버거");
jaeyun.spend(5500); 

![Untitled 1](https://user-images.githubusercontent.com/72185011/160834481-63ced8a8-a660-4d41-aa2e-c87b61cf7bc5.png)

여기까지가 얕은 복사가 이루어지는 과정이다.

## 깊은 복사

깊은복사를 구현하는 방법은 크게 3가지 방법이 존재한다.

1. 복사 생성자 또는 복사 팩터리를 이용하여 복사한다.
2. 직접 객체 생성하여 복사한다.
3. Cloneable을 구현하여 clone()을 사용한다.
하지만 clone 사용시 재정의가 필요한데 final 인스턴스 또는 배열이 아닌 경우 사용을 권장하지 않는다.

따라서 cloneable을 구현하여 clone()을 재정의하기보다는 복사 생성자나 복사 팩터리를 이용하여 

깊은 복사하는게 좋다.

### 복사생성자 or 복사팩터리

```java
public Jaeyun(Jaeyun jaeyun) {
        this.menu = jaeyun.menu;
        this.money = jaeyun.money;
    }

public static Jaeyun newInstance(Jaeyun jaeyun){
    Jaeyun jae = new Jaeyun(jaeyun);
    jae.menu = jaeyun.menu;
    jae.money = jaeyun.money;
    return jae;
}
```

### 직접 객체를 생성하여 복사

```java
				Jaeyun jaeyun = new Jaeyun("징거버거", 8000);
        Jaeyun jaeyunCopy = new Jaeyun();
        jaeyunCopy.setMenu(jaeyun.getMenu());
        jaeyunCopy.setMoney(jaeyun.getMoney());
        
        jaeyun.orderFood("싸이버거");
        jaeyun.spend(5500);
```

이제 결과를 보면 서로 다른 값을 가지는 깊은 복사가 완료된것을 볼수있다.

```java
System.out.println(jaeyun.getMenu());
System.out.println(jaeyun.getMoney());

System.out.println(jaeyunCopy.getMenu());
System.out.println(jaeyunCopy.getMoney());
```

싸이버거
5500

징거버거
8000

![Untitled 2](https://user-images.githubusercontent.com/72185011/160834509-f5f62357-a915-42a5-9e35-2136c8b8002a.png)

 jaeyun.orderFood("싸이버거");
 jaeyun.spend(5500);
 
![Untitled 3](https://user-images.githubusercontent.com/72185011/160834543-0307aa1c-29fb-4f19-b67e-15b07342648b.png)

### 방어적 복사

깊은 복사인경우 객체를 담는 몸통과 객체 안에 필드 주소들을 모두 새롭게 만드는것이다. 

즉 원본 객체와의 관계를 완전히 끊어내는 복사라고 볼수있다. 하지만 내가 객체의 10%만 복사하고

싶은데 깊은 복사인경우 set을 사용하여 100%를 가져와야한다. 나머지 90%는 낭비인것이다.

방어적복사는 간단히 설명하자면 **객체의 몸통만 바꿔주는것**을 말한다. 

몸통만 새거고 객체안 주소들은 원본 객체를 참조하게 되어있다.

원본의 값을 set으로 바꿔준다면 방어적 복사를 이용한 객체에도 영향을 준다. 

즉 접근은 간단한 얕은 복사를 통해 필드값들을 가져와놓고 get으로 꼭 필요한 값들만 깊은 복사

를 이용하여 내보내는것이다.

깊은 복사에 비해 단점이 있지만 상황에 따라 사용해야할때가 존재한다.

객체에 final을 붙여 생성시에만 값을 넣어주고 이후에는 변경할수없는 불변객체라면 

방어적 복사를 해도 원본 객체에 지장이 가지않는 새로운 객체를 만들수있다.

깊은 복사는 컬렉션을 새로 만들어서 모든 필드값들을 set으로 담아줘야하는 단점이 있다.

길어지고 장황한 코드가 될 수 있지만 방어적 복사는 비교적 짧은 코드로 구현이 가능하다.

**불변으로 설계할수 없다면 깊은 복사, 불변이라면 방어적 복사.**
