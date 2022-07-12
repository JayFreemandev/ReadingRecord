# 더 적합한 데이터 타입이 있거나 새로 작성 할 수 있다면,  문자열 사용을 자제하자  

## **문자열은 다른 타입을 대신하기에 적절하지 않다.**

입력받을 데이터가 진짜 문자열인 경우에만 사용하는것이 좋다.  
데이터가 수치형 → 수치에 대한 타입 (int, double, float)  
데이터가 참, 거짓 → boolean or Enum 자료형   
적절한 타입이 있다면 그것을 사용하고 없다면 새로운 타입을 하나 만들어서 사용하자.  
되도록이면, 문자열 사용을 피하자.   

## **문자열은 열거 타입을 나타내기에 적절하지 않다.(이전 enum 예시)**

```java
public static final int APPLE_FUJI          = 0;
public static final int APPLE_PIPPIN        = 1;
public static final int APPLE_GRANNY_SMITH  = 2;

public static final int ORANGE_NAVEL  = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD  = 2;
```

### **정수 열거 패턴의 문제점**

이와 같은 정수 열거 패턴에도 문제점이 많다. → 아이템 34

- 타입의 안전성을 보장할 수 없으며   
- 접두어를 사용하여 이름 충돌을 방지해야 한다(별도의 이름 공간이 없다) 
- 프로그램이 깨지기 쉽다.  

평범한 상수를 나열한 것뿐이라 컴파일하면 그 값이 클라이언트(API가 적용된 클래스: 코드, 프로그램) 파일에 그대로 새겨진다.  
즉, API의 상수 값이 바뀌면 클라이언트도 재컴파일을 해야만 한다.  
- 정수 상수는 문자열로 출력하기 까다롭다  

### **문자열 패턴의 문제점**

상수의 의미를 출력하는 것은 좋지만 이를 하드 코딩 해야한다는 점  
오타가 있어도 컴파일러가 확인할 방법이 없다.   

### **👍 열거 패턴 → 열거 타입**

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```

## **문자열은 혼합 타입을 대신하기에 적합하지 않다.**

```java
String compoundKey = className + "#" + i.next();
```

className, i.next() 에서 "#" 이 쓰이면 혼란스러울 것  
각 요소를 분리해서 사용하려면 파싱해야 하므로 느리고, 귀찮고, 오류가능성도 증가한다.  
적절하게 equals, toString, CompareTo 를 사용할 수 없고, String Class 가 제공하는 메서드만 사용해야 한다.  
차라리 두 요소를 병합해서 관리하는 클래스를 만드는 것이 낫다.  

```java
public class studyInfo {
    private String team;
    private int itemNum;
    private static class compoundKey{
        private String team;
        private int itemNum;

        public compoundKey(String team, int itemNum) {
            this.team = team;
            this.itemNum = itemNum;
        }
        public studyInfo compound(){
            return new studyInfo(this);
        }
    }
    private studyInfo(compoundKey compoundKey){
        team = compoundKey.team;
        itemNum = compoundKey.itemNum;
    }

    public static void main(String[] args) {
        ArrayList<Integer> itemList = new ArrayList<>(List.of(60,61,62));
        Iterator<Integer> i = itemList.iterator();
        
        studyInfo student1 = new studyInfo.compoundKey("B팀",i.next()).compound();
        studyInfo student2 = new studyInfo.compoundKey("A팀",i.next()).compound();
        studyInfo student3 = new studyInfo.compoundKey("B팀",i.next()).compound();

        System.out.println("student1: " + student1.team + " 아이템 " + student1.itemNum);
        System.out.println("student2: " + student1.team + " 아이템 " + student2.itemNum);
        System.out.println("student3: " + student1.team + " 아이템 " + student3.itemNum);
    }
}
/*
student1: B팀 아이템 60
student2: A팀 아이템 61
student3: B팀 아이템 62
*/
```

## **문자열은 권한을 표시하기에 적합하지 않다.**

ThreadLocal이란?
일반 변수의 경우 : 수명이 코드 블록 내에서만 가능하다  

```java
{
	int a = 10;
// 블록 안에서 사용가능
}
//변수 a 는 위 코드 블록이 끝나면 사용할 수 없다/
```

[https://t1.daumcdn.net/tistoryfile/fs14/36_tistory_2009_02_13_15_44_499516b885314?x-content-disposition=inline](https://t1.daumcdn.net/tistoryfile/fs14/36_tistory_2009_02_13_15_44_499516b885314?x-content-disposition=inline)

스레드 지역 변수 설계 기능이란 , 각 스레드가 자신만의 변수를 갖게 해주는 기능을 의미한다.  
자바2 이전에는 이를 개발자가 직접 설계했고, 클라이언트가 제공한 문자열 키로 지역변수를 식별하기까지 했다.  

```java
public class ThreadLocal {
    private ThreadLocal() { } //객체 생성 불가

    // 현 스레드의 값을 키로 구분해 저장
    public static void set(String key, Object value);

    // (키가 가리키는) 현 스레드의 값을 반환한다.
    public static Object get(String key);
}
```

그런데 만약 두 클라이언트가 서로 같은 키를 사용하게 되면, 두 클라이언트 모두 제대로 동작하지 못하게 된다.  
악의적인 클라이언트라면 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 획득할 수도 있다.  
이런 경우에는 String으로 권한을 구분하는 것이 아니라 별도의 타입을 만들어 해결해야 한다.  

```java
public class ThreadLocal {
    private ThreadLocal() {} //객체 생성 불가
    
    public static class Key {
        key() {}
    }
    
    //위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
		return new Key();
    }
    
    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

set / get method는 더이상 static 메소드가 아니어도 된다    
Key 클래스의 인스턴스로 만든 후, 타입의 안전성을 위해 제네릭으로 변경해보면 실제 ThreadLocal과 비슷한 형태를 띈다.   

```java
public final class Key {
    public Key();
    public void set(Object value);
    public Object get();
}
//=============================================================================
public final class Key<T> {
    public Key();
    public void set(T value);
    public T get();
}
```

## 결론

문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입등이 있다.  
문자열을 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류가능성도 크다.  
더 적합한 데이터 타입이 있거나 새로 작성 할 수 있다면, 문자열 사용을 자제하자!  
