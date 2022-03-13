# Item9, try-finally 보다 try-with-resource를 사용해라

# **try-finally**

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다

```
InputStream in =new FileInputStream(src);
try {
            OutputStream out =new FileOutputStream(dst);
try {
byte[] buf =newbyte[BUFFER_SIZE];
int n;
while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            }finally {
                out.close();
            }
        }finally {
            in.close();
        }
```

자원이 둘 이상 쓰이면 위 코드처럼 자원을 계속 감싸다보니 코드가 지저분해진다

또한 예외처리 문제도 있다

**예외는 try, finally 블록 모두 발생할 수 있는데, 한 예외가 다른 예외를 집어삼킬 수 있다**

# **try-with-resources**

이러한 문제들은 자바 7의 try-with-resources로 해결할 수 있다

이 구조를 사용하려면 해당 자원이 **AutoCloseable** 인터페이스를 구현해야 한다

만약 닫아야 하는 자원을 뜻하는 클래스를 작성한다면, 꼭 ***AutoCloseable***을 구현하자

```
try (InputStream in =new FileInputStream(src);
		OutputStream out =new FileOutputStream(dst)) {
byte[] buf =newbyte[BUFFER_SIZE];
int n;
while ((n = in.read(buf)) >= 0)
    out.write(buf, 0, n);
}
```

위의 try-finally 코드를 try-with-resources로 변경하였다

가독성이 좋아져서 문제점을 빠르게 파악할수있다.

catch 절도 쓸 수 있어서 try 문을 중첩하지 않고 다수의 예외를 처리할 수 있다

```
try (BufferedReader br =new BufferedReader(new FileReader(path))) {
return br.readLine();
    }catch (IOException e) {
return defaultVal;
  }
```

# 결론

자원 해제가 필요한 라이브러리를 try할때 (with resource)에 담아서 사용하자. 단 Autocloseable 인터페이스를 구현받은 자원만 가능하다.