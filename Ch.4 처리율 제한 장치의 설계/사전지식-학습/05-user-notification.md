# 사용자 알림

## 왜 필요한가

요청이 차단됐을 때 클라이언트가 아무 정보도 받지 못하면, 언제 재시도해야 할지 알 수 없다.
HTTP 표준 헤더를 통해 클라이언트에게 명확한 정보를 제공해야 한다.

---

## HTTP 상태 코드

| 코드 | 의미 | Rate Limiting 적합 여부 |
|------|------|----------------------|
| 200 | 성공 | X |
| 400 | 잘못된 요청 | X (요청 자체의 문제가 아님) |
| **429** | **Too Many Requests** | **O — 표준** |
| 503 | 서비스 불가 | X (서버 장애 상황에 사용) |

---

## 응답 헤더

| 헤더 | 의미 | 예시 |
|------|------|------|
| `Retry-After` | N초 후 재시도 가능 | `Retry-After: 30` |
| `X-RateLimit-Limit` | 허용 한도 | `X-RateLimit-Limit: 100` |
| `X-RateLimit-Remaining` | 현재 윈도우에서 남은 횟수 | `X-RateLimit-Remaining: 0` |
| `X-RateLimit-Reset` | 윈도우 리셋 시각 (Unix timestamp) | `X-RateLimit-Reset: 1735689600` |

차단 시 응답 예시:
```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1735689600
Retry-After: 30
```

---

## 이 챕터에서의 적용

> 설계 결정 단계에서 작성
