## 라이브러리를 익히고 사용하라
---
```Java
private static final Random rnd = new Random();  
public static void main(String[] args) {  
   int n = 2 * (Integer.MAX_VALUE / 3);  
   int low = 0;  
  
   for (int i = 0; i < 1000000; i++) {  
      if (random(n) < n / 2) {  
         low++;  
      }  
   }  
  
   System.out.println("low = " + low);  
}  
  
private static int random(int n) {  
   return Math.abs(rnd.nextInt()) % n;  
}
```
- `random` 메서드는 잘 작성한 것처럼 보이지만 문제가 많다
	- n이 그리 크지 않은 2의 제곱수라면 얼마 지나지 않아 같은 수열이 반복된다
	- n이 2의 제곱수가 아니라면 몇몇 숫자가 평군적으로 더 자주 반환된다
	- n 값이 커지면 이 현상은 더욱 두드러진다
- low의 값이 이상적이라면 50만이 되어야 하지만, 실제로는 66,666에 가까운 수가된다
- `Random.nextInt(int)`를 사용하면 이 문제는 해결된다
- 자바 7부터는 `Random` 대신 `ThreadLocalRandom`을 사용하자
	- 포크-조인 풀이나 병렬 스트림에서는 `SplittableRandom`을 사용하자

### 표준 라이브러리를 사용하자
- 표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 앞서 사용한 다른 프로그래머의 경험을 활용할 수 있다
- 표준 라이브러리를 사용하면 핵심적인 일과 크게 관련 없는 문제를 해결하느라 시간을 허비하지 않아도 된다
- 표준 라이브러리를 사용하면 따로 노력하지 않아도 성능이 개선된다
- 표준 라이브러리를 사용하면 기능이 점점 많아진다
- 표준 라이브러리를 사용하면 우리가 작성한 코드가 다른 사람에게 낯익은 코드가 된다
- 메이저 릴리즈에는 주목할 만한 수많은 기능이 라이브러리에 추가된다
- 자바 프로그래머라면 적어도 `java.lang`, `java.util` , `java.io`와 그 하위 패키지들에는 익숙해져야 한다
	- 추가적으로 `java.util.concurrent`도 알아두면 좋다

### 그 다음은 품질 좋은 서드파티 라이브러리다
- 자바 표준 라이브러리에서 원하는 기능을 찾지 못하면 구글의 구아바같은 품질 좋은 서드파티 라이브러리를 사용하자