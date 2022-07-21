# Item 87, 커스텀 직렬화 형태를 고려하라.

# 정리

- 직렬화는 어떤 데이터를 다른 데이터의 형태로 변환하는 것이다. 신뢰할 수 없는 역직렬화하는 것 자체가 스스로를 공격에 노출하는 행위다역직렬화를 해야한다면 ObjectInputFilter를 사용하자
- 외부 저장소로 저장되는 데이터는 짧은 만료시간의 데이터를 제외하고 자바 직렬화를 사용을 지양합니다.
- 역직렬화시 반드시 예외가 생긴다는 것을 생각하고 개발합니다.
- 자주 변경되는 비즈니스적인 데이터를 자바 직렬화을 사용하지 않습니다.
- 긴 만료 시간을 가지는 데이터는 JSON 등 다른 포맷을 사용하여 저장합니다.
<br>



## **객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.**

"이상적인 직렬화 상태"  
물리적 표현과 논리적 내용이 같은 상태  
물리적 표현 → 코드로 어떻게 구현했는지  
논리적 내용 → 실제로 어떤 것을 의미하는지  

```java
public class Name implements Serializable {

    private final Stirng lastName;

    private final String firstName;

    private final String middleName;
}
```
<br>

## **객체의 물리적 표현과 논리적 내용이 같은 다른 경우 문제점**

```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;

    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    // ... 생략
}
```

1. 공개API가 현재의 내부 표현 방식에 영구히 묶인다.  
    - 예를 들어, 향후 버전에서는 연결 리스트를 사용하지 않게 바꾸더라도 관련 처리는 필요해진다. 따라서 코드를 절대 제거할 수 없다.  
2. 너무 많은 공간을 차지할 수 있다.  
    - 위의 StringList 클래스를 예로 들면, 기본 직렬화를 사용할 때 각 노드의 연결 정보까지 모두 포함될 것  
    - 하지만 이런 정보는 내부 구현에 해당하고, 직렬화 형태에 가치가 없다. 네트워크로 전송하는 속도만 느려진다.  
3. 시간이 너무 많이 걸릴 수 있다.  
    - 직렬화 로직은 객체 그래프의 위상에 관한 정보를 알 수 없으니, 직접 순회할 수밖에 없다.  
4. 스택 오버플로를 일으킬 수 있다.  
    - 기본 직렬화 형태는 객체 그래프를 재귀 순회한다. 호출 정도가 많아지면 이를 위한 스택이 감당하지 못할 것이다.  
<br>

## **합리적인 직렬화 형태**

```java
public final class StringList implements Serializable {
    private transient int size = 0;// 직렬화 대상에서 제외한다.
    private transient Entry head = null;

    // 이번에는 직렬화 하지 않는다.
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }

    // 문자열을 리스트에 추가한다.
    public final void add(String s) { ... }

    /**
     * StringList 인스턴스를 직렬화한다.
     */
    private void writeObject(ObjectOutputStream stream)
            throws IOException {
        stream.defaultWriteObject();
        stream.writeInt(size);

        // 모든 원소를 순서대로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }

    private void readObject(ObjectInputStream stream)
            throws IOException, ClassNotFoundException {
        stream.defaultReadObject();
        int numElements = stream.readInt();

        for (int i = 0; i < numElements; i++) {
            add((String) stream.readObject());
        }
    }
    // ... 생략
}
```
<br>

**transient**

클래스에서 transient 또는 static 키워드가 선언된 필드를 제외하고는 모두 직렬화 대상이 된다.  
transient 키워드가 선언된 멤버 변수는 직렬화 대상에 제외되었기 때문에 자바 객체로 변환되는 역직렬화 결과에서도 값을 확인할 수 없다.  
**writeObject 와 readObject 가 private 으로 기술되어 있다는 사실에 주목**해볼만 하다.  

다른 접근 지정자로 선언된 경우 호출되지 않는다. **private** 으로 선언되었다는 것은  
이 클래스를 상속한 서브 클래스에서 메서드를 **재정의(override)**를 하지 못하게 한다는 것이다.  

또한 다른 객체는 호출할 수 없기 때문에 클래스의 무결성이 유지되며 수퍼 클래스와 서브 클래스는 독립적으로 직렬화 방식을 유지하며 확장될 수 있다.   
직렬화 과정에서는 **리플렉션(reflection)**을 통해 메서드를 호출하기 때문에 접근 지정자는 문제가 되지 않는다.  

defaultWriteObject() 와 defaultReadObject() 는 각각 기본 serialization 을 수행한다.  
따라서 custom serialization 의 전후에 반드시 호출해줘야 한다.  

```java
public class SomeClass implements Serializable {
    private String fld1;
    private int fld2;
    private transient String fld3; 
    private void readObject(java.io.ObjectInputStream stream)
         throws IOException, ClassNotFoundException {
         stream.defaultReadObject(); //fills fld1 and fld2;
         fld3 = Configuration.getFooConfigValue();
    }
]
```

이렇게 해야 향후 릴리즈에서 transient가 아닌 필드가 추가되더라도 상위와 하위 모두 호환이 가능하기 때문이다.
<br>

## **SerialVersionUID**

```java
// 무작위로 고른 long 값
private static final long serialVersionUID = 0204L;
```

직렬화를 할 때 SUID 선언이 없다면 내부에서 자동으로 유니크한 번호를 생성하여 관리하게 된다.  
SUID는 직렬화와 역직렬화 과정에서 값이 서로 맞는지 확인한 후에 처리를 하기 때문에 이 값이 맞지 않다면  InvalidClassException 예외가 발생한다.  
자바의 직렬화 스펙 정의를 살펴보면 SUID 값은 필수가 아니며 선언되어 있지 않으면 클래스의 기본 해시값을 사용한다.  

따라서 **직접 SUID를 명시하지 않더라도 내부에서 자동으로 값이 추가**되며   
이 값들은 클래스의 이름, 생성자 등과 같이 클래스의 구조를 이용해서 생성한다  

직렬화 가능한 클래스(Article)를 선언할 때 SUID 값을 생략해도 내부적으로 정보가 생성되어 있음을 유추할 수 있다  
<br>

### **명시적으로 선언하는 것을 권장한다**
 
- 런타임에 많은 시간이 소요된다.  
- 꼭 유니크할 필요는 없지만 이전 버전 클래스와의 호환을 위해 값을 변경하지 않는 것이 좋다.  

개발을 하다 보면, 종종 다음 릴리즈에서 제대로 구현하기로 하고 이번 릴리즈에서는 동작만 하도록 하는 경우 발생    
근데 이걸 Serializable을 구현하고 기본 직렬화 형태를 사용하면 다음 릴리즈 때 발이 묶임    
먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라    

---

## **참고**

<순환참조가 걸려있는 객체의 경우 직렬화를 하면>
[Does Java Serialization work for cyclic references?](https://stackoverflow.com/questions/1792501/does-java-serialization-work-for-cyclic-references)
java의 기본 직렬화 구조는 순환 참조가 있는 클래스에서 **이미 직렬화 되어있는 클래스라면 직렬화 하지않는다**  

```java
import java.io.*;
class A implements Serializable { B b; }
class B implements Serializable { C c; }
class C implements Serializable { A a; }
public class Test {
    public static void main( String [] args ) throws IOException, ClassNotFoundException {
        A a = new A();
        a.b = new B();
        a.b.c = new C();
        a.b.c.a = a;

        ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
        ObjectOutputStream outputStream = new ObjectOutputStream(byteStream);
        outputStream.writeObject(a);

        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteStream.toByteArray());
        ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);

        A deserialzedA = (A) objectInputStream.readObject();

    }
}
```
