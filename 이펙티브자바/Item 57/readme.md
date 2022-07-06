# Item57, 지역변수의 범위를 최소화하라

### **지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성은 낮아진다.**

# 지역변수의 범위를 줄이는방법

## 1. 가장 처음에 쓰일 때 선언한다

지역변수를 사용하려면 멀었는데 **미리 선언 부터 해두면 가독성이 떨어진다**.  
또한 지역변수의 범위는 선언된 지점부터 그 지점을 포함한 블록이 끝날 때까지이므로,   
실제 사용되는 블록 바깥에 선언된 지역변수는 그 블록이 끝나더라도 살아남게 된다.   
실수로 해당 변수를 사용하게 된다면 예기치 못한 상황이 발생할 가능성이 있다.   

**메소드 상단에서 사용할 변수를 모두 선언한경우**  

```java
public void calculate() {
	int a = 1;
	int b = 2;
	int c = 3;

	b = b + c;
	c = c + c;

	...

	a = b + c;

}
```

**변수를 사용하는 위치에서 선언한 경우**

```java
public void calculate() {
	int b = 2;
	int c = 3;

	b = b + c;
	c = c + c;

	...

	int a = b + c;

}
```

## 2. 거의 모든 지역변수는 선언과 동시에 초기화한다

만약 초기화에 필요한 정보가 충분하지 않다면 충분해질때까지 선언을 미뤄두자.  

```java
public void calculate() {
	Person person = new Person();

	...

	person.setName("incheol");
	person.setAge(20);
}
```

```java
public void calculate() {
	Person person = new Person("incheol", 20);
	...
}
```

그런데 이 규칙의 예외로 초기화에 검사 예외를 던질 가능성이 있는 경우는 try블록 안에서 초기화해야한다.   
만약 변수를 try블록이 끝난 뒤에도 사용해야 한다면 선언은 try블록 바로 앞에서 선언하도록 하자.  

**반복문은 독특한 방식으로 변수 범위를 최소화 해준다**.   
반복 변수의 범위는 for문의 반복문의 몸체, for 키워드와 몸체 사이의 괄호 안으로 제한된다.  
따라서 for 문안에서 선언된 반복 변수는 해당 for 문 밖에선 사용할 수 없게된다.  

for문의 경우 for문에서 선언된 반복 변수를 for 문 바깥에서 사용할 경우 컴파일 타임에 잡아주기도 한다.  
![https://blog.kakaocdn.net/dn/ckrfgX/btqFdnnZG2T/c7tSSazW43zS8vhxuepVv0/img.png](https://blog.kakaocdn.net/dn/ckrfgX/btqFdnnZG2T/c7tSSazW43zS8vhxuepVv0/img.png)
사용할 수 없다   

또한 for문의 변수 유효 범위가 for문 범위와 일치하기 때문에 똑같은 이름의 변수를 여러 반복문에서 사용하여도 서로 영향을 주지않는다.  
![https://blog.kakaocdn.net/dn/Ugn7D/btqFa6Vss85/sKthW6RpvdPJoJHJSWqMuK/img.png](https://blog.kakaocdn.net/dn/Ugn7D/btqFa6Vss85/sKthW6RpvdPJoJHJSWqMuK/img.png)
영향을 주지않는다  

또 for문 관용구의 반복 변수를 여러개 선언하여 사용할 수 있다.  
![https://blog.kakaocdn.net/dn/d928fZ/btqFbAIymMi/gIxXyuWJBDmKx6syikxu80/img.png](https://blog.kakaocdn.net/dn/d928fZ/btqFbAIymMi/gIxXyuWJBDmKx6syikxu80/img.png)
반복 변수 선언  

반복 여부를 결정짓는 변수 i 의 한계값을 변수 k 에 저장하여, 반복 때마다 다시 계산하는 비용을 없앴다.   
같은 값을 반환하는 메소드를 매번 호출한다면 이런 관용구를 사용하면 좋을 것이다.   

## 3. 메소드를 작게 유지하고 한 가지 기능에 집중한다.

한 메소드에서 여러 가지 기능을 처리한다면 그중 한기능과만 관련된 지역변수라도 다른 기능을 수행하는 코드에서 접근할 수 있을 것이다.단순히 메소드를 기능별로 쪼갠다면 이런 것을 해결할 수 있을 것이다.

```java
// case1. 하나의 메소드에 여러 기능을 포함한 경우
public void buy() {
	// 아이템의 가격을 확인한다. 
	// 아이템의 할인 정책을 적용한다. 
	// 사용자의 보유 금액을 아이템 가격만큼 차감한다. 
}

// case2. 기능단위로 메소드를 쪼개서 사용한 경우
public void buy() {
	아이템_가격_확인()
	아이템_할인_정책_적용()
	사용자_보유_금액_차감()
}

public Long 아이템_가격_확인() {
	...
}

public void 아이템_할인_정책_적용() {
	...
}

public void 사용자_보유_금액_차감() {
	...
}

//Incheol's 예제
```
