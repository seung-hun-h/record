## 프로그램의 동작을 스레드 스케줄러에 기대지 말라
---
- 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다
	- 구체적인 스케줄링 정책은 운영체제마다 다르다
- 견고하고 빠릿하고 이식성이 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다
	- 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만들자
- 실행 가능한 스레드 수를 가능한 적게 유지하는 방법은 각 스레드가 유용한 작업을 완료한 후에는 다음 일거리가 생길때까지 기다리게 하는 것이다
	- 스레드가 당장 처리할 일이 없다면 실행해서는 안된다
	- 스레드는 바쁜 대기 상태가 되면 안된다
- 특정 스레드가 CPU 시간을 충분히 얻지 못한다고 스레드 우선순위를 조작하면 안된다
	- Thread.yield를 사용하거나 하지마라