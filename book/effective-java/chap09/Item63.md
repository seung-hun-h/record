## 문자열 연결은 느리니 주의하라
---
- 문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단이다
- 하지만 문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례한다
	- 문자열은 불변이라 양쪽의 내용을 모두 복사해야하기 때문이다
- 성능을 포기하고 싶지 않다면 String 대신 StringBuilder를 사용하자

```Java
String sentence = "hello" + name + ", haha";
```
- 최근 JVM은 위 코드를 StringBuilder로 최적화 해준다. 따라서 한 줄로 문자열 연결 연산자를 사용하면 괜찮다. JDK 버전에 따라 다르겠지만

```Java
String sentence = "";
for(int i = 0; i < 100; i++) {
	sentence += i;
}
```
- 이러한 코드에서는 StringBuilder를 사용하는 것이 좋다
