# 키-값 저장소 (Key-Value Store) API 명세서

> **문서 목적**: 클라이언트가 키-값 저장소와 통신하기 위한 API 인터페이스를 정의한다.  
> **참조 문서**: `key-value-store-requirements.md`, `key-value-store-architecture.md`, `key-value-store-data-model.md`  
> **통신 방식**: HTTP/1.1 REST API  
> **기본 URL**: `http://{coordinator-host}:{port}/v1`

---

## 1. 공통 규약

### 1.1 요청 / 응답 형식

- 요청 본문(Request Body): `application/json`
- 응답 본문(Response Body): `application/json`
- 문자 인코딩: `UTF-8`
- 키 인코딩: URL에 포함되는 키는 URL 인코딩(Percent Encoding) 적용

### 1.2 공통 요청 헤더

| 헤더 | 필수 여부 | 설명 | 예시 |
|------|---------|------|------|
| `Content-Type` | 본문 있는 요청 시 필수 | 요청 본문 형식 | `application/json` |
| `X-Consistency-Level` | 선택 | 일관성 수준 지정 | `strong` / `eventual` / `weak` |
| `X-Request-Id` | 선택 | 요청 추적용 고유 ID | `req-550e8400-e29b` |

### 1.3 일관성 수준 (X-Consistency-Level)

아키텍처 문서의 쿼럼(W/R/N) 설정과 매핑된다.

| 값 | W | R | 특성 |
|----|---|---|------|
| `strong` | N | N | 모든 노드 동의 후 응답. 가장 느리지만 항상 최신 값 |
| `eventual` | 1 | 1 | 즉시 응답. 일시적으로 오래된 값 반환 가능 |
| `weak` (기본값) | 1 | 1 | 최신 값 보장 없음, 최고 성능 |

헤더를 생략하면 `weak`가 적용된다.

### 1.4 공통 응답 헤더

| 헤더 | 설명 | 예시 |
|------|------|------|
| `X-Request-Id` | 요청 추적 ID (요청 헤더 값 그대로 반환) | `req-550e8400-e29b` |
| `X-Version` | 해당 키-값의 현재 버전 번호 | `7` |
| `X-Node-Id` | 응답을 처리한 스토리지 노드 ID | `node-b` |

### 1.5 공통 에러 응답 형식

모든 에러는 아래 구조로 반환된다.

```json
{
  "error": {
    "code": "KEY_NOT_FOUND",
    "message": "The requested key does not exist.",
    "request_id": "req-550e8400-e29b"
  }
}
```

### 1.6 에러 코드 목록

| HTTP 상태 | error.code | 설명 |
|----------|-----------|------|
| `400` | `INVALID_KEY` | 키 형식이 잘못됨 (제어 문자 포함 등) |
| `400` | `PAYLOAD_TOO_LARGE` | 키+값 크기가 10KB 초과 |
| `400` | `INVALID_TTL` | TTL 값이 음수이거나 형식 오류 |
| `404` | `KEY_NOT_FOUND` | 해당 키가 존재하지 않음 |
| `408` | `REQUEST_TIMEOUT` | 쿼럼 응답 대기 시간 초과 |
| `429` | `RATE_LIMITED` | 요청 한도 초과 |
| `500` | `INTERNAL_ERROR` | 서버 내부 오류 |
| `503` | `QUORUM_UNAVAILABLE` | 가용 노드 수가 쿼럼 미달 (CP 모드 시 발생 가능) |

---

## 2. API 목록

| 메서드 | 경로 | 설명 |
|--------|------|------|
| `PUT` | `/v1/keys/{key}` | 키-값 저장 (생성 또는 덮어쓰기) |
| `GET` | `/v1/keys/{key}` | 키로 값 조회 |
| `DELETE` | `/v1/keys/{key}` | 키-값 삭제 |
| `PUT` | `/v1/keys` (batch) | 여러 키-값 일괄 저장 |
| `GET` | `/v1/keys` (batch) | 여러 키 일괄 조회 |
| `GET` | `/v1/health` | 시스템 상태 확인 |

---

## 3. 단건 API 상세

### 3.1 PUT /v1/keys/{key} — 키-값 저장

키-값 쌍을 저장한다. 키가 이미 존재하면 덮어쓴다.

#### 요청

```
PUT /v1/keys/user%3Aprofile%3A1001 HTTP/1.1
Host: coordinator.kv-store.internal
Content-Type: application/json
X-Consistency-Level: eventual
X-Request-Id: req-550e8400-e29b
```

```json
{
  "value": "{\"name\": \"Alice\", \"age\": 30}",
  "ttl": 3600
}
```

#### 요청 파라미터

| 위치 | 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| Path | `key` | string | ✅ | 저장할 키 (URL 인코딩 적용) |
| Body | `value` | string | ✅ | 저장할 값 (문자열로 직렬화) |
| Body | `ttl` | integer | ❌ | 만료 시간 (초 단위, 생략 시 영구 보존) |

#### 응답 — 성공 (201 Created / 200 OK)

- 새로 생성: `201 Created`
- 기존 키 덮어쓰기: `200 OK`

```
HTTP/1.1 201 Created
X-Request-Id: req-550e8400-e29b
X-Version: 1
X-Node-Id: node-a
```

```json
{
  "key": "user:profile:1001",
  "version": 1,
  "expires_at": "2026-07-03T12:00:00Z"
}
```

#### 응답 — 실패 예시

```
HTTP/1.1 400 Bad Request
```

```json
{
  "error": {
    "code": "PAYLOAD_TOO_LARGE",
    "message": "Key-value pair size 12345 bytes exceeds the 10240 byte limit.",
    "request_id": "req-550e8400-e29b"
  }
}
```

#### 동작 흐름

```
① 클라이언트 → 코디네이터: PUT /v1/keys/user:profile:1001
② 코디네이터: 키 유효성 검사 (크기, 형식)
③ 코디네이터: 안정 해시로 담당 노드 결정
④ 코디네이터 → 스토리지 노드(들): 쓰기 요청 전달
⑤ 각 노드: WAL 기록 → Memtable 저장
⑥ W개 노드 성공 응답 수신
⑦ 코디네이터 → 클라이언트: 201/200 응답 반환
```

---

### 3.2 GET /v1/keys/{key} — 값 조회

키에 해당하는 값을 반환한다.

#### 요청

```
GET /v1/keys/user%3Aprofile%3A1001 HTTP/1.1
Host: coordinator.kv-store.internal
X-Consistency-Level: strong
X-Request-Id: req-7a0c1234-b3d8
```

#### 요청 파라미터

| 위치 | 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| Path | `key` | string | ✅ | 조회할 키 (URL 인코딩 적용) |

#### 응답 — 성공 (200 OK)

```
HTTP/1.1 200 OK
X-Request-Id: req-7a0c1234-b3d8
X-Version: 7
X-Node-Id: node-b
```

```json
{
  "key": "user:profile:1001",
  "value": "{\"name\": \"Alice\", \"age\": 30}",
  "version": 7,
  "created_at": "2026-06-01T09:00:00Z",
  "updated_at": "2026-06-03T14:22:00Z",
  "expires_at": null
}
```

#### 응답 — 키 없음 (404 Not Found)

```
HTTP/1.1 404 Not Found
```

```json
{
  "error": {
    "code": "KEY_NOT_FOUND",
    "message": "The requested key 'user:profile:1001' does not exist.",
    "request_id": "req-7a0c1234-b3d8"
  }
}
```

#### 응답 필드 설명

| 필드 | 타입 | 설명 |
|------|------|------|
| `key` | string | 조회한 키 |
| `value` | string | 저장된 값 |
| `version` | integer | 현재 버전 번호 (쓰기 횟수) |
| `created_at` | string (ISO 8601) | 최초 생성 시각 |
| `updated_at` | string (ISO 8601) | 마지막 수정 시각 |
| `expires_at` | string \| null | 만료 시각. null이면 영구 보존 |

#### 동작 흐름

```
① 클라이언트 → 코디네이터: GET /v1/keys/user:profile:1001
② 코디네이터: 안정 해시로 담당 노드 결정
③ 코디네이터 → R개 스토리지 노드: 동시 읽기 요청
④ 각 노드: 블룸 필터 → Memtable → SSTable 순서로 탐색
⑤ R개 응답 수집 → 가장 높은 version 값 선택
⑥ (선택) Read Repair: 오래된 복제본에 최신 값 동기화
⑦ 코디네이터 → 클라이언트: 200 응답 반환
```

---

### 3.3 DELETE /v1/keys/{key} — 키-값 삭제

키-값 쌍을 삭제한다. 내부적으로는 tombstone 마킹(논리적 삭제)이며, 물리적 제거는 Compaction 시 수행된다.

#### 요청

```
DELETE /v1/keys/session%3Aabc123 HTTP/1.1
Host: coordinator.kv-store.internal
X-Request-Id: req-c91f2200-44a1
```

#### 요청 파라미터

| 위치 | 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| Path | `key` | string | ✅ | 삭제할 키 (URL 인코딩 적용) |

#### 응답 — 성공 (200 OK)

```
HTTP/1.1 200 OK
X-Request-Id: req-c91f2200-44a1
X-Version: 8
```

```json
{
  "key": "session:abc123",
  "deleted": true,
  "version": 8
}
```

#### 응답 — 키 없음 (404 Not Found)

```json
{
  "error": {
    "code": "KEY_NOT_FOUND",
    "message": "The requested key 'session:abc123' does not exist.",
    "request_id": "req-c91f2200-44a1"
  }
}
```

---

## 4. 배치(Batch) API 상세

단건 API를 반복 호출하면 네트워크 오버헤드가 크다. 여러 키를 한 번에 처리할 때 배치 API를 사용한다.

### 4.1 PUT /v1/keys — 일괄 저장

여러 키-값 쌍을 한 번에 저장한다. 각 키는 독립적으로 처리되며, 일부 실패해도 나머지는 저장된다 (Partial Success).

#### 요청

```
PUT /v1/keys HTTP/1.1
Host: coordinator.kv-store.internal
Content-Type: application/json
X-Consistency-Level: eventual
```

```json
{
  "entries": [
    {
      "key": "user:profile:1001",
      "value": "{\"name\": \"Alice\"}",
      "ttl": null
    },
    {
      "key": "user:profile:2345",
      "value": "{\"name\": \"Bob\"}",
      "ttl": 86400
    },
    {
      "key": "session:abc123",
      "value": "user:1001",
      "ttl": 3600
    }
  ]
}
```

#### 요청 제약

| 항목 | 제약 |
|------|------|
| 최대 항목 수 | 한 번에 최대 100개 |
| 개별 크기 제한 | 각 키-값 쌍은 10KB 이하 |
| 중복 키 | 배열 내 동일 키 존재 시 마지막 항목으로 저장 |

#### 응답 — 207 Multi-Status

일부 성공, 일부 실패가 가능하므로 `207 Multi-Status`로 반환한다.

```
HTTP/1.1 207 Multi-Status
```

```json
{
  "results": [
    {
      "key": "user:profile:1001",
      "status": 201,
      "version": 1
    },
    {
      "key": "user:profile:2345",
      "status": 200,
      "version": 4
    },
    {
      "key": "session:abc123",
      "status": 400,
      "error": {
        "code": "INVALID_KEY",
        "message": "Key contains invalid characters."
      }
    }
  ],
  "summary": {
    "total": 3,
    "succeeded": 2,
    "failed": 1
  }
}
```

---

### 4.2 GET /v1/keys — 일괄 조회

여러 키의 값을 한 번에 조회한다.

#### 요청

```
GET /v1/keys?keys=user%3Aprofile%3A1001,user%3Aprofile%3A2345,session%3Aabc123 HTTP/1.1
Host: coordinator.kv-store.internal
X-Consistency-Level: weak
```

#### 요청 파라미터

| 위치 | 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|------|
| Query | `keys` | string | ✅ | 쉼표로 구분된 키 목록 (URL 인코딩 적용) |

최대 100개 키 조회 가능.

#### 응답 — 200 OK

존재하지 않는 키도 결과에 포함하되 `found: false`로 표시한다.

```json
{
  "results": [
    {
      "key": "user:profile:1001",
      "found": true,
      "value": "{\"name\": \"Alice\"}",
      "version": 1,
      "expires_at": null
    },
    {
      "key": "user:profile:2345",
      "found": true,
      "value": "{\"name\": \"Bob\"}",
      "version": 4,
      "expires_at": "2026-06-04T12:00:00Z"
    },
    {
      "key": "session:abc123",
      "found": false
    }
  ],
  "summary": {
    "total": 3,
    "found": 2,
    "not_found": 1
  }
}
```

---

## 5. 시스템 상태 API

### 5.1 GET /v1/health — 헬스 체크

코디네이터 및 스토리지 노드의 상태를 반환한다. 로드 밸런서의 헬스 체크 엔드포인트로 활용한다.

#### 요청

```
GET /v1/health HTTP/1.1
Host: coordinator.kv-store.internal
```

#### 응답 — 정상 (200 OK)

```json
{
  "status": "healthy",
  "coordinator": "node-coordinator-1",
  "timestamp": "2026-06-03T14:00:00Z",
  "nodes": {
    "total": 3,
    "healthy": 3,
    "unhealthy": 0,
    "detail": [
      { "id": "node-a", "status": "healthy", "latency_ms": 1 },
      { "id": "node-b", "status": "healthy", "latency_ms": 2 },
      { "id": "node-c", "status": "healthy", "latency_ms": 1 }
    ]
  },
  "quorum": {
    "N": 3,
    "W": 2,
    "R": 2,
    "available": true
  }
}
```

#### 응답 — 쿼럼 미달 (503 Service Unavailable)

```json
{
  "status": "degraded",
  "coordinator": "node-coordinator-1",
  "timestamp": "2026-06-03T14:00:00Z",
  "nodes": {
    "total": 3,
    "healthy": 1,
    "unhealthy": 2,
    "detail": [
      { "id": "node-a", "status": "healthy",   "latency_ms": 1 },
      { "id": "node-b", "status": "unhealthy", "latency_ms": null },
      { "id": "node-c", "status": "unhealthy", "latency_ms": null }
    ]
  },
  "quorum": {
    "N": 3,
    "W": 2,
    "R": 2,
    "available": false
  }
}
```

---

## 6. 요청 / 응답 전체 흐름 예시

### 시나리오: 세션 저장 후 조회, 만료 후 재조회

```
1. 세션 저장
   PUT /v1/keys/session%3Aabc123
   Body: { "value": "user:1001", "ttl": 3600 }
   → 201 Created, version: 1

2. 세션 조회 (TTL 유효 중)
   GET /v1/keys/session%3Aabc123
   → 200 OK
     { "value": "user:1001", "version": 1, "expires_at": "2026-06-03T15:00:00Z" }

3. 세션 조회 (TTL 만료 후)
   GET /v1/keys/session%3Aabc123
   → 404 Not Found
     { "error": { "code": "KEY_NOT_FOUND", ... } }

4. 세션 명시적 삭제
   DELETE /v1/keys/session%3Aabc123
   → 200 OK, deleted: true, version: 2
     (내부적으로 tombstone 마킹)
```

---

## 7. 제약 조건 및 한계

| 항목 | 제약 |
|------|------|
| 키-값 크기 | 키 + 값 합산 10KB 이하 |
| 배치 최대 항목 수 | 100개 |
| 범위 쿼리 | 미지원 (키 기반 단건/배치 접근만 가능) |
| 트랜잭션 | 미지원 (여러 키에 대한 원자적 연산 불가) |
| 키 목록 조회 | 미지원 (전체 키 스캔 불가) |
| 값 부분 업데이트 | 미지원 (항상 전체 값 덮어쓰기) |

---

## 8. 버전 이력

| 버전 | 변경 내용 |
|------|---------|
| v1.0 | 최초 작성. PUT / GET / DELETE / Batch / Health API 정의 |

---

*본 문서는 클라이언트-서버 간 인터페이스를 정의하며, 내부 노드 간 통신 프로토콜은 별도 내부 API 문서에서 다룬다.*
