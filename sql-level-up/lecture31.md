# 모델 갱신의 주의점
---
- 복잡한 쿼리 때문에 머리를 싸매지 않아도 된다는 점에서 모델 갱신은 좋은 해결책이지만, 역시 트레이드오프가 존재한다

## 1. 높아지는 갱신비용
- Orders 테이블의 배송 지연 플래그 필드에 값을 넣는 처리가 필요하므로, 검식 부하를 갱신 부하로 미루는 꼴이다
- Orders 테이블에 레코드를 등록할 때 이미 플래그 값이 정해져 있다면 갱신 비용은 거의 올라가지 않는다
- 하지만 등록할 때 아직 개별 상품의 배송 일정이 정해져 있지 않을 수 있다
- 이런 경우에는 플래그를 UPDATE 해야 하므로 갱신비용이 올라간다

## 2. 갱신까지의 시간 랙(Time Rag) 발생
- 이 방법에는 데이터 실시간성이라는 문제가 발생한다
- 배송 예정일이 주문 등록 후에 갱신되는 경우에는 Orders 테이블의 배송 지연 플래그 필드와 OrderReceipts 테이블의 배송 예정일 필드가 실시간으로 동기화 되지 않으므로 차이가 발생할 수 있다
- 이러한 처리를 야간에 배치 갱신을 통해 일괄 처리한다면 시간 랙 기간은 더 길어질 수 있다


## 3. 모델 갱신 비용 발생
- RDB 데이터 모델 갱신은 코드 기반의 수정에 비해 대대적인 수정이 요구된다
- 갱신 대상 테이블을 사용하는 다른 처리에 문제가 발생할 가능성도 있으므로 개발 프로젝트 막바지 단계에 모델을 변경하는 것은 시스템 품질과 개발 일정 모두에 큰 리스크가 된다
- 실제 운영에 들어가면 더 이상의 모델 변경은 불가능하다