# 상황에 따라 float 와 double은 피하라가 자연스러운 제목같다.

## **float과 double의 문제**

float과 double의 연산은 정확하지 않다.

```java
    void 부동소수점_연산은_정확하지_않음 (){
        double result = 1.03 - 0.42;
        System.out.println(result); // 0.6100000000000001
        assertThat(result).isNotEqualTo(0.61);
    }
```

`1.03 - 0.42`와 같은 단순한 계산에서도 오차가 발생  
그 이유는 float과 double은 **부동소수점 방식**을 사용하고 있기 때문  

## **고정소수점과 부동소수점**

컴퓨터는 숫자를 2진수로 표현한다.  
이 때, 정수를 2진수로 표현하는 것은 문제가 없다.    
하지만 소수점을 포함한 실수를 표현할 때는 상황이 조금 달라진다.   

**소수점의 위치를 표현하고, 무엇이 정수 부분이고 무엇이 실수 부분인지 구분**해야 하기 때문이다.  
컴퓨터는 이를 위해 `고정소수점` 방식과 `부동소수점` 방식을 사용한다.

### **(1) 고정소수점 방식**

고정소수점 방식은 `소수점의 위치를 고정`하는 것이다.  
만약 실수를 표현하는 5비트의 **고정소수점 방식 자료형**이 있다고 가정해보자.  
그리고 이 자료형은 2비트를 정수부로 사용하고 나머지 3비트를 실수부로 사용한다고 가정한다.  

![https://github.com/Meet-Coder-Study/book-effective-java/raw/main/9%EC%9E%A5/images/60-1.png](https://github.com/Meet-Coder-Study/book-effective-java/raw/main/9%EC%9E%A5/images/60-1.png)

이 방법은 소수점의 위치가 고정되어 있기 때문에 정수를 표현하는 것과 똑같이 실수를 표현할 수 있다.  
따라서 **오차 없는 정확한 연산이 가능**하다는 장점이 있고,  
반면에 **표현 범위가 제한적**이라는 단점이 있다.  

`12.345는 이 자료형으로 표현할 수 있지만
123.45 나 1234.5 는 표현할 수 없다.`

### **(2) 부동소수점 방식**

![https://github.com/Meet-Coder-Study/book-effective-java/raw/main/9%EC%9E%A5/images/60-2.png](https://github.com/Meet-Coder-Study/book-effective-java/raw/main/9%EC%9E%A5/images/60-2.png)

반면에 고정 소수점 방식은 수를 정수/실수로 구분하지 않고  
지수부와 가수부로 구분한다.   
이와 같은 방법은 소수점이 고정되지 않으므로 더 **폭넓은 범위를 표현**할 수 있지만,  
이 계산방식은 필연적으로 **오차가 발생**하여 근사값을 구한다고 한다.    
자바의 float과 double은 이 부동소수점 방식으로 되어있다. (대부분의 언어에서 실수를 부동소수점 방식으로 표현한다.  

## **해결법은?**

### **(1) BigDecimal**

![Untitled](https://user-images.githubusercontent.com/72185011/178525511-f84f0d1f-cc26-4e92-91ef-ea7d36287517.png)


불변 클래스이며 정수. 정수를 저장하는데 `BigInteger`를 사용   
double이나 float으로 선어시 표현할 수 있는 값의 한계가 있기에 valueOf를 통해 바꿔주거나 string으로 표현  

![Untitled 1](https://user-images.githubusercontent.com/72185011/178525532-bbd7de6d-aee9-40e4-a429-e4f93c4bc506.png)

인텔리제이에서는 String으로 변환하라고 경고해준다.  

```java
    void BigDecimal을_활용한_연산() {
        BigDecimal b1 = new BigDecimal("1.03");
        BigDecimal b2 = new BigDecimal("0.42");
        String result = b1.subtract(b2).toString();
        System.out.println(result);	//0.61
    }
```

double형 대신 BigDecimal 객체를 사용하면 실수의 연산을 오차 없이 수행할 수 있다.  
하지만 BigDecimal은 사용하기 번거롭고, 느리다.  

### **(2) 정수 타입을 이용**

실수 타입을 정수 타입으로 사용하는 방법이다.

예를 들어 `0.1 달러 = 10센트`로 표현할 수 있다.

이 방법은 정수 기본형을 사용하기 때문에 연산도 정확하고, 사용하기도 편리하고, 빠르다.

다만, 모든 상황에서 이 방법을 사용할 수 있는 것은 아니다.

```java
1 Satoshi = 0.00000001 BTC
```

![Untitled 2](https://user-images.githubusercontent.com/72185011/178525553-de0b5386-1c22-4f07-8528-913067baff71.png)


오늘자 도지코인의 BTC 가격이다.   
현재가 0.00000652의 의미는 도지코인 1개로   
0.00000652개 만큼의 비트코인을 살 수 있다는 의미이다.   
현재가를 float이나 double로 구현했다면 무슨일이 일어날까?  

```java
public class AppRunner implements ApplicationRunner {
    float dogeCoinPrice = 0.00000652f;
    double quantity = 1000000d;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("당신의 현재 보유액: "+dogeCoinPrice * quantity);
    }
}
```

![Untitled 3](https://user-images.githubusercontent.com/72185011/178525570-23af3331-3a7e-4453-82c7-2911083ee091.png)


정확하지 않은 금액이 계산되어 나온다.   
이와 같이 금액처럼 정확한 소수점을 계산이 필요할 때 float이나 double형을 쓰게 된다면 부정확한 결과가 나오고 정산시에 아주 큰 문제  
그렇기 때문에 금액처럼 정확한 소수점을 계산할 때는 반드시 Big Decimal을 사용해야 한다.  
정확한 계산을 제공하는 대신에 성능 손해는 감수해야한다.   

float 100만번 더하기 vs BigDecimal 100만번 더하기  

```java
float floatVal = 1.1f;
    BigDecimal bigDecimalVal = new BigDecimal("1.1");

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Long begin1 = System.currentTimeMillis();
        for(int i=0; i<=1000000; i++){
            floatVal = floatVal + floatVal;
        }
        System.out.println("float 100만번 더하기 계산시간: " + (System.currentTimeMillis() - begin1));

        Long begin2 = System.currentTimeMillis();
        for(int i=0; i<=1000000; i++){
            bigDecimalVal = bigDecimalVal.add(bigDecimalVal);
        }
        System.out.println("BigDecimal 100만번 더하기 계산시간: " + (System.currentTimeMillis() - begin2));
    }
```

결과

![Untitled 4](https://user-images.githubusercontent.com/72185011/178525588-a2a122e1-3ab9-4b31-9b8a-a7ff7ad250dd.png)


불변객체인만큼 반환하는 객체가 BigDecimal인경우 새로운 인스턴스를 반환하게된다. 따라서 valueOf를 사용한다면 캐싱된 static을 최대한 활용하여 비용을 아낄 수 있을거같다.

이번 아이템의 제목은 정확한 계산을 위해서는 피하라라고 되어있는데 상황에 맞게 float과 double을 사용하라가 옳아보인다 왜냐하면 float은 7자리, double은 15자리까지 표현이 가능하다.

해당 범위 내에서 계산이 가능하다면 BigDecimal의 메모리를 할당하는것보다는 기본 자료형으로 사용하는것이 최선의 상황이라고 생각하기 떄문이다.
