# Item37. ordinal 인덱싱 대신 EnumMap을 사용

## 

식물의 생명주기를 나타내는 클래스

```java
class Plant {
    enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override
    public String toString() {
        return name;
    }
}
```

요구 사항이 식물들의 생명주기를 묶어 배열 하나로 관리하는것이다.
![Untitled 1](https://user-images.githubusercontent.com/72185011/176198964-90000771-23f4-4325-b684-dface232c39c.png)


성능적인 이유가 아니라면 굳이 배열을 사용하는 이유를 모르겠다. Ordianal 배열의 인덱스를 사용하는 예시코드이다. 코드에는 물과 기름인 배열과 제네릭도 보이고 가독성도 좋지 않다.  정리하는 김에 배열과 제네릭에 대해 다시 짚고 넘어가자.

배열은 ‘공변’ 이고 제네릭은 ‘불공변’이다. 여기서 공변이라는 의미는 예를 들어 배열의 경우 Sub가 Super의 하위 타입일때 Sub[]는 Super[]의 하위 타입이 된다. 상속관계가 분명하다.

반면에 제네릭의 경우 Sub가 Super의 하위 타입이더라도 List<Sub>은 List<Super>의 하위 타입이 아니다. 이런 경우를 불공변이라고 표현한다. 

```java
Object[] Objects = new String[1];
Objects[0] = 1;
```

String은 Object의 하위 타입이므로 컴파일이 가능하다. Objects는 컴파일 타임에는 Ojbect[] 이기에 Integer 1을 할당이 가능하나 런타임때에는 String[] 이기 때문에 예외가 발생한다.

자바를 사용하는 이유 중 하나가 **정적 컴파일 언어로 시스템의 안정성을 높여**주기 위함인데 배열의 이러한 특징은 컴파일 타임에 안정성을 보장해줄 수 없다.

반면에 제네릭의 경우 Object일지라도 String을 바로 컴파일에서 찾아내어 타입 안정성있는 개발이 가능하다 왜냐하면 배열은 런타임때 실체화가 되고 제네릭은 런타임때 타입이 다 날라가버린다.

이런 상황에서 제네릭 배열을 사용해버리면 타입 안정성을 보장하지못함. 물론 직접적으로는 제네릭 배열을 생성하지못하지만 와일드 카드나 강제 형변환을 통해 제네릭 배열을 생성할 수 는 있다.

## 배열 코드 대신에 EnumMap으로 변경

EnumMap Put의 내부를 들여다보면 훨 씬 더 간결하고 ordinal의 key를 사용하여 같은 성능을 보장하는 코드를 작성할수있다.

  ![Untitled](https://user-images.githubusercontent.com/72185011/176198989-d420f443-e892-4b23-bb4d-ea3bc72ee7cb.png)

Ordinal에 대한 직접적인 사용을 피하고 EnumMap을 사용해라 Steam에 대해 익숙한 사람이라면 EnumMap과 함께 좀 더 간결하게 작성해볼 수 있다. 

나도 그렇고 설명하는분 의견도 그렇고 책의 예제를 통해서 틀린점을 확 와닿기가 쉽지 않았다. Ordinal을 사용해서 Enum을 작성해본적이 없기 때문인거같다. 

아이템 42부터는 람다와 스트림에 대해서 나오기 시작하는데 여기서는 예제를 한땀 한땀 따라치며 다양한 레퍼런스를 접해서 조금씩 변형해보기도하는 연습이 필요해보인다.
