# CAP 정리 & PACELC 모델 완전 정리

---

## 1. CAP 정리란?

2000년 Eric Brewer가 제시하고, 2002년 Gilbert & Lynch가 수학적으로 증명한 이론.

> **분산 시스템은 Consistency, Availability, Partition Tolerance 세 가지를 동시에 모두 만족할 수 없다.**

---

## 2. 세 가지 속성

### Consistency (일관성)
- 어떤 노드에서 읽어도 **항상 최신값**을 반환해야 함
- CAP에서의 C는 정확히 **Linearizability(선형화 가능성)**
- "틀린 답보다 차라리 응답 안 하겠다"

### Availability (가용성)
- 살아있는 노드는 **모든 요청에 반드시 응답**해야 함
- 응답 내용이 최신값인지는 보장하지 않음
- "틀린 답일 수 있어도 일단 응답하겠다"

### Partition Tolerance (분단 내성)
- 네트워크 분리가 발생해도 **시스템이 계속 동작**
- 실제 분산 환경에서 네트워크 장애는 반드시 발생함
- **P는 포기할 수 없음** → 결국 C vs A 선택

---

## 3. 왜 셋 다 불가능한가?

```
[네트워크 분단 발생]
Client → Node A  ~~~X~~~  Node B
          (서로 통신 불가)

Client 1 → Node A에 write(x = 99)
Client 2 → Node B에 read(x) 요청

선택 1: 응답 거부  →  CP (Availability 포기)
선택 2: x = 10 반환  →  AP (Consistency 포기)
```

---

## 4. C와 A의 정확한 차이

### 헷갈리기 쉬운 포인트

| 오해                         | 정확한 의미               |
| -------------------------- | -------------------- |
| Availability = 서버가 죽지 않는 것 | 살아있는 노드가 반드시 응답하는 것  |
| Consistency = 노드끼리 같은 값    | 읽기가 항상 최신 쓰기를 반영하는 것 |
| 둘 중 하나만 선택                 | 실제로는 스펙트럼으로 존재       |

### 일관성 스펙트럼

```
강한 일관성 ◄─────────────────────────────► 강한 가용성
Linearizability  Sequential  Causal  Eventual
     ZooKeeper      ←──────────────────────→   Cassandra
```

### ACID의 C vs CAP의 C

| | ACID의 C | CAP의 C |
|---|---|---|
| 의미 | 트랜잭션 전후 DB 무결성 유지 | 분산 노드 간 최신값 동기화 |
| 주체 | 단일 DB 내부 | 분산 시스템 전체 |
| 예시 | 계좌이체 후 잔액 합계 유지 | 모든 노드가 같은 최신값 반환 |

---

## 5. CP vs AP 동작 방식

### 기준: Quorum(과반수)

```
노드 3개 → 과반수 = 2개

CP: 과반수 미달 시 쓰기 차단
AP: 노드 1개만 살아있어도 응답
```

### CP에서 노드 장애 시

```
[1개 죽음] → 과반수 충족 → 쓰기 허용 ✅
[2개 죽음] → 과반수 미달 → 쓰기 차단 ❌  ← Availability 포기
```

### AP에서 노드 장애 시

```
[분단 발생]

그룹 A              그룹 B
Node 1  ~~~X~~~   Node 3
Node 2

Client 1 → Node 1에 write(x = 99) → 성공
Client 2 → Node 3에 read(x) → x = 10 반환  ← 옛날 값, Consistency 포기
```

### AP의 분단 복구 후 충돌 해결 전략

| 전략 | 방식 | 대표 시스템 | 단점 |
|---|---|---|---|
| Last Write Wins (LWW) | 타임스탬프 기준 나중 값 채택 | Cassandra | 시계 오차 시 잘못된 값 선택 가능 |
| Vector Clock | 쓰기 순서를 버전으로 추적 | DynamoDB | 충돌 해결을 앱 레벨에서 처리 |
| CRDT | 수학적으로 항상 합칠 수 있는 자료구조 | Redis 일부 | 적용 가능한 자료구조 제한 |

---

## 6. CP vs AP 트레이드오프

### CP 선택 시

**얻는 것**
- 데이터 정확성 보장 (읽으면 항상 최신값)
- 충돌 해결 로직 불필요 (복구 후 자동 정합성 유지)
- 개발 단순성 ("DB 믿고 써도 된다"는 전제로 개발 가능)

**잃는 것**
- 가용성 저하 (과반수 장애 시 쓰기 중단)
- 레이턴시 증가 (쓰기마다 과반수 응답 대기)
- 운영 복잡성 (노드 추가/제거 시 리밸런싱 동안 중단 가능)

**적합한 상황**
```
금융 송금    → 잔액이 틀리면 안 됨
재고 차감    → 중복 차감은 손실
쿠폰 발급    → 중복 발급은 비용
분산 락      → 두 노드가 동시에 Lock을 가지면 안 됨
리더 선출    → Split-Brain 방지
```

### AP 선택 시

**얻는 것**
- 높은 가용성 (노드 1개만 살아있어도 응답)
- 낮은 레이턴시 (과반수 대기 없이 즉시 응답)
- 수평 확장 용이 (노드 추가해도 서비스 중단 없음)

**잃는 것**
- 일시적 데이터 불일치 (분단 중 노드마다 다른 값 반환 가능)
- 충돌 해결 책임이 개발자에게 (LWW, Vector Clock, CRDT 등 직접 구현)
- 비즈니스 로직 복잡성 ("이 데이터가 최신이 아닐 수 있다"는 전제로 설계)

**적합한 상황**
```
SNS 좋아요 수   → 잠깐 틀려도 무방
장바구니        → 일시적 이전 상태 허용
로그 수집       → 순서 약간 틀려도 결국 다 쌓이면 됨
추천 피드       → 최신이 아닌 추천이 나와도 UX에 큰 영향 없음
DNS             → 전파 지연 허용, 대신 항상 응답
```

---

## 7. 주요 시스템 분류

| 시스템 | 분류 | 특징 |
|---|---|---|
| ZooKeeper | CP | Leader Quorum 기반, 강한 일관성 |
| HBase | CP | HDFS 위에서 동작 |
| etcd | CP | Kubernetes 메타데이터 저장 |
| Cassandra | AP | Consistent Hashing, LWW |
| DynamoDB | AP | Vector Clock, Eventual Consistency |
| CouchDB | AP | 멀티 마스터 복제 |
| Redis Cluster | AP | Hash Slot 기반 샤딩 |
| Kafka (acks=all) | CP에 가까움 | ISR 전체 확인 후 응답 |
| Kafka (acks=1) | AP에 가까움 | Leader만 확인 |

---

## 8. PACELC 모델

### CAP의 한계

CAP은 "장애 시 어떻게 할 것인가"만 다룸.  
**정상 상태(99.9%)의 Latency 트레이드오프를 설명 못 함.**

### PACELC 정의

2012년 Daniel Abadi가 제안.

```
if Partition:
    Availability  vs  Consistency   (= CAP의 AP/CP)
Else (정상 상태):
    Latency       vs  Consistency
```

### 정상 상태에서의 딜레마

```
강한 일관성 원할 시
→ 쓰기 시 모든 복제본에 동기적 반영 확인
→ 네트워크 RTT만큼 Latency 증가

낮은 Latency 원할 시
→ 쓰기 시 일부 복제본만 확인 or 비동기 복제
→ 일시적 불일치 발생 가능
```

### PACELC 4가지 조합

| 분류 | 장애 시 | 정상 시 | 대표 시스템 |
|---|---|---|---|
| **PA / EL** | Availability 우선 | Latency 우선 | Cassandra, DynamoDB |
| **PC / EC** | Consistency 우선 | Consistency 우선 | ZooKeeper, HBase, etcd |

실제로는 **PA/EL** 과 **PC/EC** 가 대부분.

### CAP vs PACELC 비교

| 항목 | CAP | PACELC |
|---|---|---|
| 제안자 | Eric Brewer (2000) | Daniel Abadi (2012) |
| 고려 상황 | 네트워크 분단 시 | 분단 시 + 정상 상태 |
| 트레이드오프 | C vs A | (분단) C vs A + (정상) C vs L |

---

## 9. 실무 적용

### 한 서비스 안에서 혼용

```
주문 처리 (재고 차감)   → CP  (중복 차감 절대 안 됨)
상품 조회 (캐시)        → AP  (잠깐 이전 가격 보여도 무방)
알림/로그               → AP  (유실보다 속도가 중요)
결제 원장               → CP  (금액은 정확해야 함)
```

### Kafka acks 설정

```
acks=all  →  ISR 전체 확인  →  PC/EC (강한 일관성)
acks=1    →  Leader만 확인  →  PA/EL에 가까움
acks=0    →  확인 없음      →  극단적 PA/EL
```

---

## 10. 한 줄 정리

```
Consistency = "맞는 답만 준다"  (틀리느니 침묵)
Availability = "항상 답을 준다"  (틀려도 일단 응답)

CP = "틀리느니 멈추겠다"  → 정확성이 생명인 도메인
AP = "멈추느니 틀리겠다"  → 가용성이 생명인 도메인

둘 중 하나를 고르는 게 아니라
도메인별로 적절히 혼용하는 것이 실제 설계
```
