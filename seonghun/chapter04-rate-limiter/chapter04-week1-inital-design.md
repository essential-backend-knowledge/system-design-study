# chapter04-week1-inital-design
## 처리율 제한 장치 설계

### 요구사항
- 설정된 처리율 초과 요청 정확히 제한
- 낮은 응답시간 (이 장치가 http 응답시간 영향주면 안됨)
- 가능한 적은 메모리 사용
- 분산형 처리율 제한 (하나 장치로 여러 분산된 서버나 프로세스에서 공유 가능해야한다)
- 요청 제한 시 사용자에게 분명히 제한되었음을 알려줘야함
- 높은 결함 감내성 (제한장치 장애 발생해도 전체 시스템 영향 주면 안된다)

#### 토큰 버킷 VS 커넥션풀
- 토큰 버킷 방식 선택
- 동시 작업 처리수 제한보다는 시간당 요청수 제한이 목적

#### API 게이트웨이 내장 VS 독립 인프라로 구성
- 독립 인프라로 구성
- 하나 장치로 여러 분산된 서버, 프로세스 공유 가능 요구
  -> 게이트웨이도 여러대인 환경도 지원가능해야한다고 판단
- 아주 약간의 속도 포기 대신 여러 게이트웨이 지원 가능한 구조 선택

#### 실제 토큰 생성 VS 의존 인스턴스당 마지막 요청 기준 토큰 계산
- 실제 토큰 생성 : 주기적으로 설정한 수만큼 토큰 저장 (메모리 큐 추가)
- 마지막 요청 기준 토큰 계산
  - 맵 자료구조 사용
  - 키 : 버킷 의존 인스턴스 아이디
  - 값 : 마지막 요청 시간
  - 기존 마지막 요청 시간 기준 과 현재 시간 계산하여 그 사이 남은 토큰 있으면 허용

- 후자 선택
- 굳이 스케줄러를 돌리며 토큰을 채워줄 필요가 없다.
- 메모리 측면에서도 후자 방식이 사용량이 적다

#### 제한장치 장애 시
- 응답없거나 장애시 게이트웨이측에서 그냥 서비스로 흘리도록 한다

##### 설정값
- 최대 토큰 수
- 주기당 생성할 토큰 수

![[image.png]]
``` lua
-- KEYS[1]: 버킷 키 이름 (예: "ratelimit:user123")
-- ARGV[1]: 버킷의 최대 토큰 용량 (Max Capacity)
-- ARGV[2]: 초당 토큰 충전 속도 (Refill Rate per second)
-- ARGV[3]: 현재 요청에서 소비할 토큰 수 (보통 1)
-- ARGV[4]: 현재 시간 (Unix Timestamp, 초 단위)

local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local refill_rate = tonumber(ARGV[2])
local requested = tonumber(ARGV[3])
local now = tonumber(ARGV[4])

-- 레디스에서 기존 버킷 데이터 가져오기
local data = redis.call('HMGET', key, 'tokens', 'last_updated')
local tokens = tonumber(data[1])
local last_updated = tonumber(data[2])

-- 기존 데이터가 없으면 초기화
if not tokens then
    tokens = capacity
    last_updated = now
else
    -- 경과 시간 계산 및 토큰 충전
    local elapsed = now - last_updated
    if elapsed > 0 then
        tokens = math.min(capacity, tokens + (elapsed * refill_rate))
        last_updated = now
    end
end

-- 토큰이 충분한지 확인
if tokens >= requested then
    tokens = tokens - requested
    -- 업데이트된 상태 저장 및 만료 시간 설정 (예: 1시간)
    redis.call('HMSET', key, 'tokens', tokens, 'last_updated', last_updated)
    redis.call('EXPIRE', key, 3600)
    return 1 -- 요청 승인 (Allowed)
else
    -- 토큰이 부족하면 저장만 업데이트하고 거절
    redis.call('HMSET', key, 'tokens', tokens, 'last_updated', last_updated)
    return 0 -- 요청 거절 (Rate Limited)
end

```
![[image 2.png]]
