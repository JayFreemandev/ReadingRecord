# Item64, 객체는 인터페이스로 참조하라

# 인터페이스로 확장해야 유연해진다

## Item51, 메서드 시그니처 설계편

에는 매개변수의 타입으로는 클래스보다 인터페이스가 낫다라는 내용이 있었다.   
즉 객체는 클래스가 아닌 인터페이스로 확장해야 유연해진다는 의미이다.  

예를 들어 아래와 같다  
List<String> stringList = new ArrayList<>();  O  
ArrayList<String> stringList = new ArrayList<>(); X  

## 물론 적합한 인터페이스가 없다면 클래스 참조

String이나 BIgInteger같은 클래스를 여러가지로 확장시킬 목적으로 설계하는일은 거의 없다. 
final인 경우가 많고 상응하는 인터페이스가 별도로 존재하는 경우도 드물다.
클래스 기반 프레임워크, InputStream이나 OutputSteam같은 객체들도 기반 클래스를 이용한다. 

### 결론
특별한 이유 없으면 인터페이스를 참조하게 만들어서 확장성에 대비하고 유연하게 설계하자. 
이 외 특별한 내용은 없었다.
