# 카프카 커맨드 라인 툴 소개
---
## 카프카 커맨드 라인 툴
- 카프카 커맨드 라인 툴들은 카프카를 운영할 때 가장 많이 접하는 도구다
- 애플리케이션을 운영할 때는 토픽이나 파티션 개수 변경과 같은 명령을 실행해야 하는 경우가 자주 발생하기 때문에 카프카 커맨드 라인툴과 툴별 옵션을 잘 알아둬야 한다
- 커맨드 라인 툴을 통해 토픽 관련 명령을 실행할 때는 필수 옵션과 선택 옵션이 있다
	- 선택 옵션은 지정하지 않을 시 브로커에 설정된 기본 설정값 또는 커맨드 라인 툴의 기본 설정값으로 대체된다
	- 커맨드 라인 툴을 사용하기 전에 현재 브로커 옵션이 어떻게 되어 있는지 확인한 후에 사용하면 실수할 확률이 줄어든다

## kafka-topics.sh
---
- 토픽 생성(`--create`)
```CLI
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic hello.kafka
```
- 토픽 생성 추가정보(`--partitions`, `--replication-factor`, `--config`)
```CLI
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic hello.kafka --partition 10 -- replication-factor 1 --config retension.ms=1728000000
```

- 토픽 정보 확인(`--describe`)
```CLI
bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic hello.kafka --describe
```
- 토픽 조회(`--list`)
```CLI
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```
- 파티션 개수 증가(`--alter --partition`)
```CLI
bin/kafka-topics.sh --bootstrap-server localhost:9092 --topic hello.kafka --alter --partitions 4
```

	- 이미 생성된 파티션을 다시 줄일 수 없다
	- 생성된 파티션보다 작은 파티션의 수를 입력하면 `InvalidPartitionsException`이 발생한다
	- 만약 파티션을 줄여야 한다면 새로운 토픽을 생성하는 편이 낫다

## kafka-configs.sh
---
- 토픽 일부 옵션 설정(`--alter --add-config`)
```CLI
bin/kafka-configs.sh --bootstrap-server localhost:9092 --topic test --alter --add-config min.insync.replicas=2
```
- 브로커에 설정된 기본값 조회(`--broker`, `--all`, `--describe`)
```CLI
bin/kafka-configs.sh --bootstrap-server localhost:9092 --broker 0 --all --describe
```

## kafka-console-producer.sh
---
- 토픽에 데이터 전송
```CLI
bin/kafka-console.producer --bootstrap-server localhost:9092 --topic hello.kafka
```
- 메시지 키 설정
	- `parse.key=true`, `key.seperator=:` 처럼 별도의 설정이 필요하다
	- `key.seperator`를 설정하지 않으면 기본 설정은 Tab delimeter이다
```CLI
bin/kafka-console.producer --bootstrap-server localhost:9092 --topic hello.kafka --property "parse.key=true" --property "key.separator=:"
```

- 메시지 키와 메세지 값을 함께 전송한 레코드는 토픽의 파티션에 저장된다
- 키가 null인 경우에는 프로듀서가 파티션으로 전송할 때 레코드 배치 단위로 라운드 로빈으로 전송한다
- 베시지 키가 존재하는 경우에는 키의 해시값을 작성하여 존재하는 파티션 중 한개에 할당된다

## kafka-console-consumer.sh
---
- 프로듀서에 전송한 데이터 확인
	- `--from-beginning` 옵션을 주면 처음 데이터부터 출력한다
```CLI
bin/kafka-console-consumer.sh --bootstrap.server localhost:9092 --topic hello.kafka --from-beginning
```
- 메시지키와 메시지 값을 확인
```CLI
bin/kafka-console-consumer.sh --bootstrap.server localhost:9092 --topic hello.kafka --property print.key=true --property key.separator="-" --from-beginning
```
- 최대 메시지 개수 설정(`--max-message`)
```CLI
bin/kafka-console-consumer.sh --bootstrap.server localhost:9092 --topic hello.kafka --from-beginning --max-messages 1
```
- 그룹 설정(`--group`)
	- 해당 옵션을 사용하면 컨슈머 그룹 기반으로 `kafka-console-consumer`가 동작한다
	- 컨슈머 그룹이란 특정 목적을 가진 컨슈머들을 묶음으로 사용하는 것을 뜻한다
	- 컨슈머 그룹으로 토픽의 레코드를 가져갈 경우 어느 레코드까지 읽었는지에 대한 데이터가 카프카 브로커에 저장된다

## kafka-consumer-groups.sh
---
- 컨슈머 그룹은 따로 생성하는 명령을 하지 않고 컨슈머를 동작시킬 때 그룹 이름을 지정하면 새로 생성된다
- 컨슈머 그룹 확인(`--list`)
```CLI
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```
- 컨슈머 그룹 정보 확인(`--describe`)
	- 그룹 이름, 토픽, 파티션 개수, 현재 오프셋, 마지막 오프셋, 컨슈머 랙을 나타낸다
	- 컨슈머 랙은 현재 오프셋과 마지막 오프셋까지의 차이를 의미한다
```CLI
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group hello-group --describe
```

- 오프셋 리셋(`--reset-offset`)
	- `--to-earliest`: 가장 처음 오프셋으로 리셋
	- `--to-latest`: 가장 마지감 오프셋으로 리셋
	- `--to-current`: 현 시점 기준 오프셋으로 리셋
	- `--to-datetime {YYYY-MM-DDTHH:mmSS.sss}`: 특정 일시로 오프셋 리셋(레코드 타임스탬프 기준)
	- `--to-offset {long}`: 특정 오프셋으로 리셋
	- `--shift-by {+/-long}`: 현재 컨슈머 오프셋에서 앞뒤로 옮겨서 리셋
```CLI
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group hello-group --topic hello.kafka --reset-offsets --to-earliest --execute
```

## 그 외 커맨드 라인 툴
---
### kafka-producer-perf-test.sh
- 카프카 프로듀서로 퍼포먼스를 측정할 때 사용한다
![image](https://user-images.githubusercontent.com/60502370/204707967-170cd2db-e069-4ae1-bfb5-26eb825cd530.png)

### kafka-consumer-perf-test.sh
- 카프카 컨슈머로  퍼포먼스를  측정할 때 사용된다
- 카프카 브로커와 컨슈머간의 네트워크를 체크할 때 사용할 수 있다
![image](https://user-images.githubusercontent.com/60502370/204708156-2d24c0d2-0588-4890-9861-568b120233af.png)

### kafka-reassign-partitions.sh
- 리더 파티션과 팔로워 파티션의 위치를 변경할 수 있다
- 특정 브로커에 파티션이 몰려 있는 핫스팟 현상을 해결 할 때 사용할 수 있다
- 카프카 브로커에는 `auto.leader.rebalance.enable` 옵션이 있는데 이 옵션의 기본값은 true로, 클러스터 단위에서 리더 파티션을 자동 리밸런싱하도록 한다
- 브로커의 백그라운드 스레드가 일정한 간격으로 리더의 위치를 파악하고 필요시 리밸런싱 한다

![image](https://user-images.githubusercontent.com/60502370/204708464-96f23c45-e514-436e-b383-5056f61bb7ae.png)
![image](https://user-images.githubusercontent.com/60502370/204708532-90dd72f0-1710-4264-b9fe-80f70713704b.png)

### kafka-delete-record.sh
- 레코드를 삭제한다
- `offset` 에 지정된 오프셋까지 레코드를 삭제한다

![image](https://user-images.githubusercontent.com/60502370/204708725-892c1bf4-afc9-457f-b746-43b74c45b5bc.png)

### kafka-dumb-log.sh
- 카프카 로그 파일 정보를 읽는다
![image](https://user-images.githubusercontent.com/60502370/204708892-aa44544b-fd66-4359-ac39-c85bfd611295.png)

# 토픽을 생성하는 두 가지 방법
---
1. 카프카 컨슈머 또는 프로듀서가 카프카 브로커에 생성되지 않은 토픽에 대해 데이터를 요쳥할 떄
   - **allow.auto.create.topics=true** 로 설정되어 있으면, 존재하지 않는 토픽에 데이터를 요청할 때 새로 만든다

2. 커맨드 라인 툴로 명시적으로 토픽을 생성
   - 토픽을 효과적으로 유지보수하기 위해서는 토픽을 명시적으로 생성하는 것을 추천한다
   - 데이터의 특성에 따라 토픽 설정을 다르게할 수도 있다
	   - 동시 처리량이 많아야 하는  토픽에는 파티션을 100개를 줄수도 있다
	   - 단기간 데이터만 필요한 경우에는 데이터 보관기간 옵션을 짧게 줄 수도 있다
	     
# 카프카 브로커와 CLI 버전을 맞춰야 한다
---
- 카프카 브로커로 커맨드 라인 툴 명령을 내릴 때 브로커의 버전과 커맨드 라인 툴 버전을 맞추는 것을 권장한다
- 브로커 버전이 업그레이드됨에 따라서 커맨드 라인 툴의 상세 옵션이 달라지기 때문이다