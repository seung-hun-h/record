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
- `--group`
	- 해당 옵션을 사용하면 컨슈머 그룹 기반으로 `kafka-console-consumer`가 동작한다
	- 컨슈머 그룹이란 특정 목적을 가진 컨슈머들을 묶음으로 사용하는 것을 뜻한다
	- 컨슈머 그룹으로 토픽의 레코드를 가져갈 경우 어느 레코드까지 읽었는지에 대한 데이터가 카프카 브로커에 저장된다
