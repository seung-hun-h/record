## null이 아닌, 빈 컬렉션이나 배열을 반환하라
---
- 컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할 때면 항시 방어 코드를 넣어줘야 한다
- `Collections.emptyList`, `Collections.emptySet`, `Collections.emptyMap`을 활용하자
