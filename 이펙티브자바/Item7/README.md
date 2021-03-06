# Item7, 다 쓴 객체 참조를 해제하라

## 다 쓴 객체를 수동으로 참조해제할 상황은 다음과 같다.

- 클래스가 메모리를 관리
- 캐시
- 리스너/콜백

## 클래스가 메모리를 관리

자바가 자동으로 메모리를 수거하는 가비지 컬렉션이 있다고 메모리 관리가 끝나는것은 아니다.

스택의 예시로 설명이 되어있다 스택에 값을 넣고 pop으로 빼더라도 GC가 일어나지않는다. 

스택의 구현체가 레퍼런스를 그대로 가지고있기때문에 Null이 아니라 GC의 대상이 아니기때문이다.

스택에서 값을 꺼낸 그 위치를 this.stack[size] = null ; 과 같이 설정해주면 다음 GC에서 수거가된다.

다시 해당 위치의 객체에 다시 접근하려고 할 때 `NullPointerException`이 발생할 수 있지만 그 자리

에 있는 객체를 비우지 않고 실수로 잘못된 객체를 돌려주는 것 보다는 낫다. 발견하기 힘든 버그를 

찾기 위해 코드를 샅샅히 뒤지는 것보다는 차라리 프로그램이 에러를 던져주는 편이 훨씬 낫기 때문

이다.

하지만 객체에 Null을 넣는거는 일반적인 대처방법은 아니다. 그럼 언제 null로 설정해주느냐, 위처럼 

클래스 내에서 메모리 관리를 해주는 경우에 해준다. `Stack` 구현체처럼 배열을 내부에서 직접 관리

하는 경우에 GC는 어떤 객체가 필요없는 객체인지 알 수 없다. 따라서 해당 레퍼런스를 null로 만들어

서 GC에게 필요없는 부분이라는 것을 알려주어야한다.

## 캐시나 리스너/콜백

캐시 역시 메모리 누수를 일으키는 주범이다. 객체 참조를 캐시에 넣어두고, 이 사실을 까맣게 잊은 

채 그 객체를 다 쓴 뒤로도 한참을 그냥 놔두는 경우를 자주 접할 수 있다. **캐시 외부에서 key를 참조**

**하는 동안만 엔트리가 살아있는 캐시가 필요한 상황**이라면 `WeakHashMap` 사용을 고려해볼 수 있다.

HashMap경우 예를 들어 Map<Foo, String> map = new HashMap<>(); 일때 map.put(”key”, 1)을 넣

어주고 다시 key = null을 선언하더라도 GC가 일어나지않는다. 왜냐하면 해시맵은 참조를 끊어도 

map이 비어져있지 않기때문이다. 이럴경우 WeakHashMap을 사용하여 같은 상황으로 key에 null을 

준다면 map이 비워져 GC가 수거하게된다. 

## 결론

메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년 간 잠복하는 경우도 있다. 이런 누수는 철저

한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 

문제는 예방법을 익혀 두는 것이 중요하다.