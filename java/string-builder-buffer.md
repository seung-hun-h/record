# String, StringBuilder, StringBuffer

문자열을 저장하는 String은 내부 문자열을 수정할 수 없다. replace 메소드는 메소드 내부 문자를 대치하는 것이 아니라 새로운 String 객체를 생성하는 것이다. 문자열을 결합하는 + 연산자를 많이 사용할 수록 String의 객체 수가 늘어나기 때문에 프로그램의 성능이 느려질 수 있다.

따라서 문자열의 변경이 많을 경우에는 StringBuilder나 StringBuffer를 사용하는 것이 좋다. 이 두 클래스는 내부 버퍼에 문자열을 저장해두고 그 안에서 수정, 삽입, 삭제할 수 있도록 설계되어 있기 떄문이다. 

SpringBuffer와 StringBuilder에도 차이가 있는데 StringBuffer는 동기화가 적용되어 있어 멀티 스레드 환경에서 사용할 수 있고, StringBuilder는 동기화를 제공하지 않는다. 따라서 싱글 스레드에서는 StringBuilder를 사용하는 것이 성능에 좋다.

