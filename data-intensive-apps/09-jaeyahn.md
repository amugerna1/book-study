# 09장 일관성과 합의

> 살아 있지만 틀린 게 나은가, 올바르지만 죽은 게 나은가?

분산 시스템에서는 많은 것이 잘 못 될 수 있다.
-> 분산 데이터베이스는 최종적 일관성을 제공한다.
- 데이터베이스에 쓰기를 멈추고 불 특정 시간동안 기다리면 모든 읽기 요청이 같은 값을 반환한다.

최종적 일관성을 지닌 데이터베이스에서 두 개의 다른 복제본은 서로 다른 데이터를 가지고 있을 수 있다.

# 선형성(Linearizability)

비선형적

<img width="646" alt="image" src="https://user-images.githubusercontent.com/8626130/199660561-9d801e71-7a5b-4802-91ff-ca4edf437014.png">

-> 엘리스와 밥이 각자의 스마트폰으로 월드컵을 시청한다.
-> 앨리스가 조회했을때는 팔로워 1에서 조회해서 우승했다고 안다.
-> 밥은 팔로워 2에서 조회를 하게되어서 우승한지 모른다.

> 앨리스가 밥에게 우승결과를 말해준다. 이때도 밥은 결과를 모른다. 팔로워 2는 아직 쓰고있기 때문이다.

만약 데이터 베이스가 복제본이 하나만 있는 척 하면 훨씬 단순하지 않을까? 두 개의 다른 복제본에 같은 질문을 동시에 하면 두가지 대답이 아니라,
한가지만 말하도록 말이다.

-> 선형성 시스템은 여기서 아이디어가 출발했다.

선형성 시스템에서는 클라이언트가 쓰기를 성공적으로 완료하자마자 그 데이터베이스를 읽는 모든 클라이언트는 새 값을 볼 수 있어야 한다.

클라이언트는 반드시 하나의 일관된 신 상태만을 관찰할 수 있어야 한다.

- 즉, 선형성은 최신성 보장이라고 할 수 있다.

<img width="643" alt="image" src="https://user-images.githubusercontent.com/8626130/199661394-f6b2c5c7-5245-468e-b10a-21bf463e53de.png">

과연 클라이언트 C가 작성하는 동안은 조회를 요청한 클라이언트 A와 B에게는 0을 반환해야하나, 아니면 1을 반환해야할까?


<img width="647" alt="image" src="https://user-images.githubusercontent.com/8626130/199661568-dff736ec-6c3d-46eb-8b81-aa4ad34296c5.png">

선형적으로는 최신성을 보장해야 함으로 0을 반환했다고 하면 1을 반환하는것은 __선택__이지만,
최신값인 1을 반환했던 이력이 있으면 항상 1을 반환하는 것은 __필수__이다.

> # 선형성 대 직렬성
> 직렬성은 모든 트랜잭션이 여러 객체를 읽고 쓸 수 있는 상황에서의 트랜잭션의 격리 속성이다.
> 선형성은 레지스터(개별 객체)에 실행되는 읽기와 쓰기에 대한 최신성 보장이다.
> 직렬성이 보장된다고 선형성이 반드시 보장되진 않는다.

선형성을 요구하는 곳들

## 잠금과 리더 선출
단일 리더 복제 시스템은 리더가 여러 개(스플릿 브레인)가 아니라 반드시 하나만 존재하도록 보장해야 한다.
리더를 선출하는 방법으로 잠금을 사용할 수 있고 이 잠금은 선형적이어야 한다.
분산 잠금과 리더 선출을 위해 아파치 주키퍼나 etcd 같은 코디네이션 서비스가 종종 사용된다.

## 제약 조건과 유일성 보장
데이터베이스에서 흔하게 사용되는 유일성 제약 조건도 잠금과 비슷하게 선형성을 필요로 한다.

## 채널간 타이밍 의존성
아까 위에서 보았던 앨리스와 밥사례에서는 앨리스가 밥에게 알려주지 않았다면
밥은 결코 본인이 늦은 결과를 조회하고 있다는 사실을 모른다.

이처럼 선형성 위반을 알릴 수 있는 채널이 있어야 하고,선형성을 구현하는 시스템들은 이런 채널에 의존한다.

<img width="631" alt="image" src="https://user-images.githubusercontent.com/8626130/199661979-a18e772a-40e0-452c-b585-92c5841124f4.png">

만약에 파일 변환 시스템을 운영한다고 하자.
이때 파일이 신규로 업데이트 된 경우라면 각 파일 저장소의 파일을 직접 활용하여 추가작업유무를 판단하여 진행 할 수 있겠지만,
파일의 사이즈가 큰 경우에는 직접 파일을 들고다니기도 어렵고, 파일을 업로드 하는 시간의 차도 있다.

업로드를 끝가지 하는동안, 이전 파일을 사용하고 싶은 (Read) 유저는 사용하기 어렵다.
따라서 이런 경우에도 별도의 메시지 큐를 두고서 선형적인 시스템으로 운용되도록 설계 할 수 있다.

---

# 선형성 시스템 구축하기

그럼 어떻게 만드나?

## 순서화와 인과성

순서와 인과..

### 순서화는 인과성을 보존하는 데 도움을 준다. 인과성이 중요한 경우가 있다.
- 인과성이 지켜지지 않으면 대화의 관찰자가 질문보다 그 질문의 응답의 먼저 볼 수 있다.

### 인과성은 이벤트에 순서를 부과한다. 결과가 나타나기 전에 원인이 발생한다.
- 메시지를 받기 전에 메시지를 보낸다.
- 답변하기 전에 질문을 한다.


# 인과적 일관성 아이디어
# 1) 일련번호 순서화
- 일련번호나 타임스탬프를 사용하여 이벤트의 순서를 정할 수 있다.
- 단일 리더 시스템에서는 하나의 리더의 기반으로 결정되므로 순서를 정할 수 있으나 단일 리더 시스템이 아니라면 이 방식은 인과적 일관성을 보장할 수 없다.
- 일련번호를 위해 쓰기 노드별로 특별한 규칙을 정해서 중복이 발생하지 않도록 할 수 있지만 인과성에 일관적이지 않다.
- 타임스탬프를 활용하더라도 해상도가 충분하지 않다면 인과성에 일관적이지 않다.

# 2) 인과적 의존성 담기
- 각 데이터베이스에 이전 결과에 관한 이력을 담고 들고다닌다.

![image](https://user-images.githubusercontent.com/8626130/199663771-d8e463e2-f8ec-448f-82f7-f34a298806fd.png)

https://en.wikipedia.org/wiki/Version_vector
https://martinfowler.com/articles/patterns-of-distributed-systems/version-vector.html

# 2) 램포트 타임스탬프

<img width="682" alt="image" src="https://user-images.githubusercontent.com/8626130/199663409-eb1d5ec4-a162-42c1-a737-c60dd9de1156.png">

- 램포트 타임스탬프를 할용해서 분산 시스템에서 일관적인 일련번호를 생성할 수 있다.
- 램포트 타임스탬프는 (카운터, 노드 ID) 쌍을 가져 각 노드의 카운터 값이 같더라도 노드 ID 기반으로 식별이 가능하도록 한다.
- 각 노드 별로 처리한 연산 개수를 카운터로 유지한다.
- 램포트 타임스탬프는 일 기준 시계와 관련이 없지만 전체 순서화를 제공한다.
- 카운터가 큰 것이 타임 스탬프가 크다.
- 카운터 값이 같으면 노드 ID가 큰 것이 타임 스탬프가 크다.
- 이 방식을 지원하기 위해서는 모든 노드와 클라이언트가 카운터 최댓값을 추적하고 모든 요청에 그 최대값을 포함시켜야 한다.
- 해당 최대값을 기반으로 각 노드는 자신의 카운터를 그 최대값으로 동기화하여 언제나 모든 노드가 최대값을 알 수 있도록 한다.
- 타임스탬프 순서화는 모든 연산이 처리되고 그 연산을 모은 후에 순서가 드러나기 때문에 유일성 제약 조건과 같은 구현에서 사용할 수 없다.


#3) 전체 순서 브로드캐스트 (Total Order Broadcast)

모든 노드가 같은 순서로 쓰기를 하게끔 하자.
1. 단일 리더방식 -> 단일실패지점 -> 단일리더가 처리 할 수 있는 수준을 넘어설 때 어떻게 할것인가? 
2. 로그를 기반으로 전체 노드의 쓰기가 compare-and-set이 동작하게끔 한다.

<img width="919" alt="image" src="https://user-images.githubusercontent.com/8626130/199665362-0329eb19-4461-4836-bd26-f7b2376981e8.png">

https://1ambda.github.io/cloud-computing/cloud-computing-6/


- 전체 순서 브로드캐스트를 통해 유일성 제약 조건을 구현할 수 있다.
- 전체 순서 브로드캐스트는 노드 사이에 메시지를 교환하는 프로토콜로 기술된다.
- 전체 순서 브로드캐스트 알고리즘은 노드나 네트워크에 결함이 있더라도 신뢰성(정확히 한번)과 순서화 속성이 항상 만족되도록 보장해야 한다.
- 결함 당시엔 메시지 전달이 불가하더라도 재시도를 통해 복구 시 순서에 맞게 전달이 가능하도록 해야 한다.
- 메시지의 순서는 해당 메시지가 전달되는 시점에 고정된다.
- 주키퍼나 etcd같은 분산 코디네이션 서비스가 이를 구현한다.
- 전체 순서 브로드캐스트는 신뢰성과 순서화 속성이 항상 보장되므로 데이터베이스 복제에 사용하기 적절하다.


# 분산 트랜젝션에서의 합의 (Consensus)

분산시스템에서의 트랜젝션은 일련의 합의 알고리즘을 통해서 진행한다.

대부분의 경우에는 비잔틴 문제를 해결하는 것을 시작으로 다양한 이점의 견고한 합의 알고리즘 이용한다.

Atomic Commit and Two-Phase Commit 이 대표적이나 그 결함이 있다. (책에서 소개)

- Viewstamped Replication, Raft, Zab 등 이 대표적이다.



# CAP 정리
- 흔히 CAP 정리는 일관성(Consistency), 가용성(Availability), 분단 내성(Partition tolerance) 중 두 개를 고를 수 있다고 표현되지만 이는 오해의 소지가 있다.
- 네트워크 분단은 결함이므로 선택할 수 있는 것이 아니고 반드시 발생하기 때문에 분단 내성은 반드시 고려되어야 한다.

즉 CAP 정리는 네트워크 분단(P)이 생겼을 때 일관성(C)과 가용성(A) 중 하나를 선택할 수 있다고 보는게 좋다.
여기서 일관성은 선형성(Strong Consistency)로 보는 것이 옳다.

- 만약 애플리케이션에서 선형성이 요구된다면 네트워크 분단 시 가용성을 보장할 수 없게 된다. 
- 반대로 애플리케이션에서 선형성을 요구하지 않는다면 네트워크 분단 시 선형적이지 않지만 가용한 상태를 제공할 수 있다.
- 선형성이 필요 없는 애플리케이션은 네트워크 문제에 더 강인하다.

http://eincs.com/2013/07/misleading-and-truth-of-cap-theorem/