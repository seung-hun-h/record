# 자바 모듈 시스템

## 압력: 소프트웨어 유추
### 관심사 분리
- 관심사분리(SoC: Seperation of concerns)는 컴퓨터 프로그램을 고유의 기능으로 나누는 동작을 권장하는 원칙이다
- SoC를 적용하면 서로 거의 겹치지 않는 코드 그룹으로 분리할 수 있다
  - 클래스를 그룹화한 모듈을 이용해 애플리케이션 의 클래스 간의 관계를 시각적으로 보여줄 수 있다
- 자바 9 모듈은 클래스가 어떤 다른 클래스를 볼 수 있는지를 컴파일 시간에 정교하게 제어할 수 있다.
  - 자바 패키지는 모듈성을 지원하지 않는다
- SoC 원칙의 장점
  - 개별 기능을 따로 작업할 수 있으므로 팀이 쉽게 협업할 수 있다
  - 개별 부분을 재사용하기 쉽다
  - 전체 시스템을 쉽게 유지보수할 수 있다

### 정보 은닉
- 정보 은닉은 세부 구현을 숨기도록 장려하는 원칙이다
  - 세부 구현을 숨김으로써 한 부분의 변화가 다른 부분까지 영향을 미치지 않는다
- 자바 9 이전까지는 클래스와 패키지가 의도된 대로 공개되었는지를 컴파일러로 확인할 수 있는 기능이 없었다

### 자바 소프트웨어
- 잘 설계된 소프트웨어를 만드려면 이 두 가지 원칙을 따르는 것이 필수다
  - 자바는 객체지향 언어로 클래스와 인터페이스를 제공한다
  - 특정 문제와 관련된 패키지, 클래스, 인터페이스 그룹으로 만들어 코드를 그룹화할 수 있다
  - 코드 자체를 보고 소프트웨어의 동작을 추론하는 것은 현실적으로 어렵기 때문에 UML 다이어그램 같은 도구를 이용하면 그룹 코드 간의 의존성을 시각적으로 보여줄 수 있다
  - 자바에서는 접근 제어자와 패키지 수준 접근 권한 등을 이용해 메서드, 필드 클래스의 접근을 제어한다
    - 이러한 방식은 원하는 접근 제한을 달성하기 어렵다
    - 최종 사용자에게 원치 않는 메서드를 제공해야할 수도 있다
    - 클래스에 public 필드가 있다면 사용자 입장에서는 당연히 사용할 수 있다고 생각한다

## 자바 모듈 시스템을 설계한 이유
### 모듈화의 한계
- 자바는 클래스, 패키지, JAR 세 가지 수준의 코드 그룹화를 제공한다
- 패키지와 JAR 수준에서는 캡슐화를 거의 지원하지 않는다

**제한된 가시성 제어**
- 많은 애플리케이션은 다양한 클래스 그룹을 정의한 여러 패키지가 있는데 패키지의 가시성 제어 기능은 거의 유명무실한 수준이다
- 내부적으로 사용할 목적으로 만든 구현을 다른 프로그래머가 임시적으로 사용해서 정착해버릴 수 있으므로 결국 기존의 애플리케이션을 망가뜨리지 않고 라이브러리 코드를 바꾸기가 어려워진다
- 보안 측면에서 볼 때 코드가 노출되었으므로 코드를 임의로 조작하는 위협에 더 많이 노출될 수가 있다

**클래스 경로**
- 애플리케이션을 번들하고 실행하는 기능과 관련해 자바는 태생적으로 약점을 가지고있다
  - 클래스를 모두 컴파일한 다음 보통 한 개의 평범한 JAR 파일에 넣고 클래스 경로에 이 JAR 파일을 추가해 사용할 수 있다
  - 그러면 JVM이 동적으로 클래스 경로에 정의된 클래스를 필요할 때 읽는다
- 클래스 경로와 JAR 조합의 약점
  - 클래스 경로에는 같은 클래스를 구분하는 버전개념이 없다
    - 파싱 라이브러리의 JSONParser 클래스를 지정할 때 버전 1.0을 사용하는지 버전 2.0을 사용하는지 지정할 수 없어 무슨 일이 발생할 지 예측할 수 없다
  - 클래스 경로는 명시적인 의존성을 지원하지 않는다
    - 각각의 JAR 안에 있는 모든 클래스는 classes라는 한 주머니로 합쳐진다
    - 한 JAR가 다른 JAR에 포함된 클래스 집합을 사용하라고 명시적으로 의존성을 정의하는 기능을 제공하지 않는다
    - 클래스 경로때문에 어떤일이 발생하는지 파악하기 어렵다
  - 메이븐이나 그래들이 위 같은 문제점을 해결하는데 도움을 준다

### 거대한 JDK
- JDK는 자바 프로그램을 만들고 실행하는 데 도움을 주는 도구의 집합이다
- JDK에는 많은 기술이 추가되었다가 사라졌다
  - 스프링, 네티, 모키토 같은 많은 라이브러리에서 JDK 내부에서만 사용하도록 만든 클래스를 사용했다
  - 이는 자바 언어의 낮은 캡슐화 지원 떄문이다
  - 결과적으로 호환성을 깨지 않고는 관련 API를 바꾸기 아주 어려운 상황이 되었다
- 이러한 문제들로 인해 JDK 자체도 모듈화할 수 있는 자바 모듈 시스템 설계의 필요성이 제기되었다
  - JDK에서 필요한 부분만 골라 사용하고, 클래스 경로를 쉽게 유추할 수 있으며, 플랫폼을 진화시킬 수 있는 강력한 캡슐화를 제공할 새로운 건축 구조가 필요했다

## 자바 모듈: 큰 그림
- 자바 8은 모듈이라는 새로운 자바 프로그램 구조 단위를 제공한다
  - module이라는 새 키워드에 이름과 바디를 추가해서 정의한다
  - 모듈 디스크립터는 module-info.java라는 특별한 파일에 저장된다
  - 모듈 디스크립터는 보통 패키지와 같은 폴터에 위치하며 한 개 이상의 패키지를 서술하고 캡슐화할 수 있지만 단순한 상황에서는 이들 패키지 중 한 개만 외부로 노출시킨다.

![image](https://user-images.githubusercontent.com/60502370/177288545-39937f16-0cda-4aa6-af09-462db7c185bb.png)

- 메이븐 같은 도구를 사용할 때 모듈의 많은 세부 사항을 IDE가 처리하며 사용자에게는 잘 드러나지 않는다

## 모듈 정의와 구문들
### module
- module 지시어르 이용해 모듈을 정의할 수 있다.

```java
module com.iteratrlearning.application {

}
```
### requires
- requires 구문은 컴파일 타임과 런타임에 한 모듈이 다른 모듈에 의존함을 정의한다

```java
module com.iteratrlearning.application {
  requires com.iteratlearning.ui;
}
```
- com.iteratlearning.application은 com.iteratlearning.ui에 의존한다

### exports
- exports 구문은 지정한 패키지를 다른 모듈에서 이용할 수 있도록 공개 형식으로 만든다
  - 아무 패키지도 공개하지 않는 것이 기본 설정이다
- exports는 패키지명을 인수로 받고, requires는 모듈명을 인수로 받는다

```java
module com.iteratrlearning.ui {
  requires com.iteratlearning.core;
  exports com.iteratrlearning.ui.panels;
  exports com.iteratrlearning.ui.widgets;
}
```

### requires transitive
- requires-transitive를 사용하면 다른 모듈이 제공하는 공개 형식을 한 모듈에서 사용할 수 있다고 지정할 수 있다

```java
module com.iteratrlearning.ui {
  requires transitive com.iteratlearning.core;
  exports com.iteratrlearning.ui.panels;
  exports com.iteratrlearning.ui.widgets;
}
```

```java
module com.iteratrlearning.application {
  requires com.iteratlearning.ui;
}
```
- com.iteratrlearning.application 모듈은 com.iteratlearning.core에서 노출한 공개 형식에 접근할 수 있다


### exports to
- exports to 구문을 이용해 사용자에게 공개할 기능을 제한함으로 가시성을 좀 더 종교하게 제어할 수 있다

```java
module com.iteratrlearning.ui {
  requires transitive com.iteratlearning.core;
  exports com.iteratrlearning.ui.panels;
  exports com.iteratrlearning.ui.widgets 새
    com.iteratrlearning.ui.widgetuser;
}
```

- com.iteratrlearning.ui.widgets의 접근 권한을 가진 사용자의 권한을 com.iteratrlearning.ui.widgetuser로 제한할 수 있다

### open과 opens
- 모듈 선언에 open 한정자를 이용하면 모든 패키지를 다른 모듈에 반사적으로 접근을 허용할 수 있다(?)
  - 반사적인 접근 권한을 주는 것 이외에 open 한정자는 모듈의 가시성에 다른 영향을 미치지 않는다

```java
open module com.iteratrlearning.ui {
  
}
```
- 자바 9 이전에는 리플렉션으로 객체의 비공개 상태를 확인할 수 있었다
  - 진정한 의미의 캡슐화는 존재하지 않았다
- 자바 9에서는 기본적으로 리플렉션이 이런 기능을 허용하지 않는다(?)
  - 다른 모듈에서는 불가능
  - 같은 모듈은 가능
  - Jackson ObjectMapper는 기본생성자가 필요하지만 private으로 해도 잘 동작하는데 왜일까?

### uses와 provides
- provides 구문으로 서비스 제공자를 uses 구문으로 서비스 소비자를 지정할 수 있다