## 일반적으로 통용되는 명명 규칙을 따르라
---
## 철자 규칙
### 패키지
- 패키지와 모듈 이름은 각 요소를 점(.)으로 구분하여 계층적으로 짓는다
	-  요소들은 모두 소문자 알파벳 혹은 숫자로 이뤄진다
	- 조직 바깥에서 사용될 패키지는 조직의 인터넷 도메인 이름을 역순으로 사용한다
	- 표준 라이브러리는 java, javax로 시작한다
- 패키지 이름의 나머지는 해당 패키지를 설명하는 하나 이상의 요소로 이뤄진다
	- 각 요소는 일반적으로 8자 이하의 짧은 단어로 한다
	- 여러 단어로 구성되었다면 각 단어의 첫 글자만 사용해도 좋다
- 많은 기능을 제공하는 경우에는 계층을 나눠 더 많은 요소로 구성해도 좋다

### 클래스와 인터페이스
- 하나 이상의 단어로 이뤄지며, 각 단어는 대문자로 시작한다
- 단어의 첫 글자만 딴 약자나 max, min 같이 널리 통용되는 줄임말을 제외하고는 단어를 줄여쓰지 않도록 한다
- 약자의 경우 모두 대문자로 해도 되지만, 첫 글자만 대문자로 하는 쪽이 훨씬 많다

### 메서드와 필드
- 첫 글자를 소문자로 작성한다는 것외에는 클래스 명명법과 동일하다
- 상수 필드는 모두 대문자로 쓰고 밑줄(\_)로 구분한다

### 지역 변수
- 다른 멤버와 유사한 규칙이 적용된다
- 약어를 써도 좋다
	- 다만 문맥에서 이해할 수 있어야 한다

### 타입 매개 변수
- 보통 한 문자로  표현한다
- T: 임의의 타입에 사용
- E: 컬렉션 원소의 타입
- K, V: 맵의 키와 값
- X: 예외
- R: 메서드의 반환 타입
- T, U, V/T1, T2, T3: 임의 타입 시퀀스

## 문법 규칙
### 클래스와 인터페이스
- 객체를 생성할 수 있는 클래스는 단수 명사나 명사 구를 사용한다
- 객체를 생성할 수 없는 클래스는 복수 명사로 짓는다
- 인터페이스는 클래스와 동일하거나, able/ible로 끝나는 형용사로 짓는다
- 애너테이션은 규칙이 없다

### 메서드
- 동작을 수행하는 메서드는 동사 혹은 동사구를 사용한다
- boolean 값을 반환하면 `is`, `has`를 붙인다
- 인스턴스의 속성을 반환하는 메서드는 명사, 명사구, `get`을 사용한다

### 특별한 메서드
- 객체 타입을 바꿔 다른 타입으로 전환하는 메서드느 `toType`
- 객체의 내용을 다른 뷰로 보여주는 메서드는 `asType`
- 객체의 값을 기본 타입으로 보여주는 경우 `typeValue`
- 정적 팩터리 from, of, valueOf, instance, getInstance, newInstance, getType, newType

### 지역 변수와 필드
- 필드 이름에 관한 문법 규칙은 클래스, 인터페이스, 메서드 이름에 비해 덜 명확하고 덜 중요하다
	- API 설계를 잘헀다면 노출될 일이 없다
- boolean 타입의 필드는 boolean 접근자 메서드에서 앞 단어를 뺀 형태이다
- 다른 타입의 필드라면 명사나 명사구를 사용한다
- 지역변수 이름도 필드와 비슷하게 지으면 된다
