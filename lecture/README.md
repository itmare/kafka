kafka 정리중

<br><br><br>

---

Kafka 개요
==========

---

### Messaging System

-	비동기 통신 프로토콜을 지닌, 즉각적인 응답이 필요하지 않은 큐 시스템
-	email은 대표적인 비동기 메시징
-	메시지 큐는 서로 독립적으로 분리되어 처리
-	다른 프로세스에 영향없이, 메시지를 큐에 보내고 꺼낸다.
-	대용량 처리를 하기 위한 배치 작업, 채팅, 비동기 작업등에 이용

### pub/sub 모델

-	publish, subscribe
-	메시지에 특정한 수신자가 정해져 있지 않음
-	구독을 신청한 수신자에게 전달

### 카프카?

-	카프카는 분산, replication 로그 서비스
-	독특한 디자인 아니지만 pub/sub 메시징 시스템 기능
-	프로듀서, 컨슈머가 분리되어 있음
-	브로커라 불리는 하나 또는 여러대의 서버 클러스터로 동작
-	실시간 데이터 파이프 라인 구성
-	실시간 스트리밍 애플리케이션 구성

### 특징

<img src="./pictures/kafka-streaming-platform.png" width="500" assign="center">

-	High Throughput
-	실시간 로그 통합
-	무중단
-	다른 툴과의 호환성
-	간단한 scale-out
-	producer와 consumer 역할 분리

![kafka-streaming-platform-linkedin](./pictures/kafka-streaming-platform-linkedin.png)

-	**offset < partition < topic < broker < kafka cluster**

### zookeeper?

-	코디네이션 어플리케이션
-	"분산시스템 개발 + 코디네이션 개발"은 비효율적
-	하둡의 서브 프로젝트
-	클러스터라 부르지 않고, 앙상블
-	과반수 방식, 주키퍼 서버 수는 항상 홀수여야 함
-	계층형 구조
-	0.9버젼부터 컨슈머와 주키퍼 연결 끊음, 대신 오프셋 토픽으로...

<br><br>

##### - 주키퍼는 5대가 대체적으로 좋음 (5, 7, 9대 별 차이 없음)

-	처리량 그래프
	-	ZooKeeper release 3.2 running on servers
	-	dual 2Ghz Xeon
	-	two SATA 15K RPM drives ![zookeeper-server-num](./pictures/zookeeper-server-num.png)<br><br>

##### - 읽기는 로컬에서 처리, 쓰기는 리더

![zookeeper-struc](./pictures/zookeeper-struc.png) ![zookeeper-struc2](./pictures/zookeeper-struc2.png)

##### - 주키퍼와 카프카 구조

![zookeeper-kafka-struc](./pictures/zookeeper-kafka-struc.png)

<br><br><br><br><br>

---

Kafka 디자인
============

---

### motivation

-	대량의 실시간 데이터를 다루기 위해 카프카 개발
-	여러가지 사례들을 통해 고민
-	실시간 로그 통합 같은 이벤트를 처리하기 위해 높은 처리량 필요
-	빠른 전송 속도 보장
-	분산, 분할, 실시간 처리를 위해 생겨난 것이 파티셔닝과 컨슈머 모델
-	장비의 장애 상황이라도 안정적인 서비스 유지

### Persistence

-	디스크는 느리다라는 선입견
-	RAIDS 7200rpm디스크는 linear write 초당 600MB, random write 초당 100k
-	linear read와 write는 OS에 의해 최적화되어 있다.
-	OS는 read-ahead(미리읽기)와 write-behind(데이터를 메모리 버퍼에 임시 저장한 후 한번에 저장)를 제공
-	OS는 최근 적극적으로 디스크 캐싱에 메인 메모리를 사용하도록 하고 있다.
-	메모리가 반환될 때 약간의 성능 저하가 있지만, 사용가능한 모든 메모리를 디스크 캐싱으로 전환하려고 한다.
-	모든 디스크의 read와 write는 이 캐싱을 통하게 된다.
-	JVM에서 JAVA를 사용해 본 사람은 아래 두가지를 경험
	-	객체의 메모리 오버헤드가 높다.
	-	Java Garbage Collection은 heap 데이터가 증가할 수록 불편하고, 느리다.
-	여러 이유들로 페이지 캐시에 의존하는 것이 유리하므로 페이지 캐시를 사용한다.
-	서비스가 재시작되어도, 새로 캐시가 메모리에 만들어지거나 캐시 초기화를 하더라도 패이지 캐시는 유지된다.
-	모든 데이터는 반드시 디스크로 내리지 않고, 페이지 캐시로 저장된다. ![kafka-page-cache01](./pictures/kafka-page-cache01.png) ![kafka-page-cache02](./pictures/kafka-page-cache02.png)

### Page Cache

-	준비중

### dirty page

-	준비중

### free -m

-	실행 시, 표시 정보

```json
      total   used  free  shared  buff/cache  available
Mem:  31896   9776  1811  1617    20309       20016
Swap: 10239   0     10239
```

-	total: 전체 메모리
-	used: 시스템에서 사용하는 메모리
-	free: 시스템에서 아직 사용하지 않은 메모리
-	shared: 프로세스 사이에서 공유하는 메모리
-	buff/cache: 버퍼와 캐시
-	available: 상요 가능한 메모리
-	used보다 available이 더 크면, page cache에서 바로 할당해줌, 메모리가 부족한게 아님

### Effiiency

-	준비중

### Topic

-	캬프카 클러스터 내에서 메시지를 전송 또는 가져오기 위해 구분하는 단위
-	메일 서버의 메일주소와 같은 역할
-	토픽 이름은 249자 미만으로 영문, 숫자, .-_ 만 가능
-	prefix로 사용 (A-log, B-log 요런식으로..)

### Partition

-	토픽을 분할
-	성능을 높이기 위해 사용
-	파일 핸들러, 장애 복구 시간
-	많으면 성능이 빠를거다라는 생각은 오산, 그러나 파일 핸들러가 컨트롤해야하기 떄문에 같이 늘어남, 장애났을때 장애시간 늘어남, 파티션의 리더에게 읽고 쓰기, 파티션이 많으면 파티션 마다 리더정보가 있는데 장애나면 주키퍼에서 장애정보가져오는데만해도 시간소요가 많음, 적절한 파티션수 설정 중요
-	파티션을 늘리는건 운영중에도 가능, 노프랍
-	파티션을 줄이는건 불가능, 꼭 줄여야 된다. (삭제 방법 밖에 없음)

![partition01](./pictures/partition01.png)

메시지를 하나씩 보내면 카프카 뉴스 토픽에 들어가는데 4초

![partition02](./pictures/partition02.png)

뉴스 토픽을 파티션 4개로 쪼개버리고, 프로듀스도 4개 병렬로 메시지 보내기 총 1초

<br><br>

Message Delivery Semantics: Producer
------------------------------------

-	At most once - 메시지는 손실될 수 있다. 하지만 절대로 재전송 하지 않음
-	At least once - 메시지는 절대 손실되지 않는다. 하지만 중복 전송될 수 있다.
-	Exactly once - 사람들 대부분 원한다. 메시지는 한번만 전달
	-	굉장히 어렵고, exactly once를 보장한다고 하면서, 예외 경우를 두고 있다. (컨슈머나 프로듀서 실패 등)
-	카프카는 메시지가 "committed"라는 개념을 가지고 있어, 한번 퍼블리싱되어 커밋된 메시지는 브로커가 살아 있는 한 절대 손실되지 않는다.
-	0.11.0.0 이전 버전에서는 프로듀서가 응답을 받지 못하면 재전송을 해야 했고 이는 at least once 전송 방식이다.
-	0.11.0.0 이후 버전부터는 exactly once 지원, 중복이 발생하지 않도록 멱등성 전송 옵션을 지원한다.
-	프로듀서는 설정을 통하여 리더가 받기를 기다릴 수 있고, 비동기로 처리할 수 있다.

<br><br>

Message Delivery Semantics: Consumer
------------------------------------

-	컨슈머는 메시지 로그에서 자신의 위치를 조정
-	컨슈머는 절대 crash하지 않으면 자신의 위치를 메모리에 저장할 수 있지만, 컨슈머가 fail하면 다른 프로세스에 의해 이 메시지의 위치를 가져가길 원한다.
-	다른 프로세스가 어느 위치부터 시작할 것인지를 선택해야 한다.
-	컨슈머가 메시지를 읽는다. -> 그 위치를 저장 -> 메시지를 처리(최대 한번)
	-	컨슈머가 위치를 저장한 후, 메시지를 처리하기 전에 crash 발생
	-	일부 메시지가 처리되지 않은 상태로 대체된 프로세스가 저장된 위치부터 시작
	-	at-most-once 동작 방식과 같다.

#### consumer의 at-most-once 동작 방식

![consumer-at-most-once01](./pictures/consumer-at-most-once01.png)

-	5개의 메시지를 가지고 있는 broker1 <br><br>

![consumer-at-most-once02](./pictures/consumer-at-most-once02.png)

-	컨슈머가 메시지1~3을 읽어 온다. (위치 기록)<br><br>

![consumer-at-most-once03](./pictures/consumer-at-most-once03.png)

-	다음 읽을 건 메시지4 라고 표시<br><br>

![consumer-at-most-once04](./pictures/consumer-at-most-once04.png)

-	메시지1 저장 후, 컨슈머가 죽는 상황<br><br>

![consumer-at-most-once05](./pictures/consumer-at-most-once05.png)

-	컨슈머 다시 살아남, 메시지2~3이 컨슈머에서 사라짐 <br><br>

![consumer-at-most-once06](./pictures/consumer-at-most-once06.png)

-	컨슈머 메시지4 부터 메시지 읽기 시작<br><br>

-	컨슈머가 메시지를 읽는다 -> 메시지를 처리 -> 그 위치를 저장

	-	컨슈머가 메시지를 처리한 후 crash 발생
	-	하지만 그 포지션은 이미 저장되어 있는 위치
	-	다른 프로세스는 이미 처리된 위치 부터 다시 시작
	-	at-least-once 동작 방식과 같다.

#### consumer의 at-least-once 동작 방식

![consumer-at-least-once01](./pictures/consumer-at-least-once01.png)

-	5개의 메시지를 가지고 있는 broker1 <br><br>

![consumer-at-least-once02](./pictures/consumer-at-least-once02.png)

-	브로커로 부터 메시지1~3 읽음 (위치기록 안함)<br><br>

![consumer-at-least-once03](./pictures/consumer-at-least-once03.png)

-	메시지를 저장하고 위치를 기록 <br><br>

![consumer-at-least-once04](./pictures/consumer-at-least-once04.png)

-	메시지2를 저장하려고 보냄<br><br>

![consumer-at-least-once05](./pictures/consumer-at-least-once05.png)

-	메시지2의 저장이 완료전에 컨슈머가 죽음 (현재 위치 메시지2)<br><br>

![consumer-at-least-once06](./pictures/consumer-at-least-once06.png)

-	메시지2가 저장됐지만 현재 위치는 여전히 메시지2<br><br>

![consumer-at-least-once07](./pictures/consumer-at-least-once07.png)

-	컨슈머는 브로커로부터 다시 메시지2부터 가져온다. (중복)<br><br>

<br><br><br><br><br>
--------------------

Replication
===========

---

-	각 토픽의 파티션 로그를 replication factor 갯수로 브로커들로 복제
-	클러스터의 브로커가 고장 났을 때 장애 복구 가능
-	다른 메세징 시스템에도 복제가 있긴 하지만 처리량에 큰 영향을 주곤 한다.
-	복제의 단위는 토픽이 아닌 파티션
-	리더와 팔로워가 있고 모든 읽기 쓰기는 리더가 담당
-	리더와 팔로워는 모두 동일하다(로그의 끝의 일부는 리더와 팔로워가 일치하지 않을 수 있다.)

-	대부분의 분산 시스템들과 마찬가지로 자동으로 장애를 처리하기 위해 "alive"정의가 필요

	-	주키퍼와 세션을 유지하고 있어야 함
	-	슬레이브는 리더로부터 모든 것을 복제해야 한다. 그리고 너무 떨어지면 안된다.

-	파티션에 ISR그룹 모두에게 적용되었을 때 commited라고 정의 한다.

-	커밋된 메시지만 컨슈머에게 전달한다.

-	카프카는 커밋된 메시지에 대해 손실되지 않는다는 것을 보장한다.

-	리더가 골고루 분산되어 있는지 확인 항상 필요!

-	<br><br>준비중<br><br>

### ISR (In Sync Replica)

-	isr 구성원만이 리더가 될 수 있다.

-	\*\** rabbitMq는 리더와 가장 오랫동안 싱크하고 있던 놈이 마스터가 된다. (즉, 가장 오래된 놈을 체크하고 있다;)

-	다운된 파티션이 다시 올라온 후 리더와 동기화가 다시 되면, isr리더가 될 자격이 주어진다. (리더와 팔로워)

-	isr에 들어오고 나가는거는 유저가 컨트롤 하는게 아니라, 브로커의 내부적 로직에 의해 컨트롤

### Controller

-	카프카 클러스터는 하나의 토픽, 하나의 파티션만 다루는게 아니다. 수백 수천의 파티션을 관리한다.
-	소수의 브로커로 쏠리는 현상을 방지하기 위해 클러스터 내부에서 파티션을 round-robin방식으로 균형을 맞추려 한다.
-	리더를 선출하는 과정을 최적화하는 것은 매우 중요
-	노드 장애로 리더 역할을 하고 있던 모든 파티션에 대해 역할이 끝나게 되면, 리덛 선출이 동작한다.
-	클러스터 내 브로커 중 하나가 컨트롤러(주키퍼)역할을 하고 새로운 리더를 결정하고 모든 브로커에게 전달
-	배치로 빠르고 정확하게 처리하기 위해 컨트롤러 동작

### Znode

-	ZooKeeper가 제공해주는 파일시스템에 저장되는 파일 하나하나를 znode라고 부른다.
-	unix의 파일 시스템처럼 node간에 hierarchy namespace를 가지고, /(슬레쉬)를 사용한다.
-	기존 파일 시스템과의 차이점은 zookeeper는 file과 directory의 개념이 없어 znode하나만 쓰인다.

![znode](./pictures/znode.png)

### RabbitMQ

-	준비중

<br><br><br><br><br>

---

Producer와 Consumer
===================

---

Producer
--------

-	주요 옵셜 설정

```shell
bootstrap.servers #브로커에 모든 호스트네임을 다 넣어주는 걸 추천
buffer.memory # 메시지를 보내기전에 잠깐 버퍼에 담아둘수 있는 사이즈
compression.type # 압축을 하면 효율성이 좋음
retries # 실패 시, 재시도 횟수
batch.size # 얼마만큼 배치로 해서 보낼건지 사이즈 설정
linger.ms # 지연시간, 얼마나 대기했다가 보낼지,
acks # 통신 방식
```

### Producer ACKS

#### ACKS = 0

-	매우 빠르게 전송할 수 있지만, 파티션의 리더가 받았는지는 알 수 없음
-	메시지 유실가능성 높음

![producer-ack-0-01](./pictures/producer-ack-0-01.png)

-	토픽의 리더로 A를 보내면 프로듀서의 역할은 끝 (A가 갔든 안갔든 돈케어)

![producer-ack-0-02](./pictures/producer-ack-0-02.png)

-	토픽의 리더로 B를 보내면 프로듀서의 역할은 끝 (B가 갔든 안갔든 돈케어)

![producer-ack-0-03](./pictures/producer-ack-0-03.png)

-	토픽의 리더로 C를 보내면 프로듀서의 역할은 끝 (C가 갔든 안갔든 돈케어)

<br><br>

#### ACKS = 1 (Strongly Recommend)

-	메시지 전송동 빠른 편이고, 파티션의 리더가 받았는지 확인한다.
-	가장 많이 사용되고, 대부분 기본값으로 설정되어 있다. (filebeat등의 프로듀서 앱 같은거)
-	프로듀서 앱 사용 시, 반드시 acks의 기본값이 어떤 걸로 설정되어있는지 확인 필요

![producer-ack-1-01](./pictures/producer-ack-1-01.png)

-	프로듀서는 A라는 메시지를 보낼 준비한다.<br><br>

![producer-ack-1-02](./pictures/producer-ack-1-02.png)

-	A를 리더에게 보낸다<br><br>

![producer-ack-1-03](./pictures/producer-ack-1-03.png)

-	메시지가 잘 들어갔으면 리더는 프로듀서에게 ack를 보낸다.<br><br>

![producer-ack-1-04](./pictures/producer-ack-1-04.png)

-	B를 보낼 준비한다.<br><br>

![producer-ack-1-05](./pictures/producer-ack-1-05.png)

-	A는 내부적으로 replication일어나고, 리더는 B를 받는다.<br><br>

![producer-ack-1-06](./pictures/producer-ack-1-06.png)

-	리더는 프로듀서에게 B를 잘 받았다고 ack를 보낸다.<br><br>

![producer-ack-1-07](./pictures/producer-ack-1-07.png)

-	내부적으로 B의 replication을 준비한다.<br><br>

![producer-ack-1-08](./pictures/producer-ack-1-08.png)

-	프로듀서는 C 준비, 근데 B복제의 찰나 rack2가 죽는다.<br><br>

![producer-ack-1-09](./pictures/producer-ack-1-09.png)

-	리더가 rack1로 변경된다.<br><br>

![producer-ack-1-10](./pictures/producer-ack-1-10.png)

-	프로듀서는 C를 새로운 리더에게 보낸다.<br><br>

![producer-ack-1-11](./pictures/producer-ack-1-11.png)

-	뉴리더는 c를 받는다.<br><br>

![producer-ack-1-12](./pictures/producer-ack-1-12.png)

-	뉴리더는 C에 대한 ack를 프로듀서에게 보내고, 남아있는 팔로우(rack3)에게 replication한다.<br><br>

![producer-ack-1-13](./pictures/producer-ack-1-13.png)

-	프로듀서는 D를 준비한다.<br><br>

![producer-ack-1-14](./pictures/producer-ack-1-14.png)

-	D의 ack를 보낸 후, replication한다.<br><br>

![producer-ack-1-15](./pictures/producer-ack-1-15.png)

-	이때, rack2가 복구되고, 팔로우로 합류한다.<br><br>

![producer-ack-1-16](./pictures/producer-ack-1-16.png)

-	new리더(rack1)는 old리더(rack2:현재는팔로워)에게 replication을 한다. <br><br>

<br><br>

#### ACKS = ALL

-	메시지 전송은 느리지만, 손실 없는 메시지 전송이 가능하다.
-	min.insync.replicas 옵션은 프로듀서가 acks=all로 설정하여 메시지를 보낼 때, write를 성공하기 위한 최소 복제본의 수를 의미한다. (브로커의 옵션임: server.properties, default: 1) -

![producer-ack-all-01](./pictures/producer-ack-all-01.png)

-	프로듀서는 ack와 상관없이 A를 리더에게 쏜다.<br><br>

![producer-ack-all-02](./pictures/producer-ack-all-02.png)

-	min.insync.replicas(최소 replica갯수)의 설정 적용<br><br>

![producer-ack-all-03](./pictures/producer-ack-all-03.png)

-	replica갯수가 2이기때문에 하나를 팔로워에게 replication<br><br>

![producer-ack-all-04](./pictures/producer-ack-all-04.png)

-	리더는 replication체크를 하고 프로듀서에게 ack를 보낸다. <br><br>

![producer-ack-all-05](./pictures/producer-ack-all-05.png)

-	replica갯수가 3일 경우,<br><br>

![producer-ack-all-06](./pictures/producer-ack-all-06.png)

-	A가 총갯수가 3이 되고, 리더는 체크한다.<br><br>

![producer-ack-all-07](./pictures/producer-ack-all-07.png)

-	리더는 프로듀서에게 ack를 보낸다.<br><br>

![producer-ack-all-08](./pictures/producer-ack-all-08.png)

-	만약, 3개의 브로커중에 하나가 죽었다면, <br><br>
-	즉, 요구조건 충족 하지 못하면, 프로듀서가 쏘는데로 에러가 난다.
-	ack 1로 초당 100개, all 초당 30개 복제 체크때문에.

<br><br>

#### KEY를 이용한 전송

-	특정 파티션에 쏠때, key를 이용
-	key 내용을 해쉬해서 그 key시작하는 건 특정 파티션에만 들어가게 한다.
-	user id같은 유니크한 값들을 받아서 쓸때, 유저아이디를 키값으로 쓰는 경우가 종종 있다.

![key-value-01](./pictures/key-value-01.png)

-	A는 파티0으로, B는 파티션1로 정의해서 보낼 순 없다.

<br><br>

Consumer
--------

-	컨슈머는 파티션의 리더에게 "fetch" 요청을 하는 역할
-	컨슈머는 위치를 기록하고 있는 offset으로부터 메시지를 가져온다.
-	처음 설계 시, PUSH vs PULL 사이에서의 개발자들이 고민했다.
	-	push를 하게 되면 컨슈머가 아무리 빨리 받아도, 보내는 주체에 따라 느려짐
	-	내가 push를 하게 되면 컴파운트가 어플리케이션이 하나하나 늘어날때마다 모두 체크해줘야 함.
-	push방식의 경우 브로커가 데이터 전달되는 속도를 제어
-	다양한 컨슈머들을 다루는데 어려움
-	컨슈머의 목적은 컨슈머가 가능한 최대 속도로 가져갈 수 있도록 하는 것
-	pull바식을 이용함으로서, 일시적인 컨슈머 멈춤도 가능 (문제가 생기면 잠시 중단도 가능)
-	특정 위치 부터 메시지를 가져오기 때문에 불필요한 대기 시간이 없다.
-	pull방식의 단점은 데이터가 없으면 계속해서 loop가 돌 수 있다.
-	방지하기 위해 poll에 파라미터를 추가
-	메시징 시스템에서 어디까지 메시지를 가져갔는지를 추적하는 것은 매우 중요
-	대부분의 메시징 시스템들은 메타데이터를 브로커에 보관
-	컨슈머가 메시지를 가져가면, 즉시 로컬에 기록할지 또는 컨슈머의 ack를 기달릴지 선택
-	메시지를 가져간 것을 알고 즉시 삭제하면 데이터를 적게 유지할 수 있다.
-	브로커가 컨슘된 메시지를 기록하는 경우 컨슈머가 장애등으로 처리하지 못하면 일부 메시지를 손실하게 된다.
-	이를 해결하기 위해 ack를 추가
-	컨슈머의 처리가 끝나면 ack를 보내 메시지 손실 문제를 해결
-	중복과 성능의 이슈

### Not Ack

![consumer-ackoff-01](./pictures/consumer-ackoff-01.png)

-	5개의 메시지를 가진 broker1이 있다.

![consumer-ackoff-02](./pictures/consumer-ackoff-02.png)

-	consumer가 메시지1~3을 pull하자마서 broker에 있는 메시지 삭제

![consumer-ackoff-03](./pictures/consumer-ackoff-03.png)

-	컨슈머가 메시지를 처리

![consumer-ackoff-04](./pictures/consumer-ackoff-04.png)

-	메시지1을 처리하고 메시지2~3을 가지고 있던 컨슈머가 죽음 (메시지2~3은 사라짐)

![consumer-ackoff-05](./pictures/consumer-ackoff-05.png)

-	새로운 컨슈머 등장

![consumer-ackoff-06](./pictures/consumer-ackoff-06.png)

-	컨슈머는 브로커에서 메시지를 다시 가져오기 시작 (메시지4~5)

![consumer-ackoff-07](./pictures/consumer-ackoff-07.png)

-	컨슈머가 메시지4~5 처리 (결과적으로 메시지2~3은 손실)

<br><br>

### Ack

![consumer-ackon-01](./pictures/consumer-ackon-01.png)

-	브로커에 5개의 메시지 있음

![consumer-ackon-02](./pictures/consumer-ackon-02.png)

-	컨슈머가 메시지1~3을 가져갔지만, 브로커에서 삭제 하지 않음

![consumer-ackon-03](./pictures/consumer-ackon-03.png)

-	컨슈머가 처리한 메시지1에 대해서 ack를 브로에게 보냄

![consumer-ackon-04](./pictures/consumer-ackon-04.png)

-	ack확인 후, 브로커는 메시지1을 삭제

![consumer-ackon-05](./pictures/consumer-ackon-05.png)

-	컨슈머가 메시지2를 처리한 후에 ack가 브로커에게 보내지지 않음

![consumer-ackon-06](./pictures/consumer-ackon-06.png)

-	브로커가 아직 메시지2에 대한 ack를 받지 못했는데, 컨슈머가 죽음

![consumer-ackon-07](./pictures/consumer-ackon-07.png)

-	새로운 컨슈머 등장

![consumer-ackon-08](./pictures/consumer-ackon-08.png)

-	새로운 컨슈머는 메시지2~4를 pull하고, 메시지2은 스토리지에 이미 저장되어 있음, 컨슈머가 메시지2를 전달하게 되면 중복대한 이슈 발생

<br><br>

-	카프카는 이러한 처리를 기존 메시징 시스템과 다르게 처리
-	토픽은 파티션 세트로 구성되고, 하나의 파티션은 컨슈머 그룹의, 하나의 컨슈머만 메시지를 가져간다.
-	각각의 파티션의 가져갈 위치는 하나의 정수로 표현되고 컨슈머는 이 오프셋 이후부터 메시지를 가져간다.
-	장점으로는 매우 단순하고, 빠르며, 오프셋의 위치를 변경하여 앞 또는 뒤부터 메시지를 ㄱ져갈 수 있다.

<br><br>

### 오더링 이슈

-	파티션 하나 일때,

	-	오프셋 순서대로 받는다. ![offset-order-issue01](./pictures/offset-order-issue01.png)

-	파티션 두개이상 일때,

	-	전체 메시지 순서대로 받을 순 없다. 단, 각 파티션 별로 오프셋 순서대로 받는다. ![offset-order-issue02](./pictures/offset-order-issue02.png)

<br><br><br>

### Consumer Group

-	토픽을 구독하는 애플리케이션이 있다고 가정
-	그 애플리케이션은 구독하고, 메시지를 가져오기 시작하고, 유효한지 확인하고 결과를 기록
-	잘 동작할 수 있지만, 프로듀서가 토픽으로 쓰는 비율을 높인다면?
-	컨슈머는 프로듀서의 속도를 따라가지 못하게 된다.
-	확장할 수 있어야 한다. (확장하기 위해 컨슈머그룹이란 개념을 사용)

<br>

-	준비중-시작
-	목적
	-	장애에 따른 가용성
	-	그룹별 offset 관리 가능
-	컨슈머그룹 만들때, 동일한 이름의 컨슈머그룹이 존재하는지 체크해야함
-	준비중-끝

<br>

##### 컨슈머그룹과 파티션 수의 관계

-	각각의 파티션은 하나의 컨슈머그룹에 하나의 컨슈머하고만 연결된다.

![consumer-group01](./pictures/consumer-group01.png) ![consumer-group02](./pictures/consumer-group02.png)

-	컨슈머가 하나였다가 3개로 늘어나면, 리발란싱이 일어난다.

![consumer-group03](./pictures/consumer-group03.png)

-	컨슈머 속도가 느려서 다빠르게 하고 싶어서 컨슈머 하나 추가하면?
	-	추가된 컨슈머는 논다.

![consumer-group04](./pictures/consumer-group04.png)

-	컨슈머4에서 장애가 나면, 컨슈머그룹내에서는 모든 정보를 공유해 준다.

![consumer-group05](./pictures/consumer-group05.png)

-	컨슈머 그룹에선 자체적으로 리발란싱 일어난다.

![consumer-group06](./pictures/consumer-group06.png)

-	각 파티션이 초당 10개의 메시지 쏘고, 컨슈머는 초당 100개를 받는다. (문제없다.)

![consumer-group07](./pictures/consumer-group07.png)

-	갑자기 초당 400개의 메시지를 쏴서, 긴급히 컨슈머 2개 추가했음에도 불구하고 총 초당300개의 메시지만 가져감, 결과적으로 초당 100개의 메시지가 쌓인다.

![consumer-group08](./pictures/consumer-group08.png)

-	파티션을 하나 늘려주고, 컨슈머도 하나 늘려서 쌓일 가능성 있는 메시지를 처리한다.
-	결과적으로 컨슈머를 늘려주기 위해 파티션을 늘린다.

![multi-consumer](./pictures/multi-consumer.png)

-	기본적으로 카프카는 메시지를 default로 7일간 보관
-	컨슈머 그룹을 이용해 멀티 컨슈머 가능

<br><br>

### Consumer Rebalance

-	컨슈머 그룹에 새로운 컨슈머가 조인되면 리발란싱
-	컨슈머 그룹에 컨슈머가 다운되면 리발란싱
-	컨슈머가 alive 상태인지 확인 필요(heartbeats)
-	그룹 코디네이터가 따로 있음

<br><br>

### Consumer 주요 옵션

-	bootstrap.servers : 브로커 리스트
-	group.id :
-	fetch.min.bytes : 최소 얼마나 가져올지
-	fetch.max.wait.ms : 얼마나 기다릴지
-	session.timeout.ms / 3 : 하트비트와 관련
-	heartbeat.interval.ms / 1 : 하트비트와 관련
	-	session.timeoutms : heartbeat.interval.ms = 3 : 1 비율이 추천값
-	auto.offset.reset : 컨슈머가 오프셋 기준으로 메시지를 가져오는데 오프셋이 없는 경우가 있다. (오프셋4번까지 가져온상황에서 하둡에 붙였을때, 그럴때 자동으로 어떤값을 쓸거냐 정함)
-	enable.auto.commit :
-	auto.commit.interval.ms :
-	max.poll.records :

<br><br>

### Commit

-	컨슈머가 파티션의 몇번째 offset까지 가져왔는지 표시
-	이러한 행동을 commit이라 함
-	rebalancing이나 컨슈머 재시작 시 commit된 위치부터 시작
-	auto commit
	-	enable.auto.commit = true
	-	가장 많이 사용되고, 대부분 기본값으로 설정
-	manual commit 물론 가능

![auto-commit](./pictures/auto-commit.png)

-	2개 가져오는데 5초걸린다고 가정
-	오토커밋이 5초 주기로 커밋함

<br><br>

### auto-commit 이슈 1

![auto-commit-issue-01](./pictures/auto-commit-issue-01.png)

-	오토커밋하고 있는 중간에 컨슈머 다운되는 상황

	1.	4번까지 읽어가고, 마지막 커밋위치 4번 (하나당 2.5초)
	2.	3초정도 지났으면 5번 메시지는 이미처리되어서 저장소에도 저장되고 함
	3.	3초정도 됐을때, 컨슈머가 다운
	4.	3초지난 위치에서 리발란싱
	5.	마지막 커밋위치는 4번이므로 4번부터 메시지를 가져온다.
	6.	메시지 5번은 중복으로 처리될 수 있음 ㅠ

### auto-commit 이슈 2

![auto-commit-issue-02](./pictures/auto-commit-issue-02.png)

-	오토커밋하고 있는데 갑자기 메시지 못보내고 있는 상황

	1.	5번을 못가져가고 징징거리고 있는데 5초가 지남
	2.	그럼 커밋 위치를 6번으로 확 찍어버림
	3.	카프카에는 있다. 이 상태에서 컨슈머가 죽음
	4.	다시 컨슈머가 나타나면 커밋 6번부터 가져감
	5.	5,6번 날라감 ㅠ

<br>

-	auto-commit은 편리하지만 중복에 대한 이슈 발생 할 수 있음

-	기본적으로 굉장히 안정적으로 작동, 하지만 이런 이슈가 있을 수도 있다는 것 염두

<br><br><br><br><br>

---

운영 가이드
===========

---

.

.

<br><br><br><br><br>

---

Data Pipeline
=============

---

.

.

.

.

.

.

.

.

.

.

.

.

.

.

.

.

.

.

---

Trouble Shooting
================

---

### 브로커 한대가 죽음, 메시지가 안나간다. 계속 에러가 난다.

-	ack all로 쓰는지 확인, `min.insync.replicas`옵션과 관련지어 확인
