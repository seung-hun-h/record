## 카프카 주요 옵션 알아보기
---

## Producer Options

bootstrap.servers

카프카 클러스터는 클러스터 마스터라는 개념이 없으므로, 클러스터 내 모든 서버가 클라이언트의 요청을 받을 수 있음. 클라이언트가 카프카 클러스터에 처음 연결하기 위한 호스트와 포트정보를 나타 냄.

client.dns.lookup

하나의 호스트에 여러 IP를 매핑해 사용하는 일부 환경에서 클라이언트가 하나의 IP와 연결하지 못할 경우에 다른 IP로 시도하는 설정. use_all_dns_ips가 기본값으로, DNS에 할당된 호스트의 모든 IP를 쿼리하고 저장. 첫 번째 IP로 접근 실패 시, 종료하지 않고 다음 IP로 접근 시도. resolve_canonical_bootstrap_servers_only 옵션은 Kerberos 환경에서 FQDN(Fully Qualified Domain Name)을 얻기 위한 용도로 사용 됨.

acks

프로듀서가 카프카 토픽의 리더 측에 메시지를 전송한 후 요청을 완료하기를 결정하는 옵션. 0, 1, all(-1)로 표현하며, 0은 빠른 전송을 의미하지만, 일부 메시지 손실 가능성 있음. 1은 리더가 메시지를 받았는지 확인하지만, 모든 팔로워를 전부 확인하지 않음. 대부분 기본값으로 1을 사용. all은 팔로워가 메시지를 받았는지 여부 확인. 다소 느릴 수 있으나 하나의 팔로우가 있는 한 메시지 손실은 없음.

buffer.memory

프로듀서가 카프카 서버로 데이터를 보내기 위해 잠시 대기(배치 전송이나 딜레이 등)할 수 있는 전체 메모리 바이트(byte)

compression.type

프로듀서가 메시지 전송 시 선택할 수 있는 압축 타입. none, gzip, snappy, lz4, zstd 중 선택

enbale.idempotence

설정을 true로 하는 경우 중복 없는 전송이 가능하며, 이와 동시에 max.in.flight.requests.per.connection은 5 이하, retries는 0 이상, acks는 all로 설정해야 함.

max.in.flight.requests.per.connection

하나의 커넥션에서 프로듀서가 최대한 ACK 없이 전송할 수 있는 요청 수. 메시지의 순서가 중요하다면 1로 설정할 것을 권장하지만 성능은 다소 떨어짐.

retries

일시적인 오류로 인해 전송에 실패한 데이터를 다시 보내주는 횟수

batch.size

프로듀서는 동일한 파티션으로 보내는 여러 데이터를 함께 배치로 보내려고 시도. 적절한 배치 크기 설정은 성능에 도움을 줌.

linger.ms

배치 형태의 메시지를 보내기 전 추가적인 메시지를 위해 기다리는 시간을 조정하고, 배치 크기에 도달하지 못한 상황에서 linger.ms 제한 시간에 도달 했을 때 메시지를 전송.

transactional.id

‘정확히 한 번 전송’을 위해 사용하는 옵션. 동일한 TransactionalId에 한해 정확히 한 번을 보장. 옵션을 사용하기 전 enable.idempotence를 true로 설정해야 함.

---

### Consumer Option

bootstrap.servers

프로듀서와 동일하게 브로커의 정보를 입력.

fetch.min.bytes

한 번에 가져올 수 있는 최소 데이터 크기. 지정한 크기보다 작은 경우, 요청에 응답하지 않고 데이터가 누적될 때까지 대기.

group.id

컨슈머가 속한 컨슈머 그룹을 식별하는 식별자. 동일한 그룹 내의 컨슈머 정보는 모두 공유 됨.

heartbeat.interval.ms

하트 비트가 있다는 것은 컨슈머의 상태가 active임을 의미. session.timeout.ms와 밀접한 관계가 있으며, session.timeout.ms보다 낮은 값으로 설정해야 함. 일반적으로 session.timeout.ms의 1/3로 설정.

max.partition.fetch.bytes

파티션 당 가져올 수 있는 최대 크기

session.timeout.ms

이 옵션을 이용해, 컨슈머가 종료된 것인지를 판단. 컨슈머는 주기적으로 하트 비트를 보내야 하고, 만약 이 시간 전까지 하트 비트를 보내지 않았다면 해당 컨슈머는 종료된 것으로 간주하고 컨슈머 그룹에서 제외 후, 리밸런싱을 시작 함.

enable.auto.commit

백그라운드로 주기적으로 오프셋을 커밋(커밋은 데이터의 현재 위치를 업데이트 하는 것. poll이 발생하면 자동 커밋)

auto.offset.reset

카프카에서 초기 오프셋이 없거나 현재 오프셋이 더 이상 존재하지 않는 경우에 다음 옵션으로 reset 함.

-   earliest: 가장 초기의 오프셋값으로 설정
-   latest: 가장 마지막 오프셋값으로 설정
-   none: 이전 오프셋값을 찾지 못하면 에러를 나타냄.

fetch.max.bytes

한 번의 가져오기 요청으로 가져올 수 있는 최대 크기

group.instance.id

컨슈머의 고유 식별자. 만약 설정한다면 static 멤버로 간주되어, 불필요한 리밸런싱을 하지 않음.

isolation.level

트랜잭션 컨슈머에서 사용되는 옵션.

-   read_uncommited(Default) : 모든 메시지를 읽음
-   read_committed : 트랜잭션이 완료된 메시지만 읽음

max.poll.records

한 번의 poll() 요청으로 가져오는 최대 메시지 수.

partition.assignment.strategy

파티션 할당 전략.(Default : range)

fetch.max.wait.ms

fetch.min.bytes에 의해 설정된 데이터보다 적은 경우 요청에 대한 응답을 기다리는 최대 시간.