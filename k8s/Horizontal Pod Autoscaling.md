- 워크로드의 크기를 수요에 맞게 자동으로 스케일링
- HorizontalPodAutoscaler는 쿠버네티스 API 자원 및 [컨트롤러](https://kubernetes.io/ko/docs/concepts/architecture/controller/) 형태로 구현되어 있다.
	- HorizontalPodAutoscaler API 자원은 컨트롤러의 행동을 결정한다
	- HPA 컨트롤러는 평균 CPU 사용률, 평균 메모리 사용률, 또는 다른 커스텀 메트릭 등의 관측된 메트릭을 목표에 맞추기 위해 목표물(예: 디플로이먼트)의 적정 크기를 주기적으로 조정한다

- HorizontalPodAutoscaler 컨트롤러는 원하는(desired) 메트릭 값과 현재(current) 메트릭 값 사이의 비율로 작동한다
	- CPU 사용률 임계값을 20%로 설정하고 80%까지 올라가면 4개의 레플리카를 생성
